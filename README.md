from flask import Flask, render_template_string, request, redirect, url_for
import os
import openai

app = Flask(__name__)
app.secret_key = "secret"

# In-memory tasks
tasks = []

# HTML Template (Bootstrap)
TEMPLATE = """
<!DOCTYPE html>
<html>
<head>
    <title>AI To-Do List</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
</head>
<body class="bg-light">
<div class="container py-5">
    <h1 class="mb-4">📝 AI To-Do List</h1>
    
    <form method="POST" action="/add" class="mb-3 d-flex">
        <input type="text" name="task" placeholder="Enter a task" class="form-control me-2" required>
        <button class="btn btn-primary">Add</button>
    </form>
    
    <form method="POST" action="/ai" class="mb-3">
        <button class="btn btn-success">✨ AI Suggest Tasks</button>
    </form>

    <ul class="list-group">
        {% for t in tasks %}
            <li class="list-group-item d-flex justify-content-between align-items-center">
                <span>{{ t }}</span>
                <a href="{{ url_for('delete', idx=loop.index0) }}" class="btn btn-danger btn-sm">Delete</a>
            </li>
        {% else %}
            <li class="list-group-item text-muted">No tasks yet.</li>
        {% endfor %}
    </ul>
</div>
</body>
</html>
"""

@app.route("/")
def home():
    return render_template_string(TEMPLATE, tasks=tasks)

@app.route("/add", methods=["POST"])
def add():
    task = request.form.get("task")
    if task:
        tasks.append(task)
    return redirect(url_for("home"))

@app.route("/delete/<int:idx>")
def delete(idx):
    if 0 <= idx < len(tasks):
        tasks.pop(idx)
    return redirect(url_for("home"))

@app.route("/ai", methods=["POST"])
def ai_suggest():
    # Optional: set OPENAI_API_KEY in your environment
    api_key = os.getenv("OPENAI_API_KEY")
    if not api_key:
        # fallback suggestions
        suggestions = [
            "Plan today’s top 3 priorities",
            "Review emails",
            "Take a 10-minute walk",
        ]
    else:
        openai.api_key = api_key
        response = openai.chat.completions.create(
            model="gpt-4o-mini",
            messages=[
                {"role": "system", "content": "Suggest 5 short, practical tasks for productivity today."},
                {"role": "user", "content": "Generate tasks for my to-do list"}
            ]
        )
        text = response.choices[0].message.content.strip()
        suggestions = [s.strip("-• ") for s in text.split("\n") if s.strip()]

    tasks.extend(suggestions)
    return redirect(url_for("home"))

if __name__ == "__main__":
    app.run(debug=True)

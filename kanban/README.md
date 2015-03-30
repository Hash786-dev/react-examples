# Kanban

This example will be, essentially, an underpowered Trello clone. Here we'll
actually run a server and implement persistent storage with optimistic updating.

## The Server

We'll be using [Flask][flask] for the server, since we don't need much. First,
the imports and the app definition:

```python
import json

from flask import Flask, request
app = Flask(__name__, static_url_path='', static_folder='.')
```

Now, some barebones models:

```python
class Task(object):
    """A task."""

    def __init__(self, text):
        self.text = text

    def to_dict(self):
        return {"text": self.text}


class TaskList(object):
    """A list of Task objects."""

    def __init__(self, name, tasks):
        self.name = name
        self.tasks = tasks

    def to_dict(self):
        return {
            "name": self.name,
            "tasks": [task.to_dict() for task in self.tasks]
        }


class Board(object):
    """A collection of TaskLists."""

    def __init__(self, lists):
        self.lists = lists

    def to_dict(self):
        return {
            "lists": [list.to_dict() for list in self.lists]
        }
```

For persistence, we'll just use the memory:

```python
DB = Board([
    TaskList(name="Todo",
             tasks=[
                 Task("Write example React app"),
                 Task("Write documentation")
             ]),
    TaskList(name="Done",
             tasks=[
                 Task("Learn the basics of React")
             ])
])
```

### Routes

And now, the routes. This is just a simple REST API that uses JSON, handles
actions and even some errors.

First, we need a route that will be called on application startup, to get the
initial (or current) state of the Kanban board:

```python
@app.route("/api/board/")
def get_board():
    """Return the state of the board."""
    return json.dumps(DB.to_dict())
```

We also need a way to add new tasks to a list, which is what `add_task` does.

```python
@app.route("/api/<int:list_id>/task", methods=["PUT"])
def add_task(list_id):
    # Add a task to a list.
    try:
        DB.lists[list_id].tasks.append(Task(text=request.form.get("text")))
    except IndexError:
        return json.dumps({"status": "FAIL"})
    return json.dumps({"status": "OK"})
```

## The Client

[flask]: http://flask.pocoo.org/
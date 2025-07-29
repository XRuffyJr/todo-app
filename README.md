<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <title>Aufgabenliste mit Datum und Drag & Drop</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f4f4f4;
      padding: 20px;
    }
    h1 {
      text-align: center;
    }
    #form {
      display: flex;
      justify-content: center;
      gap: 10px;
      margin-bottom: 20px;
    }
    input, button {
      font-size: 16px;
      padding: 8px;
    }
    ul {
      list-style: none;
      padding: 0;
      max-width: 500px;
      margin: 0 auto;
    }
    li {
      background: #fff;
      padding: 10px;
      margin-bottom: 10px;
      border-radius: 6px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
      cursor: move;
    }
    li.done {
      text-decoration: line-through;
      color: gray;
    }
    .info {
      display: flex;
      flex-direction: column;
    }
    .delete-btn {
      background: red;
      color: white;
      border: none;
      padding: 6px 10px;
      cursor: pointer;
      border-radius: 4px;
    }
    .date {
      font-size: 12px;
      color: #888;
    }

    body {
        font-family: Arial, sans-serif;
        background-color: #f4f4f4;
        padding: 20px;
        margin: 0;
    }

    #form {
        display: flex;
        flex-wrap: wrap;
        justify-content: center;
        gap: 10px;
        margin-bottom: 20px;
        max-width: 500px;
        margin-left: auto;
        margin-right: auto;
    }

    #form input,
    #form button {
        flex: 1 1 100%;
        min-width: 250px;
    }

    @media (min-width: 600px) {
    #form input[type="text"] {
        flex: 2 1 auto;
    }

    #form input[type="date"],
    #form button {
        flex: 1 1 auto;
    }
    }

    ul {
        list-style: none;
        padding: 0;
        max-width: 500px;
        margin: 0 auto;
    }

    li {
        background: #fff;
        padding: 10px;
        margin-bottom: 10px;
        border-radius: 6px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        flex-wrap: wrap;
        box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        cursor: move;
    }

    .info {
        flex: 1 1 100%;
    }

    .delete-btn {
        margin-top: 5px;
    }
  </style>
</head>
<body>

<h1>📝 Aufgabenliste</h1>

<div id="form">
  <input type="text" id="task-input" placeholder="Neue Aufgabe..." />
  <input type="date" id="due-date" />
  <button id="add-btn">Hinzufügen</button>
</div>

<ul id="task-list"></ul>

<script>
  const input = document.getElementById("task-input");
  const dateInput = document.getElementById("due-date");
  const addBtn = document.getElementById("add-btn");
  const list = document.getElementById("task-list");

  let tasks = [];

  function saveTasks() {
    localStorage.setItem("tasks", JSON.stringify(tasks));
  }

  function loadTasks() {
    const data = localStorage.getItem("tasks");
    if (data) {
      tasks = JSON.parse(data);
      tasks.forEach(task => addTaskToDOM(task));
    }
  }

  function addTaskToDOM(task, index = tasks.length - 1) {
    const li = document.createElement("li");
    li.setAttribute("draggable", "true");
    li.dataset.index = index;

    const info = document.createElement("div");
    info.className = "info";

    const text = document.createElement("span");
    text.textContent = task.text;
    text.addEventListener("click", () => {
      li.classList.toggle("done");
      task.done = !task.done;
      saveTasks();
    });

    const date = document.createElement("span");
    date.className = "date";
    date.textContent = task.date ? `Fällig: ${task.date}` : "";

    info.appendChild(text);
    info.appendChild(date);

    const delBtn = document.createElement("button");
    delBtn.textContent = "🗑️";
    delBtn.className = "delete-btn";
    delBtn.addEventListener("click", () => {
      tasks.splice(index, 1);
      saveAndReload();
    });

    li.appendChild(info);
    li.appendChild(delBtn);
    list.appendChild(li);

    setupDragAndDrop();
  }

  function saveAndReload() {
    localStorage.setItem("tasks", JSON.stringify(tasks));
    list.innerHTML = "";
    tasks.forEach((task, i) => addTaskToDOM(task, i));
  }

  addBtn.addEventListener("click", () => {
    const text = input.value.trim();
    const date = dateInput.value;
    if (text === "") return;

    const task = { text, date, done: false };
    tasks.push(task);
    addTaskToDOM(task);
    saveTasks();

    input.value = "";
    dateInput.value = "";
    input.focus();
  });

  input.addEventListener("keydown", e => {
    if (e.key === "Enter") addBtn.click();
  });

  function setupDragAndDrop() {
    let dragged;

    document.querySelectorAll("li").forEach((li, index) => {
      li.addEventListener("dragstart", () => {
        dragged = index;
      });

      li.addEventListener("dragover", e => e.preventDefault());

      li.addEventListener("drop", () => {
        const targetIndex = parseInt(li.dataset.index);
        if (dragged !== targetIndex) {
          const moved = tasks.splice(dragged, 1)[0];
          tasks.splice(targetIndex, 0, moved);
          saveAndReload();
        }
      });
    });
  }

  // Beim Laden
  loadTasks();
</script>

</body>
</html>

# Exercise 7: Error Diagnosis Challenge

## Error Scenario Selected
Scenario 6: Global Variable Being Overwritten (JavaScript)

---

## Original Error Message
```
Uncaught TypeError: Cannot read properties of undefined (reading 'map')
    at displayTasks (taskManager.js:24)
    at addTask (taskManager.js:15)
    at HTMLButtonElement.onclick (index.html:1)
```

---

## Original Buggy Code
```javascript
let tasks = [];

function initApp() {
  tasks = [
    { id: 1, name: "Complete project proposal", completed: false },
    { id: 2, name: "Meeting with team", completed: true }
  ];
  displayTasks();
}

function addTask(taskName) {
  let tasks = { id: Date.now(), name: taskName, completed: false }; // Bug here!
  console.log("Task added:", tasks);
  displayTasks();
}

function displayTasks() {
  const taskListElement = document.getElementById('task-list');
  taskListElement.innerHTML = "";
  tasks.map(task => { // Error occurs here
    const taskElement = document.createElement('div');
    taskElement.innerHTML = `<div class="task-item">${task.name}</div>`;
    taskListElement.appendChild(taskElement);
  });
}
```

---

## AI Prompt Used (Prompt 1: Error Message Translation)
```
I need help understanding this error message from my JavaScript application.

Here's the complete error message and stack trace:

Uncaught TypeError: Cannot read properties of undefined (reading 'map')
    at displayTasks (taskManager.js:24)
    at addTask (taskManager.js:15)
    at HTMLButtonElement.onclick (index.html:1)

My application context:
- This happened when I was trying to add a new task using the Add Task button
- The application is a browser-based task manager built with vanilla JavaScript
- I'm using plain JavaScript (ES6) with no framework

Could you:
1. Explain what this error means in simple, non-technical terms
2. Identify the most relevant lines in the stack trace
3. List 2-3 of the most likely causes
4. Suggest what specific information I should look for in my code
5. Provide a step-by-step debugging approach

I'm particularly unfamiliar with variable scoping in JavaScript.
```

---

## Error Analysis

### Error Description
**What this error means in plain language:**

JavaScript is saying: "I tried to call .map() on something, but that something
is undefined — meaning it doesn't exist or has no value."

The .map() method only works on arrays. When the error says
"Cannot read properties of undefined (reading 'map')", it means
the variable `tasks` is no longer an array at the point where .map() is called.
Instead it has become undefined or a single object.

**Most relevant line in the stack trace:**
```
at displayTasks (taskManager.js:24)
```
This is where the crash actually happens — inside displayTasks() when
tasks.map() is called. The line above it (addTask at line 15) shows us
what triggered displayTasks() to run.

---

### Root Cause Identification

**The root cause is variable shadowing — a scoping bug.**

Inside the `addTask` function, the developer wrote:
```javascript
let tasks = { id: Date.now(), name: taskName, completed: false };
```

The word `let` here creates a **brand new local variable** also called `tasks`.
This new local `tasks` is a single object (not an array).

The problem: this new local `tasks` completely **shadows** (hides) the global
`tasks` array for the rest of the `addTask` function. When `displayTasks()` is
then called, JavaScript looks at the global `tasks` — but since the local
variable was never pushed into it, the global array was never updated.

**Chain of events leading to the error:**
1. Page loads → `initApp()` sets the global `tasks` to an array of 2 tasks ✅
2. User clicks "Add Task" → `addTask()` runs
3. Inside `addTask()`, `let tasks = {...}` creates a LOCAL variable named `tasks`
   that shadows the global `tasks` array
4. `displayTasks()` is called — it tries to use the GLOBAL `tasks`
5. The global `tasks` is still an array and this works... until the next call
6. Actually the real crash: after `addTask` runs, the global `tasks` was
   never updated with the new task, and depending on JS engine/timing,
   `tasks` in `displayTasks` scope resolves to the local object
7. `.map()` is called on an object instead of an array → TypeError crash

---

### Suggested Solution

**Fix: Remove `let` so the global variable is updated, not shadowed**
```javascript
// BEFORE (buggy):
function addTask(taskName) {
  let tasks = { id: Date.now(), name: taskName, completed: false }; // creates local variable
  displayTasks();
}

// AFTER (fixed):
function addTask(taskName) {
  const newTask = { id: Date.now(), name: taskName, completed: false }; // use a different name
  tasks.push(newTask); // push into the GLOBAL tasks array
  displayTasks();
}
```

**Why this fix works:**
- `newTask` is a clearly named local variable holding the single new task object
- `tasks.push(newTask)` adds it to the GLOBAL `tasks` array
- When `displayTasks()` runs, `tasks` is still an array and `.map()` works correctly

---

### Tests to Verify the Fix

1. **Basic add test:** Add one task and confirm it appears in the displayed list
2. **Multiple add test:** Add 3 tasks one by one and confirm all 3 appear
3. **Persistence test:** After adding a task, call `displayTasks()` manually
   and confirm the new task is still there (global array was updated)
4. **Type check test:** After calling `addTask()`, log `typeof tasks` and
   `Array.isArray(tasks)` — should log "object" and "true"

---

### Learning Points

**1. Variable shadowing is a silent and dangerous bug**
Using `let` or `const` inside a function with the same name as an outer variable
creates a completely separate variable. JavaScript will NOT warn you about this.
The outer variable appears unchanged to code outside the function.

**2. Naming matters — use distinct names for local variables**
If you are creating a new single task object, call it `newTask` not `tasks`.
The name should reflect what it IS, not what category it belongs to.

**3. Use `const` for variables that should not be reassigned**
The global `tasks` array should be declared with `let` (since it gets
reassigned in `initApp`). But inside functions, using `const` for new
objects forces you to think about whether you mean to create a new binding
or modify an existing one.

**4. Understand the difference between reassigning and mutating**
- `tasks = something` — reassigns (replaces) the variable
- `tasks.push(something)` — mutates (modifies) the existing array
This distinction is crucial for working with shared state.

---

## Reflection Questions

**How did the AI's explanation compare to documentation found online?**
The AI explained the concept of variable shadowing in plain English
tied directly to the specific code. Online documentation covers scoping
in general terms — the AI connected the concept directly to the bug at hand,
which was much faster to understand.

**What would have been difficult to diagnose manually?**
The bug is invisible at first glance. Both the global and local variables
are called `tasks` — the eye skips over the `let` keyword without registering
that it creates a completely new binding. A developer might spend a long time
adding console.log statements in the wrong places before spotting it.

**How would I improve error messages in future code?**
Add type checking at the start of displayTasks:
```javascript
function displayTasks() {
  if (!Array.isArray(tasks)) {
    console.error('displayTasks error: tasks is not an array. Got:', typeof tasks, tasks);
    return;
  }
  // rest of function
}
```
This would immediately surface the problem with a meaningful message.

**Did the AI help understand underlying concepts, not just the fix?**
Yes — learning about variable shadowing as a concept means I can now
recognise this pattern in any language, not just fix this one specific bug.
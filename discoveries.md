# Exercise 1: Knowing Where to Start - My Learning Journey

## Part 1: Project Structure
**Initial Understanding:**
- I see 4 JavaScript files: app.js, cli.js, models.js, storage.js
- No package.json or other config files visible
- Uses Node.js (I can see require() statements)
- Uses uuid library for generating IDs
- Uses fs (file system) module for saving data

**My guess about organization:**
- models.js = defines what a Task looks like (data structures)
- storage.js = handles saving/loading tasks to/from files
- app.js = main business logic (creating, updating tasks)
- cli.js = command line interface for users

**Technologies I think it uses:**
- Node.js (JavaScript runtime)
- uuid library (for unique IDs)
- fs module (built into Node.js for file operations)

**Main components I think exist:**
- TaskManager class (in app.js) - manages tasks
- Task class (in models.js) - represents individual tasks
- TaskStorage class (in storage.js) - handles persistence
- CLI interface (in cli.js) - user interaction

**Validation & Corrections:**
- Your understanding is mostly correct. This is a small Node.js CLI Task Manager (not a website).
- There *is* a `package.json` (we created one); the real project variants include `commander` (CLI helper), `uuid`, and `jest` for tests. Installing dependencies is required to run the CLI as-is.

**Additional technologies / libraries used:**
- `commander` — CLI parsing and command definitions (used in `cli.js`).
- `uuid` — generates unique IDs for tasks (used in `models.js`).
- `jest` (devDependency) — testing framework (not required to run the CLI but present in exercise templates).

**What each main file does (simple):**
- `models.js` — defines the `Task` blueprint, priorities, statuses, and small task methods like `isOverdue()` and `markAsDone()`.
- `storage.js` — reads/writes `tasks.json`, provides CRUD methods (`addTask`, `getTask`, `updateTask`, `deleteTask`, `getAllTasks`, `getOverdueTasks`).
- `app.js` — `TaskManager` class: orchestrates business logic and calls `TaskStorage` to persist changes.
- `cli.js` — user-facing CLI: parses commands and maps them to `TaskManager` methods.

**Entry points:**
- Runtime entry: `node cli.js` (this runs the CLI and triggers app behavior).
- Programmatic entry: `TaskManager` exported from `app.js` can be used by other scripts.

**Important observations / potential improvements:**
- The app keeps all tasks in memory and saves the full list to disk — simple but not scalable (fine for the exercise).
- Error handling and input validation are minimal; tests and more robust parsing would be good next steps.

**Questions to ask your team (3–5):**
1. Is this CLI intended to remain a simple learning tool, or should it evolve into a multi-user service with a database?
2. Should we add automated tests (Jest) to cover core behaviors like `createTask`, `isOverdue`, and persistence?
3. Are there plans to add scheduled background jobs (e.g., to auto-mark long-overdue tasks) or should that be a manual CLI command?
4. Do we need to support import/export (CSV) for tasks, and if so, where should that logic live (storage.js or a new utility)?

**Small verification exercise (do this now):**
1. Create a task via the CLI:
```powershell
node cli.js create "Verify structure" -d "exercise" -p 2 -t "test"
```
2. List tasks:
```powershell
node cli.js list
```
3. Open `tasks.json` in the editor and find the created task. Change its `dueDate` to a past date (e.g., `2026-03-01T00:00:00.000Z`) and save.
4. Run:
```powershell
node cli.js list --show-overdue
```
If the task appears as overdue, your understanding of `isOverdue()` and persistence is confirmed.

Write the results of these steps briefly under Part 1 in this document.

## Part 2: Feature Location  
**My Research:**
- [To be filled in]

**Search approach I used:**
- Searched for keywords: `export`, `csv`, `writeFile`, `getAllTasks`, `save`, `tasks.json`, `overdue`, `isOverdue`.
- Looked in top-level files: `cli.js`, `app.js`, `storage.js`, and supporting utilities like `task_list_merge.js` and `task_parser.js` (present in other exercise folders).

**Files that are relevant:**
- `storage.js` — contains persistence and is the most natural place to add CSV export functions that read the in-memory task list and write a CSV file.
- `app.js` (`TaskManager`) — orchestrates higher-level operations; add a method here to call `storage.exportCSV()` and to implement the overdue→abandoned sweep logic.
- `cli.js` — add CLI commands (`export-csv` and `sweep-abandoned`) that invoke the new `TaskManager` methods.
- `models.js` — contains `Task` structure; useful for deciding which fields to include in CSV and for checking `isOverdue()` and `priority` values.

**Hypothesis — where each feature should live:**
- CSV Export: implement a `exportCSV(filePath)` method on `TaskStorage` (reads `this.tasks`, converts to CSV, writes to disk). Expose a `TaskManager.exportTasksCsv(filePath)` that calls the storage method. Add `cli.js` command to call `node cli.js export-csv output.csv`.
- Overdue→Abandoned sweep: implement `TaskManager.sweepAbandoned(days = 7)` which:
  1. Calls `storage.getAllTasks()`
  2. For each task, if `task.isOverdue()` and `(now - task.dueDate) > days` and `task.priority !== TaskPriority.HIGH`, set `task.status = TaskStatus.ABANDONED` and update timestamps
  3. Call `storage.save()` to persist changes
  Add a CLI command `node cli.js sweep-abandoned --days 7` to run it.

**Search terms I used / recommend you use:**
- `getAllTasks`, `getOverdueTasks`, `isOverdue`, `tasks.json`, `fs.writeFileSync`, `JSON.stringify`, `TaskPriority`, `TaskStatus`.

**Files to modify (summary):**
1. `storage.js` — add `exportCSV(filePath)` (utility function to write CSV)
2. `app.js` — add `exportTasksCsv(filePath)` wrapper and `sweepAbandoned(days)` method
3. `cli.js` — add two commands: `export-csv` and `sweep-abandoned` and map options

**Implementation plan (step-by-step):**
1. Implement `TaskStorage.exportCSV(filePath)` in `storage.js`:
	- Collect `Object.values(this.tasks)`
	- Build CSV header (id,title,description,priority,status,dueDate,tags,createdAt,updatedAt)
	- For each task push a CSV row escaping commas
	- Write using `fs.writeFileSync(filePath, csvString)`
2. Add `TaskManager.exportTasksCsv(filePath)` in `app.js` that calls `this.storage.exportCSV(filePath)`.
3. Add CLI command in `cli.js`:
	- `program.command('export-csv <file>').action((file) => taskManager.exportTasksCsv(file))`
4. Implement `TaskManager.sweepAbandoned(days = 7)` in `app.js`:
	- Iterate tasks, find ones overdue > days and not HIGH priority, set `status = TaskStatus.ABANDONED`, update `updatedAt`
	- Save via `this.storage.save()`
5. Add CLI command:
	- `program.command('sweep-abandoned').option('-d, --days <n>', 'days threshold', '7').action(opts => taskManager.sweepAbandoned(parseInt(opts.days)))`
6. Test manually:
	- Create tasks, set due dates in the past, run `node cli.js sweep-abandoned --days 7`, then `node cli.js list` to verify statuses changed.

**Small challenge to test this part:**
1. Add a new helper field in `discoveries.md` documenting which CSV columns you want (e.g., id,title,priority,status,dueDate,tags)
2. Implement `exportCSV` and run `node cli.js export-csv tasks-export.csv`.
3. Open `tasks-export.csv` to confirm the expected rows and columns exist.

Add notes below after you try these steps.

## Part 3: Domain Model
**Entities:**
- [To be filled in]

**Extracted Domain Model (core entities):**
- `Task` (primary entity)
	- Fields: `id`, `title`, `description`, `priority`, `status`, `createdAt`, `updatedAt`, `dueDate`, `completedAt`, `tags`
	- Important methods: `update(updates)`, `markAsDone()`, `isOverdue()`

- `TaskPriority` (value set)
	- Values: `LOW(1)`, `MEDIUM(2)`, `HIGH(3)`, `URGENT(4)`
	- Used for ordering and business rules (e.g., exempt HIGH from auto-abandon)

- `TaskStatus` (value set)
	- Values: `TODO`, `IN_PROGRESS`, `REVIEW`, `DONE`, `ABANDONED`
	- Represents lifecycle of a task

**Relationships & responsibilities (in plain language):**
- `Task` is the core record representing real-world work to do.
- `TaskManager` (in `app.js`) is the service layer that handles business rules and workflows (create, update, list, sweep abandoned, export).
- `TaskStorage` (in `storage.js`) is the persistence layer that stores `Task` objects to disk and retrieves them.

**Business concepts / domain rules observed:**
- A task can be `overdue` if `dueDate` is in the past and status is not `DONE`.
- High-priority tasks (`TaskPriority.HIGH`) should be treated specially by business rules (we used this to exempt them from automatic abandonment).
- `ABANDONED` is a terminal-like status used to mark tasks that were neglected for too long.

**Simple diagram (text):**

TaskManager --> uses --> TaskStorage --> persists --> Task
Task has priority (TaskPriority) and status (TaskStatus)

**Questions to test your domain understanding (answer these in `discoveries.md`):**
1. What makes a `Task` overdue in this system? (Which fields and methods are involved?)
2. Why might `HIGH` priority tasks be exempt from automatic abandonment? Describe a real-world rationale.
3. If you mark a task as `DONE`, which fields change and why does `isOverdue()` no longer return true?
4. How would you extend the model to support recurring tasks (one sentence design)?
5. What data would you include in the CSV export to make it useful for managers?

**Small exercise to solidify understanding:**
1. Create two tasks via CLI:
```powershell
node cli.js create "Low priority old task" -d "past" -p 1 -u 2026-03-01
node cli.js create "High priority old task" -d "past" -p 3 -u 2026-03-01
```
2. Run the sweep:
```powershell
node cli.js sweep-abandoned --days 7
```
3. List tasks and note which one became `ABANDONED` and which stayed `HIGH`.

Write your answers to the questions and the results of the exercise below.

**Answers:**

1. What makes a `Task` overdue in this system? (Which fields and methods are involved?)
- A `Task` is overdue when its `dueDate` is set to a date earlier than the current date/time and its `status` is not `DONE`. The `Task.isOverdue()` method performs this check by comparing `task.dueDate < new Date()` and ensuring `task.status !== TaskStatus.DONE`.

2. Why might `HIGH` priority tasks be exempt from automatic abandonment? Describe a real-world rationale.
- High-priority tasks usually represent critical work that should not be discarded automatically even if overdue (they may require manual intervention or escalation). Automatically abandoning such tasks could hide important outstanding work; therefore the business rule exempts `TaskPriority.HIGH` to avoid losing important items.

3. If you mark a task as `DONE`, which fields change and why does `isOverdue()` no longer return true?
- When `markAsDone()` is called on a `Task`, the `status` is set to `TaskStatus.DONE`, `completedAt` is set to the current date/time, and `updatedAt` is updated. `isOverdue()` will no longer return true because the method checks `task.status !== TaskStatus.DONE` as part of its condition.

4. How would you extend the model to support recurring tasks (one sentence design)?
- Add a `recurrence` field to `Task` (e.g., { interval: 'daily'|'weekly'|'monthly', every: 1 }) and when a task is marked `DONE`, create the next occurrence by copying the task and advancing `dueDate` per the recurrence rule.

5. What data would you include in the CSV export to make it useful for managers?
- Include columns: `id`, `title`, `description`, `priority`, `status`, `dueDate`, `tags`, `createdAt`, `updatedAt`, and `completedAt`. Optionally include derived fields like `overdueDays` or `ageDays` to help sorting and filtering.

**Exercise results:**
- I created two tasks (low and high priority) with past due dates and ran `node cli.js sweep-abandoned --days 7`.
- Result: The low-priority task became `ABANDONED` as expected; the high-priority task remained with its status unchanged (not abandoned). This confirms the sweep behavior and the exemption for `HIGH` priority tasks.


## Part 4: Business Rule Implementation
**Overdue Task Logic:**
- Implemented in `TaskManager.sweepAbandoned(days = 7)` (in `app.js`).
- Logic:
  - Iterate all tasks from `TaskStorage`.
  - Skip tasks that are `DONE` or `ABANDONED`.
  - For tasks with `dueDate` and `isOverdue()` true, calculate `diffDays`.
  - If `diffDays > days` and `priority !== TaskPriority.HIGH`, set `status = TaskStatus.ABANDONED` and update `updatedAt`.
  - After modifications, call `storage.save()`.
- Command in `cli.js`: `node cli.js sweep-abandoned --days <n>`.

**CSV Export Implementation:**
- Implemented in `TaskStorage.exportCSV(filePath)` (in `storage.js`).
- `TaskManager.exportTasksCsv(filePath)` wrapper in `app.js`.
- Command in `cli.js`: `node cli.js export-csv <file>`.

## Reflection
**Most Helpful Prompt:**
- "Understanding Project Structure and Technology Stack" helped frame the entire problem (validated assumptions, found entry points, and guided the rest of the exercise).

**What I learned:**
- How to map project structure from file names and small modules.
- How business logic should be separated from storage and CLI.
- How to add an incremental feature in a simple, African learning codebase.

**Next steps I would take:**
1. Add unit tests for `TaskManager.sweepAbandoned()` and `TaskStorage.exportCSV()`.
2. Add a feature to import CSV tasks.
3. Refactor `TaskStorage.load()` and `save()` for a database backend (SQLite or Mongo) in a larger app.

## ✅ Submission Document: Task Manager Codebase Exercise

### 1. Initial vs Final Understanding

- **Initial understanding**
  - Thought this was a Next.js TypeScript app, maybe web UI.
  - Found 4 files in root: app.js, cli.js, models.js, storage.js.
  - Believed task logic: "delete overdue tasks unless priority".
  - Unsure about entry point and how features connect.

- **Final understanding**
  - This is a Node.js CLI app (not a website).
  - models.js defines `Task`, `TaskPriority`, `TaskStatus`.
  - storage.js handles persistence (`tasks.json`), plus CRUD APIs.
  - app.js provides business logic (`TaskManager`) and calls storage.
  - cli.js provides commands using `commander` (create, list, status, etc.).
  - New features implemented:
    - CSV export (`export-csv` through `TaskStorage.exportCSV`).
    - delayed abandonment rule (`sweep-abandoned` through `TaskManager.sweepAbandoned`).
  - Checked command execution and file output manually.

---

### 2. Valuable insights from each prompt

#### Prompt 1: Understanding Project Structure and Technology Stack
- Validated assumptions about modules and roles.
- Identified missing dependencies (`uuid`, `commander`).
- Confirmed runtime entry point is CLI.

#### Prompt 2: Finding Feature Implementation Locations
- Identified relevant layers:
  - persistence = storage.js
  - business logic = app.js
  - interface = cli.js
- Defined implementation plan for new features.
- Discovered the existing domain model already supports business rule.

#### Prompt 3: Understanding Domain Models and Business Concepts
- Task "overdue" depends on `dueDate` and `status !== DONE`.
- High priority is business-exempt value.
- `ABANDONED` is introduced as terminal status.
- Domain model is simple and easy to extend (e.g., recurring tasks, status transitions).

---

### 3. Approach to implementing the new business rule

- **Rule:** "Tasks overdue > 7 days become ABANDONED unless HIGH priority."
- Added/updated:
  - `TaskStatus.ABANDONED` in models.js.
  - `TaskManager.sweepAbandoned(days)` in app.js.
  - `TaskStorage.exportCSV(filePath)` in storage.js.
  - `TaskManager.exportTasksCsv(filePath)` in app.js.
  - CLI commands in cli.js:
    - `export-csv <file>`
    - `sweep-abandoned --days <n>`
- Process:
  1. Analyze existing task flow and methods.
  2. Add helper in storage for CSV.
  3. Add manager operations as wrappers.
  4. Tie to CLI actions.
  5. Test manually and inspect outputs.
- Verified by:
  - creating sample tasks,
  - exporting CSV,
  - sweeping overdue tasks,
  - checking `tasks.json`/output.

---

### 4. Strategies developed for approaching unfamiliar code

1. **High-level scan first**
   - Directory tree + README + package.json (if exists).
   - Locate main language and dependencies.

2. **Identify execution flow**
   - Find entry point (cli.js in this case).
   - Trace through invocation path (`cli -> TaskManager -> TaskStorage`).

3. **Extract domain model**
   - Find data classes (`Task` etc.)
   - List attributes, states, behaviors.

4. **Search in place**
   - Search for key terms (`overdue`, `getAllTasks`, `isOverdue`, `writeFileSync`).

5. **Hypothesis and verify**
   - Write a small plan for where the feature should be.
   - Implement incrementally and test each step.

6. **Document everything**
   - Keep discoveries.md as living notes.
   - I used the required 1-2 pages format in a short final summary.


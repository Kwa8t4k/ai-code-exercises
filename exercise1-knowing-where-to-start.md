# Exercise 1: Knowing Where to Start

## Part 1: Understanding Project Structure

### My Initial Understanding (Before AI)
- The project appears to be a Task Manager application
- It lets users create, prioritize, update, and complete tasks
- The code is organized around models (data) and services (logic)
- Technologies used: Python (or JavaScript/Java - I chose Python)

### After Applying the AI Prompt — What I Discovered
- Core files: `task.py`, `task_manager.py`, `task_priority.py`, `task_status.py`
- The application entry point is `main.py` (or the main task manager file)
- No external frameworks — uses Python's built-in libraries
- Data is stored in memory using a dictionary keyed by task ID
- `TaskPriority` and `TaskStatus` are enums (fixed sets of values)

### Misconceptions I Had Corrected
- I assumed there would be a database — actually uses in-memory storage
- I assumed there would be a web interface — it is a library/service-style app

### Questions I Would Ask My Team
1. Is there persistent storage planned, or is in-memory intentional?
2. Are there automated tests for this project?
3. What is the primary use case — standalone app or imported as a library?
4. Are there any known bugs or areas marked for improvement?
5. Is there a preferred coding style or conventions guide for this project?

---

## Part 2: Finding the "Task Export to CSV" Feature

### My Search Approach
- Searched for keywords: `export`, `csv`, `file`, `write`, `save`
- Checked utility/helper files for any file I/O operations
- Looked at the Task model to understand what fields exist

### Where I Would Implement It
A new file: `task_exporter.py` containing an `export_to_csv(tasks)` function.

### Files That Would Need to Change
- **New file:** `task_exporter.py` — contains the export logic
- **`main.py` or equivalent:** Add a menu option to trigger the export
- **No changes needed** to the Task model itself

### My Implementation Plan
1. Import Python's built-in `csv` module
2. Define column headers matching Task fields: title, priority, status, due_date, tags
3. Loop through all tasks and write each as a row
4. Save to a file named `tasks_export.csv`

---

## Part 3: Understanding the Domain Model

### Core Entities

| Entity | What It Represents |
|--------|--------------------|
| `Task` | A single unit of work — the main object in the system |
| `TaskPriority` | An enum: LOW, MEDIUM, HIGH, URGENT |
| `TaskStatus` | An enum: TODO, IN_PROGRESS, REVIEW, DONE |

### Entity Relationship Diagram (described in text)
```
Task
 ├── priority → TaskPriority (one value: LOW/MEDIUM/HIGH/URGENT)
 ├── status   → TaskStatus (one value: TODO/IN_PROGRESS/REVIEW/DONE)
 ├── tags     → list of strings (e.g. ["blocker", "urgent"])
 ├── due_date → optional datetime
 ├── created_at → datetime (set on creation, never changes)
 └── updated_at → datetime (updates every time task is modified)
```

### Domain Glossary
- **Overdue:** A task whose due_date has passed and is not DONE
- **Blocker:** A tag meaning this task is blocking other work
- **Score:** A calculated number used to rank tasks by importance
- **Abandoned:** A task that is overdue and no longer being worked on

### Answers to AI Test Questions
1. *Why does DONE reduce a task's score by 50?*
   Because completed tasks should move to the bottom of priority lists. They need no action.
2. *What happens if an URGENT task is marked DONE?*
   URGENT gives +60 score, DONE subtracts 50, net = +10. Still ranked low overall — correct behavior since it's finished.
3. *Why are tags merged using a union during sync?*
   Both local and remote sources may have independently added different tags. Using a union means no tags are lost from either side.

---

## Part 4: Implementing the New Business Rule

**Rule:** Tasks overdue by more than 7 days should be automatically marked as "abandoned"
unless they are HIGH or URGENT priority.

### Files I Would Modify
- `task_status.py` — Add `ABANDONED` as a new status value
- `task_manager.py` — Add a new method `check_and_abandon_overdue_tasks()`
- `main.py` — Call this method on application start or on a schedule

### Pseudocode
```
function check_and_abandon_overdue_tasks(all_tasks):
    today = current date
    for each task in all_tasks:
        if task.status is not DONE and task.status is not ABANDONED:
            if task.due_date is not None:
                days_overdue = (today - task.due_date).days
                if days_overdue > 7:
                    if task.priority not in [HIGH, URGENT]:
                        task.status = ABANDONED
                        task.updated_at = now
```

### Questions I Would Ask My Team Before Implementing
1. Should this run automatically on startup, or only when manually triggered?
2. Should users receive a notification when a task is auto-abandoned?
3. Can an abandoned task be manually reactivated?
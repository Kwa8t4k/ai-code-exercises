# Exercise 2: Code Understanding Journal

## Part 1: Task Creation & Status Updates

### Files Involved
- `task.py` — Defines the Task class and its fields
- `task_manager.py` — Contains methods to create and update tasks
- `task_status.py` — Defines the TaskStatus enum

### Execution Flow When a Task is Created
1. User provides a task title (and optionally priority, due date, tags)
2. A new `Task` object is instantiated with:
   - A unique generated ID
   - Status defaulting to `TODO`
   - `created_at` and `updated_at` both set to the current time
3. The task is stored in the TaskManager's internal dictionary: `{task_id: task}`
4. The task object is returned as confirmation

### How Data is Stored
Tasks are stored in a Python dictionary (or equivalent) keyed by task ID.
This allows O(1) — instant — lookup of any task by its ID.

### Design Pattern Discovered
The **Entity + Service pattern**: The Task class only holds data.
All logic (creating, updating, searching) lives in a separate TaskManager/service class.
This keeps data and behavior cleanly separated.

### 3 Validation Experiments I Could Run
1. Create a task and print its `status` — should be `TODO`
2. Create a task without a due_date — confirm `due_date` is `None`, not an error
3. Update a task title and confirm `updated_at` changed but `created_at` did not

---

## Part 2: Task Prioritization System

### My Initial Understanding
I thought priority was a simple ranking (LOW < MEDIUM < HIGH < URGENT)
and tasks were sorted directly by this field.

### What the Guided Questions Helped Me Discover

**Q: How does the code decide which of two HIGH priority tasks comes first?**
I found `calculateTaskScore()` — priority is just ONE factor.
Due date proximity, tags like "blocker", and recency also contribute to the final score.
Two HIGH priority tasks can rank very differently.

**Q: What happens to an overdue task's score?**
Overdue tasks receive +35 bonus points on top of their priority score.
This means an overdue LOW task (10 + 35 = 45) can outrank
a non-urgent MEDIUM task (20). Intentional — overdue tasks need attention.

**Q: Why might an URGENT task rank lower than expected?**
If it's marked DONE, it receives -50 penalty. Even URGENT tasks sink to the
bottom when completed. This is correct behavior.

### Key Insight
Priority is not a simple A/B/C sort. It's a **weighted multi-factor scoring system**
that models real human judgment about what to work on next.

### My Initial Understanding vs. Final Understanding
- Before: I thought priority field = sort order
- After: I understand it's a formula combining urgency, time pressure, tags, and activity

---

## Part 3: Data Flow — Marking a Task Complete

### Complete Data Flow Diagram
```
User says: "Mark task X as complete"
         ↓
1. Look up task by ID in the tasks dictionary
         ↓
2. Validate: Does task exist? Is it already DONE?
         ↓
3. Update fields:
   - task.status     → DONE
   - task.completed_at → current timestamp
   - task.updated_at   → current timestamp
         ↓
4. Save updated task back into dictionary
         ↓
5. (If sync is active) → Flag task in toUpdateRemote list
         ↓
6. Return updated task / success confirmation
```

### State Changes That Occur
| Field | Before | After |
|-------|--------|-------|
| `status` | TODO / IN_PROGRESS / REVIEW | DONE |
| `completed_at` | None | current datetime |
| `updated_at` | previous timestamp | current datetime |

### Potential Points of Failure
- Task ID not found (task was deleted before completion)
- Task already DONE (idempotency — what happens if you complete it twice?)
- Sync failure — local updated successfully but remote update fails

### How Changes Are Persisted
In the current implementation, changes are stored in memory only.
If the application restarts, all changes are lost unless file/database storage is added.
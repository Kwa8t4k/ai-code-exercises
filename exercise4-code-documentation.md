# Exercise 4: Code Documentation

## Code Selected
Algorithm 1: calculateTaskScore() — Task Priority Sorting
Language: JavaScript
Source: Task Manager project (same codebase from Code Comprehension exercises)

---

## Original Code
````javascript
function calculateTaskScore(task) {
  const priorityWeights = {
    [TaskPriority.LOW]: 1,
    [TaskPriority.MEDIUM]: 2,
    [TaskPriority.HIGH]: 3,
    [TaskPriority.URGENT]: 4
  };

  let score = (priorityWeights[task.priority] || 0) * 10;

  if (task.dueDate) {
    const now = new Date();
    const dueDate = new Date(task.dueDate);
    const daysUntilDue = Math.ceil((dueDate - now) / (1000 * 60 * 60 * 24));

    if (daysUntilDue < 0) {
      score += 30;
    } else if (daysUntilDue === 0) {
      score += 20;
    } else if (daysUntilDue <= 2) {
      score += 15;
    } else if (daysUntilDue <= 7) {
      score += 10;
    }
  }

  if (task.status === TaskStatus.DONE) {
    score -= 50;
  } else if (task.status === TaskStatus.REVIEW) {
    score -= 15;
  }

  if (task.tags.some(tag => ["blocker", "critical", "urgent"].includes(tag))) {
    score += 8;
  }

  const now = new Date();
  const updatedAt = new Date(task.updatedAt);
  const daysSinceUpdate = Math.floor((now - updatedAt) / (1000 * 60 * 60 * 24));
  if (daysSinceUpdate < 1) {
    score += 5;
  }

  return score;
}
````

---

## Prompt 1 Output: Comprehensive JSDoc Documentation

The following documentation was generated using the Comprehensive Function Documentation prompt.
````javascript
/**
 * Calculates a numerical priority score for a task based on multiple weighted factors.
 *
 * The score determines how urgently a task needs attention, combining:
 * - Base priority level (LOW / MEDIUM / HIGH / URGENT)
 * - Due date proximity (overdue tasks score higher)
 * - Current status (completed tasks score lower to move them to the bottom)
 * - Special tags (blocker, critical, urgent add bonus points)
 * - Recent activity (tasks updated in the last 24 hours receive a small boost)
 *
 * Higher scores indicate tasks that should appear first in a prioritized list.
 *
 * @param {Object} task - The task object to score
 * @param {string} task.priority - The task priority level.
 *   Must be one of: TaskPriority.LOW, TaskPriority.MEDIUM,
 *   TaskPriority.HIGH, TaskPriority.URGENT
 * @param {string|Date|null} task.dueDate - The task due date.
 *   Can be a Date object, ISO date string, or null if no due date is set
 * @param {string} task.status - The current task status.
 *   Must be one of: TaskStatus.TODO, TaskStatus.IN_PROGRESS,
 *   TaskStatus.REVIEW, TaskStatus.DONE
 * @param {string[]} task.tags - Array of tag strings associated with the task.
 *   Tags "blocker", "critical", and "urgent" trigger a score bonus
 * @param {string|Date} task.updatedAt - Timestamp of the last task update.
 *   Used to detect recently active tasks
 *
 * @returns {number} A numerical score representing task priority.
 *   Typical range is -10 to 63, though edge cases may fall outside this range.
 *   Higher values mean higher priority.
 *
 * @example
 * // High priority task due tomorrow with a blocker tag
 * const task = {
 *   priority: TaskPriority.HIGH,
 *   dueDate: new Date(Date.now() + 24 * 60 * 60 * 1000), // tomorrow
 *   status: TaskStatus.IN_PROGRESS,
 *   tags: ["blocker"],
 *   updatedAt: new Date() // updated today
 * };
 * const score = calculateTaskScore(task);
 * // score = 30 (HIGH) + 15 (due in 1-2 days) + 8 (blocker tag) + 5 (recent) = 58
 *
 * @example
 * // Completed low priority task — should appear at bottom of list
 * const completedTask = {
 *   priority: TaskPriority.LOW,
 *   dueDate: null,
 *   status: TaskStatus.DONE,
 *   tags: [],
 *   updatedAt: new Date("2024-01-01")
 * };
 * const score = calculateTaskScore(completedTask);
 * // score = 10 (LOW) + 0 (no due date) - 50 (DONE) = -40
 *
 * @throws {TypeError} May throw if task object is null or missing required fields,
 *   as property access on undefined will cause a runtime error.
 *   Always ensure a valid task object is passed.
 *
 * @see {@link sortTasksByImportance} for using this score to sort a task list
 * @see {@link getTopPriorityTasks} for retrieving the highest-scored tasks
 */
function calculateTaskScore(task) {
  // Priority weight map: each level maps to a multiplier
  // Multiplied by 10 to give room for bonus/penalty adjustments
  const priorityWeights = {
    [TaskPriority.LOW]: 1,       // 10 base points
    [TaskPriority.MEDIUM]: 2,    // 20 base points
    [TaskPriority.HIGH]: 3,      // 30 base points
    [TaskPriority.URGENT]: 4     // 40 base points
  };

  // Start with base score from priority (defaults to 0 if priority is unrecognised)
  let score = (priorityWeights[task.priority] || 0) * 10;

  // Add due date urgency bonus — tasks due sooner score higher
  if (task.dueDate) {
    const now = new Date();
    const dueDate = new Date(task.dueDate);
    const daysUntilDue = Math.ceil((dueDate - now) / (1000 * 60 * 60 * 24));

    if (daysUntilDue < 0) {
      score += 30;       // Overdue: highest urgency bonus
    } else if (daysUntilDue === 0) {
      score += 20;       // Due today
    } else if (daysUntilDue <= 2) {
      score += 15;       // Due in 1-2 days
    } else if (daysUntilDue <= 7) {
      score += 10;       // Due this week
    }
    // Due in more than 7 days: no bonus
  }

  // Apply status penalty — reduce score for tasks that need less attention
  if (task.status === TaskStatus.DONE) {
    score -= 50;         // Completed tasks sink to the bottom
  } else if (task.status === TaskStatus.REVIEW) {
    score -= 15;         // Nearly done, slightly deprioritised
  }

  // Boost score for tasks with urgent-indicating tags
  if (task.tags.some(tag => ["blocker", "critical", "urgent"].includes(tag))) {
    score += 8;
  }

  // Boost recently updated tasks — keeps active work visible
  const now = new Date();
  const updatedAt = new Date(task.updatedAt);
  const daysSinceUpdate = Math.floor((now - updatedAt) / (1000 * 60 * 60 * 24));
  if (daysSinceUpdate < 1) {
    score += 5;          // Updated within last 24 hours
  }

  return score;
}
````

---

## Prompt 2 Output: Intent and Logic Explanation

The following was generated using the Intent and Logic Explanation prompt.

### High-Level Purpose
This function answers the question: "How urgently does this task need my attention right now?"
It does this by combining multiple factors into one number, allowing an entire task list to be
sorted by a single meaningful value rather than by any single field alone.

### Step-by-Step Logic Breakdown

**Step 1 — Establish a baseline from priority**
Priority is the foundation. URGENT tasks start with 4x the score of LOW tasks.
This ensures that, all else being equal, more important tasks rank higher.

**Step 2 — Adjust for time pressure**
A task due today is more urgent than one due next month, regardless of its stated priority.
The function adds bonus points based on how soon the task is due, with the largest bonus
going to overdue tasks. This means a forgotten LOW priority task that is 10 days overdue
will outrank a MEDIUM priority task with no deadline.

**Step 3 — Penalise tasks that need less attention**
DONE tasks receive a large penalty (-50) so they move to the bottom of any sorted list.
REVIEW tasks receive a smaller penalty (-15) because they are close to done but still
need some attention. This prevents completed work from cluttering active task views.

**Step 4 — Recognise urgent signals in tags**
Tags like "blocker", "critical", and "urgent" are signals added by humans to indicate
something important. The function respects this by adding 8 bonus points when these
tags are present.

**Step 5 — Reward recently active tasks**
A small +5 bonus for tasks updated in the last 24 hours keeps recently touched tasks
visible. This reflects the real-world pattern of working on something and wanting
it to stay near the top of the list.

### Assumptions and Edge Cases Identified

| Assumption / Edge Case | Description |
|------------------------|-------------|
| Unknown priority | If task.priority is not in the weight map, score defaults to 0. No error is thrown but the task effectively has no priority score. |
| Null dueDate | Safely handled — the due date block is skipped entirely. |
| Multiple urgent tags | Only one check is performed using .some(). Having multiple urgent tags still only adds 8 points, not 8 per tag. |
| Same updatedAt timestamp | If updatedAt equals the current time exactly, daysSinceUpdate = 0 and the boost applies. |
| No upper score cap | A task could theoretically accumulate all bonuses: 40 + 30 + 8 + 5 = 83 before penalties. |

### Potential Improvements Identified
1. The bonus and penalty numbers are hardcoded magic numbers.
   They should be extracted into named constants or a configuration object
   so they can be adjusted without changing the function logic.
2. The unrecognised priority case silently scores 0.
   A console warning or fallback to MEDIUM would make this more robust.
3. The recency boost is a flat +5 for any update in 24 hours.
   A more gradual decay (more recent = higher boost) would produce
   more nuanced rankings.

---

## Final Combined Documentation Version

The best approach combines the precise JSDoc from Prompt 1 with the inline
comments and edge case table from Prompt 2.

The final version adds:
- Complete JSDoc header with all @param, @returns, @example, @throws tags
- Inline comments explaining WHY each score adjustment exists, not just what it does
- The edge case table added as a @notes section in the JSDoc

This combined version is what I would commit to a production codebase.

---

## Reflection

### Which parts were most challenging for the AI?
The AI struggled slightly with the @throws documentation because the function
does not explicitly throw errors — it would only throw implicitly if a null task
was passed. Clarifying this in my prompt improved the output.

### What additional information did I need to provide?
I needed to specify JSDoc format explicitly. Without that, the AI defaulted to
a generic comment block. I also specified "include edge cases and example usage"
to avoid getting a minimal documentation response.

### How would I use this in my own projects?
I would use Prompt 1 at the time of writing a function to generate the initial
JSDoc block, then use Prompt 2 during code review to identify any undocumented
assumptions or edge cases I missed. This two-prompt approach catches things that
a single pass would miss.
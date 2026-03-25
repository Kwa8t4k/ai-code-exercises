# Exercise 10: Using AI to Help with Testing

## Function Selected
calculateTaskScore() — JavaScript implementation
From the Task Manager project used throughout this course.

---

## The Function Being Tested
```javascript
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

    if (daysUntilDue < 0) score += 30;
    else if (daysUntilDue === 0) score += 20;
    else if (daysUntilDue <= 2) score += 15;
    else if (daysUntilDue <= 7) score += 10;
  }

  if (task.status === TaskStatus.DONE) score -= 50;
  else if (task.status === TaskStatus.REVIEW) score -= 15;

  if (task.tags.some(tag => ["blocker", "critical", "urgent"].includes(tag))) {
    score += 8;
  }

  const now = new Date();
  const updatedAt = new Date(task.updatedAt);
  const daysSinceUpdate = Math.floor((now - updatedAt) / (1000 * 60 * 60 * 24));
  if (daysSinceUpdate < 1) score += 5;

  return score;
}
```

---

# PART 1: Understanding What to Test

## Exercise 1.1: Behavior Analysis

### AI Conversation Summary

I used the Behavior Analysis Questions prompt with the calculateTaskScore function.

**AI asked me:** "What do you think this function is trying to accomplish?"

**My answer:** I think it calculates a number that represents how urgently
a task needs attention, so tasks can be sorted by priority.

**AI asked me:** "What factors does the function consider when calculating the score?"

**My answer:** Priority level, due date, status, tags, and recent activity.

**AI confirmed my answer and identified what I missed:**
I missed that the function has a fallback of 0 when priority is unrecognised
(the `|| 0` in the score calculation). This is an important edge case to test.

**AI asked me:** "What edge cases should be tested?"

**My answer:**
- Task with no due date
- Task that is overdue
- Task marked as DONE
- Task with urgent tags

**AI identified additional edge cases I missed:**
- Task with an unrecognised priority value (scores 0)
- Task with multiple urgent tags (should still only add 8 points once)
- Task with a due date exactly at midnight today (boundary condition)
- Task with updatedAt set to exactly 24 hours ago (boundary for recency boost)
- Task with empty tags array (no tag boost)
- Null or missing fields that could cause crashes

**AI asked me:** "Which test should you write first and why?"

**My answer:** The basic priority scoring test, because it is the foundation
that all other scoring builds on. If this is wrong, everything else is wrong.

**AI confirmed this was the right choice and explained:**
Start with the "happy path" — a complete, valid task with a known priority —
then add edge cases one at a time. This way each test builds on a confirmed foundation.

---

### Test Cases List (Identified From Conversation)

**Priority:** High → Low

1. **Basic priority scoring** — verify LOW=10, MEDIUM=20, HIGH=30, URGENT=40
2. **Overdue task bonus** — task 1 day past due date should get +30
3. **Due today bonus** — task due today should get +20
4. **Due in 1-2 days bonus** — should get +15
5. **Due this week bonus** — due in 3-7 days should get +10
6. **No due date** — score should only reflect priority (no bonus)
7. **DONE status penalty** — DONE task should lose 50 points
8. **REVIEW status penalty** — REVIEW task should lose 15 points
9. **Urgent tag boost** — task tagged "blocker" should gain +8
10. **Critical tag boost** — task tagged "critical" should gain +8
11. **Multiple urgent tags** — should only add +8 once, not per tag
12. **No matching tags** — no tag boost applied
13. **Recently updated boost** — updated within 24hrs should gain +5
14. **Not recently updated** — updated 2+ days ago should not gain +5
15. **Unknown priority** — unrecognised priority value should score 0 base

---

## Exercise 1.2: Test Planning

### Structured Test Plan

**Functions covered:** calculateTaskScore, sortTasksByImportance, getTopPriorityTasks

---

#### Priority 1 — Unit Tests (Test First)

These test a single function in isolation with controlled inputs.

| Test | Function | Input | Expected |
|------|----------|-------|----------|
| Base priority scores | calculateTaskScore | Task with each priority level, no due date, TODO status, no tags, old updatedAt | LOW=10, MEDIUM=20, HIGH=30, URGENT=40 |
| Overdue bonus | calculateTaskScore | Task due 5 days ago | base + 30 |
| Due today bonus | calculateTaskScore | Task due today | base + 20 |
| DONE penalty | calculateTaskScore | Completed task | base - 50 |
| REVIEW penalty | calculateTaskScore | Task in review | base - 15 |
| Blocker tag boost | calculateTaskScore | Task with "blocker" tag | base + 8 |
| Recency boost | calculateTaskScore | Task updated 1 hour ago | base + 5 |
| No due date | calculateTaskScore | Task without dueDate field | no date bonus |
| Unknown priority | calculateTaskScore | Task with priority = "EXTREME" | base = 0 |

---

#### Priority 2 — Boundary Tests (Test Second)

These test the exact edges between different score conditions.

| Test | Condition Being Tested |
|------|------------------------|
| Due in exactly 2 days | Should get +15, not +10 |
| Due in exactly 7 days | Should get +10, not 0 |
| Due in exactly 8 days | Should get 0 due date bonus |
| Updated exactly 24 hours ago | Should NOT get recency boost |
| Updated 23 hours 59 minutes ago | Should get recency boost |

---

#### Priority 3 — Integration Tests (Test Last)

These test multiple functions working together.

| Test | Functions | Scenario |
|------|-----------|----------|
| Sort order correctness | calculateTaskScore + sortTasksByImportance | Array of 5 mixed tasks should sort highest score first |
| Top N returns correct count | all three functions | getTopPriorityTasks with limit=3 on 10 tasks returns exactly 3 |
| Top N returns highest scores | all three functions | The 3 returned tasks should have higher scores than remaining 7 |
| Empty array handling | sortTasksByImportance | Empty array input returns empty array |
| Single task | getTopPriorityTasks | Array of 1 task with limit=5 returns just that 1 task |

---

#### Test Dependencies
- calculateTaskScore must be tested and passing before testing sort functions
- sortTasksByImportance must be tested before testing getTopPriorityTasks
- All unit tests must pass before running integration tests

---

# PART 2: Improving a Single Test

## Exercise 2.1: My First Test (Before Improvement)
```javascript
// Basic test I wrote first
test('calculates score for high priority task', () => {
  const task = {
    priority: 'HIGH',
    status: 'TODO',
    tags: [],
    dueDate: null,
    updatedAt: '2024-01-01'
  };

  const score = calculateTaskScore(task);
  expect(score).toBeGreaterThan(0);
});
```

### AI Feedback on This Test

**AI asked me:** "What exactly is your test verifying?"

My answer: That the function returns a positive number for a HIGH priority task.

**AI asked me:** "Is `toBeGreaterThan(0)` checking behavior or just that the function runs?"

I realised: It is really just checking the function does not crash or return zero.
It does not verify the CORRECT score at all. A broken function returning 999
would still pass this test.

**AI suggested:** Use `toBe(30)` with a precisely calculated expected value.
This locks in the exact behavior — any regression will immediately fail the test.

**AI asked:** "What edge cases does your test miss?"

I identified: No due date case, no tag case, old updatedAt — but I did not
verify these contribute 0 bonus, so I do not know if the base score is correct.

---

### My Improved Test (After AI Guidance)
```javascript
describe('calculateTaskScore — base priority scoring', () => {

  // Helper to create a controlled task with no bonuses or penalties
  function createBaseTask(priority) {
    return {
      priority: priority,
      status: TaskStatus.TODO,        // No status penalty
      tags: [],                       // No tag boost
      dueDate: null,                  // No due date bonus
      updatedAt: '2020-01-01'         // Old update — no recency boost
    };
  }

  test('LOW priority task scores 10 base points', () => {
    const task = createBaseTask(TaskPriority.LOW);
    expect(calculateTaskScore(task)).toBe(10);
  });

  test('MEDIUM priority task scores 20 base points', () => {
    const task = createBaseTask(TaskPriority.MEDIUM);
    expect(calculateTaskScore(task)).toBe(20);
  });

  test('HIGH priority task scores 30 base points', () => {
    const task = createBaseTask(TaskPriority.HIGH);
    expect(calculateTaskScore(task)).toBe(30);
  });

  test('URGENT priority task scores 40 base points', () => {
    const task = createBaseTask(TaskPriority.URGENT);
    expect(calculateTaskScore(task)).toBe(40);
  });

  test('unrecognised priority scores 0 base points', () => {
    const task = createBaseTask('EXTREME');
    expect(calculateTaskScore(task)).toBe(0);
  });
});
```

**Why this version is better:**
- Uses `toBe()` with exact expected values — a wrong result immediately fails
- The helper function isolates each test from unrelated bonuses
- Test names describe the exact behavior being verified, not just "it works"
- Covers the unrecognised priority edge case I had previously missed

---

## Exercise 2.2: Due Date Test

### My Pseudocode Idea Before AI Guidance
```
test: task with due date in the past should get bonus points
  - create task with dueDate = yesterday
  - call calculateTaskScore
  - expect score to include overdue bonus
```

### What AI Showed Me (Example of a Better Test)
```javascript
// AI explained: A good due date test needs to:
// 1. Use a FIXED date (not relative to today) or mock the Date object
//    so the test does not break as time passes
// 2. Test each boundary explicitly — not just "overdue"
// 3. Use exact expected values so regressions are caught immediately

describe('calculateTaskScore — due date bonuses', () => {

  // Use jest.useFakeTimers() to freeze time so tests are not date-dependent
  beforeEach(() => {
    jest.useFakeTimers();
    jest.setSystemTime(new Date('2025-01-15T12:00:00Z')); // Fixed "now"
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  function createTaskWithDueDate(dueDate) {
    return {
      priority: TaskPriority.LOW,      // 10 base points — easy to calculate
      status: TaskStatus.TODO,
      tags: [],
      dueDate: dueDate,
      updatedAt: '2020-01-01'
    };
  }

  test('overdue task (due yesterday) gets +30 bonus', () => {
    const task = createTaskWithDueDate('2025-01-14'); // 1 day ago
    expect(calculateTaskScore(task)).toBe(10 + 30); // 40
  });

  test('overdue task (due 30 days ago) still gets +30 bonus', () => {
    const task = createTaskWithDueDate('2024-12-16'); // 30 days ago
    expect(calculateTaskScore(task)).toBe(10 + 30); // 40
  });

  test('task due today gets +20 bonus', () => {
    const task = createTaskWithDueDate('2025-01-15'); // today
    expect(calculateTaskScore(task)).toBe(10 + 20); // 30
  });

  test('task due in 1 day gets +15 bonus', () => {
    const task = createTaskWithDueDate('2025-01-16');
    expect(calculateTaskScore(task)).toBe(10 + 15); // 25
  });

  test('task due in 2 days gets +15 bonus (boundary)', () => {
    const task = createTaskWithDueDate('2025-01-17');
    expect(calculateTaskScore(task)).toBe(10 + 15); // 25
  });

  test('task due in 7 days gets +10 bonus (boundary)', () => {
    const task = createTaskWithDueDate('2025-01-22');
    expect(calculateTaskScore(task)).toBe(10 + 10); // 20
  });

  test('task due in 8 days gets no due date bonus', () => {
    const task = createTaskWithDueDate('2025-01-23');
    expect(calculateTaskScore(task)).toBe(10); // base only
  });

  test('task with no due date gets no bonus', () => {
    const task = createTaskWithDueDate(null);
    expect(calculateTaskScore(task)).toBe(10); // base only
  });
});
```

**Key insight from AI:** Without mocking the date, tests that use
`new Date()` internally will produce different results depending on
WHEN you run them. A test written today could pass today and fail
next week because the daysUntilDue calculation changes. Always freeze
time in tests that involve date calculations.

---

# PART 3: Test-Driven Development Practice

## Exercise 3.1: TDD — New Feature (Assigned User Boost +12)

### TDD Process I Followed

**Step 1 — Write the failing test first (RED)**
```javascript
test('task assigned to current user gets +12 score boost', () => {
  const currentUserId = 'user-123';
  const task = {
    priority: TaskPriority.LOW,
    status: TaskStatus.TODO,
    tags: [],
    dueDate: null,
    updatedAt: '2020-01-01',
    assignedTo: 'user-123'           // Assigned to current user
  };

  const score = calculateTaskScore(task, currentUserId);
  expect(score).toBe(10 + 12); // base + assigned boost = 22
});
```

At this point the test FAILS because calculateTaskScore does not accept
a second parameter and has no assigned user logic.

**Step 2 — Write the MINIMUM code to make it pass (GREEN)**
```javascript
function calculateTaskScore(task, currentUserId = null) {
  const priorityWeights = {
    [TaskPriority.LOW]: 1,
    [TaskPriority.MEDIUM]: 2,
    [TaskPriority.HIGH]: 3,
    [TaskPriority.URGENT]: 4
  };

  let score = (priorityWeights[task.priority] || 0) * 10;

  // ... existing due date, status, tag, recency logic unchanged ...

  // New: assigned user boost
  if (currentUserId && task.assignedTo === currentUserId) {
    score += 12;
  }

  return score;
}
```

**Step 3 — Test passes. Now add the next test (next RED)**
```javascript
test('task assigned to a different user gets no assigned boost', () => {
  const task = {
    priority: TaskPriority.LOW,
    status: TaskStatus.TODO,
    tags: [],
    dueDate: null,
    updatedAt: '2020-01-01',
    assignedTo: 'user-456'           // Different user
  };

  const score = calculateTaskScore(task, 'user-123');
  expect(score).toBe(10); // base only — no assigned boost
});

test('task with no assignedTo field gets no assigned boost', () => {
  const task = {
    priority: TaskPriority.LOW,
    status: TaskStatus.TODO,
    tags: [],
    dueDate: null,
    updatedAt: '2020-01-01'
    // no assignedTo field
  };

  const score = calculateTaskScore(task, 'user-123');
  expect(score).toBe(10); // base only
});

test('existing tests still pass when currentUserId is not provided', () => {
  const task = {
    priority: TaskPriority.HIGH,
    status: TaskStatus.TODO,
    tags: [],
    dueDate: null,
    updatedAt: '2020-01-01'
  };

  // Called without second argument — same as before the feature was added
  const score = calculateTaskScore(task);
  expect(score).toBe(30); // HIGH base only — no regression
});
```

**Step 4 — All tests pass. Refactor if needed (REFACTOR)**

The implementation is clean and minimal — no refactoring needed.
The feature is backward-compatible: calling calculateTaskScore without
the second argument still works exactly as before.

---

## Exercise 3.2: TDD — Bug Fix (Days Since Update Calculation)

### Test That Reproduces the Bug
```javascript
test('task updated 25 hours ago should NOT get recency boost', () => {
  jest.useFakeTimers();
  jest.setSystemTime(new Date('2025-01-15T12:00:00Z'));

  const task = {
    priority: TaskPriority.LOW,
    status: TaskStatus.TODO,
    tags: [],
    dueDate: null,
    updatedAt: new Date('2025-01-14T11:00:00Z') // 25 hours ago
  };

  const score = calculateTaskScore(task);
  expect(score).toBe(10); // base only — 25 hours > 24 hours, no recency boost

  jest.useRealTimers();
});

test('task updated 23 hours ago SHOULD get recency boost', () => {
  jest.useFakeTimers();
  jest.setSystemTime(new Date('2025-01-15T12:00:00Z'));

  const task = {
    priority: TaskPriority.LOW,
    status: TaskStatus.TODO,
    tags: [],
    dueDate: null,
    updatedAt: new Date('2025-01-14T13:00:00Z') // 23 hours ago
  };

  const score = calculateTaskScore(task);
  expect(score).toBe(15); // base (10) + recency boost (5) = 15

  jest.useRealTimers();
});
```

**Identifying the bug:** The original calculation uses `Math.floor` which
means a task updated 23.9 hours ago returns daysSinceUpdate = 0 (gets boost ✅)
and a task updated 24.1 hours ago returns daysSinceUpdate = 1 (no boost ✅).
The boundary actually works correctly in JavaScript with floor.

**However the potential bug with a different approach:**
If the code used integer division incorrectly (e.g., dividing by
86400000 without floor), tasks updated exactly 24 hours ago might
get the boost when they should not. The test above pins down this boundary.

**Regression prevention tests added:**
The two tests above together prevent any future change to the recency
logic from silently breaking the 24-hour boundary behavior.

---

# PART 4: Integration Testing

## Exercise 4.1: Full Workflow Integration Test
```javascript
describe('Task Priority Workflow — Integration Tests', () => {

  // Fixed time to make tests deterministic
  beforeEach(() => {
    jest.useFakeTimers();
    jest.setSystemTime(new Date('2025-01-15T12:00:00Z'));
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  // Test data: 5 tasks with known expected scores
  const testTasks = [
    {
      id: 1,
      priority: TaskPriority.LOW,
      status: TaskStatus.TODO,
      tags: [],
      dueDate: null,
      updatedAt: '2020-01-01'
      // Expected score: 10
    },
    {
      id: 2,
      priority: TaskPriority.HIGH,
      status: TaskStatus.TODO,
      tags: [],
      dueDate: null,
      updatedAt: '2020-01-01'
      // Expected score: 30
    },
    {
      id: 3,
      priority: TaskPriority.MEDIUM,
      status: TaskStatus.TODO,
      tags: ['blocker'],
      dueDate: null,
      updatedAt: '2020-01-01'
      // Expected score: 20 + 8 = 28
    },
    {
      id: 4,
      priority: TaskPriority.URGENT,
      status: TaskStatus.DONE,
      tags: [],
      dueDate: null,
      updatedAt: '2020-01-01'
      // Expected score: 40 - 50 = -10
    },
    {
      id: 5,
      priority: TaskPriority.LOW,
      status: TaskStatus.TODO,
      tags: [],
      dueDate: '2025-01-14', // yesterday — overdue
      updatedAt: '2020-01-01'
      // Expected score: 10 + 30 = 40
    }
  ];

  // Expected order by score: task5(40), task2(30), task3(28), task1(10), task4(-10)

  test('sortTasksByImportance returns tasks in descending score order', () => {
    const sorted = sortTasksByImportance(testTasks);
    const sortedIds = sorted.map(t => t.id);
    expect(sortedIds).toEqual([5, 2, 3, 1, 4]);
  });

  test('getTopPriorityTasks returns correct number of tasks', () => {
    const top3 = getTopPriorityTasks(testTasks, 3);
    expect(top3).toHaveLength(3);
  });

  test('getTopPriorityTasks returns the highest scoring tasks', () => {
    const top3 = getTopPriorityTasks(testTasks, 3);
    const returnedIds = top3.map(t => t.id);
    expect(returnedIds).toEqual([5, 2, 3]);
  });

  test('getTopPriorityTasks with limit larger than array returns all tasks', () => {
    const result = getTopPriorityTasks(testTasks, 10);
    expect(result).toHaveLength(5);
  });

  test('sortTasksByImportance handles empty array', () => {
    expect(sortTasksByImportance([])).toEqual([]);
  });

  test('getTopPriorityTasks handles empty array', () => {
    expect(getTopPriorityTasks([], 3)).toEqual([]);
  });

  test('getTopPriorityTasks with limit 1 returns only the highest scoring task', () => {
    const top1 = getTopPriorityTasks(testTasks, 1);
    expect(top1).toHaveLength(1);
    expect(top1[0].id).toBe(5); // overdue LOW task scores 40, highest overall
  });
});
```

---

# Reflection

### What I learned about testing through this exercise

**1. Exact assertions matter more than I thought**
My first test used `toBeGreaterThan(0)` which is almost useless.
A broken function returning 999 would still pass. Always assert the
EXACT expected value when you can calculate it.

**2. Tests need to be deterministic**
Date-dependent tests that use `new Date()` internally will produce
different results on different days. Freezing time with fake timers
is not optional — it is essential for reliable tests.

**3. TDD forces you to think about the API before the implementation**
Writing the assigned-user test first forced me to decide: should
currentUserId be a second parameter or part of the task object?
I made that decision consciously before writing any implementation code.

**4. A helper function in tests is worth writing**
The `createBaseTask()` helper removed duplication across 5 tests and made
each test clearly show only the ONE thing it is changing. Without it,
each test would be 8 lines of repeated setup code.

**5. Integration tests reveal assumptions you did not know you had**
When I calculated expected scores for the integration test,
I discovered I had assumed HIGH (id 2) would beat the overdue LOW (id 5).
It did not — the overdue bonus (+30) pushed the LOW task to score 40,
above HIGH at 30. This was a surprise that the unit tests alone would
not have revealed.
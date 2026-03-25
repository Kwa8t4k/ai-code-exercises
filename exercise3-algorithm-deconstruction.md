# Exercise 3: Algorithm Deconstruction — Task Priority Scoring

## Algorithm Selected
**Algorithm 1: Task Priority Sorting and Filtering**

Language version studied: Python

---

## Step-by-Step Breakdown

### Stage 1 — Base Priority Score
```python
priority_weights = { LOW: 1, MEDIUM: 2, HIGH: 4, URGENT: 6 }
score = priority_weights[task.priority] * 10
```
Starting scores: LOW=10, MEDIUM=20, HIGH=40, URGENT=60

### Stage 2 — Due Date Bonus
| Situation | Bonus Points |
|-----------|-------------|
| Overdue (past due date) | +35 |
| Due today | +20 |
| Due in 1–2 days | +15 |
| Due this week | +10 |
| No due date / far future | +0 |

### Stage 3 — Status Penalty
| Status | Change |
|--------|--------|
| DONE | -50 (pushes completed tasks to the bottom) |
| REVIEW | -15 (nearly done, slightly lower priority) |

### Stage 4 — Tag Boost
If task has tags "blocker", "critical", or "urgent": **+8 points**

### Stage 5 — Recency Boost
If task was updated within last 24 hours: **+5 points**

---

## Concrete Example Walkthrough

**Task A:** HIGH priority, due tomorrow, tagged "blocker", updated today
```
40 (HIGH) + 15 (due soon) + 8 (blocker tag) + 5 (recent) = 68
```

**Task B:** URGENT priority, due next month, no special tags, not recent
```
60 (URGENT) + 0 + 0 + 0 = 60
```

**Result:** Task A (score 68) ranks ABOVE Task B (score 60).
Even though Task B is URGENT, Task A is more immediately pressing.
This models real-world judgment correctly.

---

## Reflection Questions

### 1. How did the AI's explanation change your understanding?
Before exploring with AI, I thought priority was a simple ranking system.
After the AI-guided breakdown, I understand it's a multi-factor weighted scoring
system that models how a human actually decides what to work on next —
considering urgency, deadlines, labels, and recent activity together.

### 2. What was still difficult after AI explanation?
Understanding WHY specific numbers were chosen (+35 for overdue, -50 for DONE).
The AI helped me understand WHAT the algorithm does,
but the reasoning behind those exact values required inferring the designer's intent.
I concluded they were chosen empirically — tested and adjusted until rankings "felt right."

### 3. How would I explain this to another junior developer?
"Every task gets a score. You start with points based on how important the task is
(URGENT gets the most). Then you add bonus points if it's due soon or already overdue.
Add a few more if it has urgent-sounding tags. Subtract a big chunk if it's already done.
The task with the highest total score goes to the top of your list."

### 4. How did you test your understanding against AI?
I posed this scenario to the AI:
*"Task A is LOW priority but 10 days overdue. Task B is HIGH priority with no due date.
Which ranks higher?"*

My calculation:
- Task A: LOW (10) + overdue (35) = **45**
- Task B: HIGH (40) + no due date (0) = **40**
- Task A wins.

The AI confirmed this is intentional: overdue tasks demand attention regardless of
their original priority level.

### 5. How might you improve this algorithm?
1. **Hardcoded numbers:** The bonus/penalty values (35, 20, 15, etc.) are fixed in code.
   They could be user-configurable settings.
2. **Missing priority handling:** If a task has no priority, it scores 0 × 10 = 0.
   A default fallback should be explicit.
3. **Gradual recency boost:** Currently only the last 24 hours gets +5.
   A sliding scale (last hour = +10, last day = +5, last week = +2) would be more nuanced.
4. **No upper score cap:** Theoretically a task could score very high.
   A maximum cap could prevent edge-case outliers from always dominating.
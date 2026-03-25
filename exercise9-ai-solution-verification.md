# Exercise 9: AI Solution Verification Challenge

## Problem Selected
Buggy sorting function: mergeSort with an incorrect variable increment in the merge step

---

## Original Buggy Code
```javascript
function mergeSort(arr) {
  if (arr.length <= 1) return arr;

  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));

  return merge(left, right);
}

function merge(left, right) {
  let result = [];
  let i = 0;
  let j = 0;

  while (i < left.length && j < right.length) {
    if (left[i] < right[j]) {
      result.push(left[i]);
      i++;
    } else {
      result.push(right[j]);
      j++;
    }
  }

  // Bug: Only one of these loops will execute
  while (i < left.length) {
    result.push(left[i]);
    j++; // BUG: should be i++, not j++
  }

  while (j < right.length) {
    result.push(right[j]);
    j++;
  }

  return result;
}
```

---

## Step 1 — AI Suggested Solution

After presenting the buggy code to AI, the suggested fix was:
```javascript
function merge(left, right) {
  let result = [];
  let i = 0;
  let j = 0;

  while (i < left.length && j < right.length) {
    if (left[i] < right[j]) {
      result.push(left[i]);
      i++;
    } else {
      result.push(right[j]);
      j++;
    }
  }

  // Fixed: increment i when consuming from left array
  while (i < left.length) {
    result.push(left[i]);
    i++; // FIXED: was j++, now correctly i++
  }

  while (j < right.length) {
    result.push(right[j]);
    j++;
  }

  return result;
}
```

The AI also explained: "The original while loop incrementing `j` instead of `i`
means `i` never advances, so `left[i]` is pushed repeatedly and the loop
never terminates (infinite loop), eventually causing a stack overflow or
browser hang."

---

## Verification Strategy 1: Collaborative Solution Verification

### My Understanding of the Solution

**What I think it does:**
The fix changes `j++` to `i++` inside the third while loop.
The third while loop's job is to copy any remaining elements from the
LEFT array that were not yet added during the main comparison loop.
Each time we copy an element from `left[i]`, we must advance `i`
so we move to the next element. The original code advanced `j` instead,
meaning `i` stayed at the same position forever.

**Why I think it fixes the problem:**
After the main while loop ends, either `i` has reached the end of `left`,
or `j` has reached the end of `right` — but not necessarily both.
The two remaining loops handle whichever side still has elements.
The left-remainder loop must use `i` as its counter since it is reading
from the `left` array. Using `j` made no logical sense because `j`
is the counter for the `right` array.

**My test cases:**

Test 1 — Basic case: mergeSort([3, 1, 2])
Expected: [1, 2, 3]
This forces a merge where the left array has one remaining element.

Test 2 — Already sorted: mergeSort([1, 2, 3, 4])
Expected: [1, 2, 3, 4]
Should return same order without modification.

Test 3 — Reverse sorted: mergeSort([4, 3, 2, 1])
Expected: [1, 2, 3, 4]
Worst case for many sort algorithms — all elements need reordering.

Test 4 — Duplicates: mergeSort([3, 1, 3, 2])
Expected: [1, 2, 3, 3]
Ensures duplicates are preserved and not dropped.

Test 5 — Single element: mergeSort([5])
Expected: [5]
Base case — should return immediately without entering merge.

Test 6 — Empty array: mergeSort([])
Expected: []
Edge case — empty input should not crash.

**Edge cases I am concerned about:**
- What happens when ALL elements from one side are smaller than
  the other? Then the entire right or left array ends up being
  appended by one of the remainder loops.
- What about very large arrays? Recursion depth could become an issue
  for arrays with thousands of elements.

**AI Confirmation of My Understanding:**
The AI confirmed my understanding was accurate. It added that the bug
specifically causes an **infinite loop** — because `i` never advances,
`left[i]` is the same element every iteration, and the condition
`i < left.length` is always true. The browser would freeze or throw
a "maximum call stack size exceeded" or timeout error.

---

## Verification Strategy 2: Learning Through Alternative Approaches

### My Understanding of the Original Approach
The merge sort divides the array in half recursively until each piece
has one element, then builds back up by merging sorted pairs.

**Pros:**
- O(n log n) time complexity — efficient for large arrays
- Stable sort — equal elements maintain their original order
- Predictable performance — always O(n log n) regardless of input order

**Cons:**
- Requires O(n) extra memory for the result arrays during merging
- More complex to understand and implement than simpler sorts
- Recursive approach can hit call stack limits on very large arrays

### Alternative Approaches the AI Provided

**Alternative 1: Iterative merge sort (bottom-up)**
Instead of recursing top-down, start with pairs of single elements
and merge upward. Avoids recursion depth issues entirely.
```javascript
function mergeSortIterative(arr) {
  const n = arr.length;
  const result = [...arr];

  for (let size = 1; size < n; size *= 2) {
    for (let start = 0; start < n; start += size * 2) {
      const mid = Math.min(start + size, n);
      const end = Math.min(start + size * 2, n);
      // merge result[start...mid] with result[mid...end]
    }
  }
  return result;
}
```
**Best for:** Very large arrays where call stack depth is a concern.

**Alternative 2: JavaScript built-in Array.sort()**
```javascript
const sorted = [...arr].sort((a, b) => a - b);
```
**Pros:** One line, battle-tested, optimised by the JavaScript engine.
**Cons:** Not a learning exercise — hides the algorithm entirely.
**Best for:** Production code where you just need the result.

**Alternative 3: Quick Sort**
```javascript
function quickSort(arr) {
  if (arr.length <= 1) return arr;
  const pivot = arr[Math.floor(arr.length / 2)];
  const left = arr.filter(x => x < pivot);
  const middle = arr.filter(x => x === pivot);
  const right = arr.filter(x => x > pivot);
  return [...quickSort(left), ...middle, ...quickSort(right)];
}
```
**Pros:** Average O(n log n), sorts in-place in optimised implementations.
**Cons:** Worst case O(n²) on already-sorted arrays with naive pivot selection.
**Best for:** General purpose sorting where memory usage matters.

### Comparison Table

| Approach | Time Complexity | Space Complexity | Stable | Best Use Case |
|----------|----------------|------------------|--------|---------------|
| Merge Sort (recursive) | O(n log n) | O(n) | Yes | Learning, guaranteed performance |
| Merge Sort (iterative) | O(n log n) | O(n) | Yes | Very large arrays |
| Array.sort() | O(n log n) | O(log n) | Yes (V8 engine) | Production code |
| Quick Sort | O(n log n) avg | O(log n) | No | Memory-constrained environments |

---

## Verification Strategy 3: Developing a Critical Eye

### My Initial Assessment of the Fixed Solution

**Strengths I see:**
- The fix is minimal — one character change (`j` to `i`) that directly addresses
  the root cause without restructuring the whole function
- The rest of the merge logic is correct — the main comparison while loop
  and the right-remainder while loop were never broken
- The mergeSort function itself was always correct — only the merge helper had the bug

**Concerns I have:**
- The fix is so small that it could easily be overlooked in a code review
- There is no input validation — what if non-numbers are passed in the array?
- The function does not handle arrays of objects — it only compares with `<`
  which would not work for sorting objects by a property

**Questions that came to mind:**
1. Would the original bug cause a silent wrong answer or an obvious crash?
   (Answer: it causes an infinite loop / browser hang — very obvious but hard to trace)
2. Is merge sort stable? Does it preserve the order of equal elements?
   (Answer: yes, because of the strict `<` comparison — equal elements go
   to the right branch first, preserving left-array order)

### Critical Review Findings

**Hidden assumptions in the code:**
| Assumption | Risk |
|------------|------|
| Input contains only numbers | Strings or objects would produce incorrect sort order |
| Input is a flat array | Nested arrays would not be sorted recursively |
| Array size is reasonable | Very large arrays could hit JavaScript's call stack limit |
| All elements are comparable with < | Mixed types (numbers and strings) could produce unexpected results |

**How maintainable is this solution if requirements change?**

If we need to sort objects by a property in the future:
```javascript
// Current code only handles:
mergeSort([3, 1, 2])

// Would need refactoring to handle:
mergeSort(tasks, (a, b) => a.priority - b.priority)
```
The solution would need a comparator function parameter added — a common
pattern in sort implementations. This is a reasonable improvement to make
proactively.

**Improvement suggested to address biggest concerns:**
```javascript
function merge(left, right, comparator = (a, b) => a - b) {
  let result = [];
  let i = 0;
  let j = 0;

  while (i < left.length && j < right.length) {
    if (comparator(left[i], right[j]) <= 0) {
      result.push(left[i]);
      i++;
    } else {
      result.push(right[j]);
      j++;
    }
  }

  // Append any remaining elements from left
  while (i < left.length) {
    result.push(left[i]);
    i++; // Correctly advances i
  }

  // Append any remaining elements from right
  while (j < right.length) {
    result.push(right[j]);
    j++;
  }

  return result;
}
```
Adding a `comparator` parameter makes this reusable for any data type.

---

## Final Verified Solution
```javascript
function mergeSort(arr, comparator = (a, b) => a - b) {
  if (arr.length <= 1) return arr;

  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid), comparator);
  const right = mergeSort(arr.slice(mid), comparator);

  return merge(left, right, comparator);
}

function merge(left, right, comparator = (a, b) => a - b) {
  let result = [];
  let i = 0;
  let j = 0;

  // Compare elements from both halves and add the smaller one first
  while (i < left.length && j < right.length) {
    if (comparator(left[i], right[j]) <= 0) {
      result.push(left[i]);
      i++;
    } else {
      result.push(right[j]);
      j++;
    }
  }

  // Append remaining elements from left (if any)
  while (i < left.length) {
    result.push(left[i]);
    i++; // FIXED: was j++ in original — must increment i to advance through left array
  }

  // Append remaining elements from right (if any)
  while (j < right.length) {
    result.push(right[j]);
    j++;
  }

  return result;
}

// Test cases
console.log(mergeSort([3, 1, 2]));           // [1, 2, 3]
console.log(mergeSort([4, 3, 2, 1]));        // [1, 2, 3, 4]
console.log(mergeSort([1, 2, 3, 4]));        // [1, 2, 3, 4]
console.log(mergeSort([3, 1, 3, 2]));        // [1, 2, 3, 3]
console.log(mergeSort([5]));                 // [5]
console.log(mergeSort([]));                  // []

// Bonus: sorting objects using comparator
const tasks = [
  { name: "Task C", priority: 3 },
  { name: "Task A", priority: 1 },
  { name: "Task B", priority: 2 }
];
console.log(mergeSort(tasks, (a, b) => a.priority - b.priority));
// [{name: "Task A"...}, {name: "Task B"...}, {name: "Task C"...}]
```

---

## Reflection Questions

**How did confidence in the solution change after verification?**
Before verification I accepted the one-character fix at face value.
After working through all three verification strategies, I understand
exactly WHY `j++` was wrong (it belongs to the right array counter),
WHY `i++` is correct (it advances through the left array), and
WHY the bug causes an infinite loop rather than a wrong answer.
My confidence went from "I think this is right" to "I can explain
exactly why this is right to someone else."

**What aspects required the most scrutiny?**
The infinite loop behaviour was not immediately obvious to me.
At first I thought the bug might just produce a wrong sort order.
Understanding that `i` never advancing means the while condition
`i < left.length` is permanently true — that required careful tracing
through the logic step by step.

**Which verification technique was most valuable?**
Strategy 3 (Developing a Critical Eye) was most valuable because it
revealed the hidden assumption about input type. The fix for the
immediate bug was simple, but examining what the code ASSUMES about
its input led to the comparator improvement — which makes the function
genuinely reusable rather than just fixed.
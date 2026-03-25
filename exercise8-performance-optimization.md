# Exercise 8: Performance Optimization Challenge

## Scenario Selected
Scenario 1: Slow Code Analysis — Python find_product_combinations function

---

## Original Code with Performance Issue
```python
def find_product_combinations(products, target_price, price_margin=10):
    results = []

    for i in range(len(products)):
        for j in range(len(products)):
            if i != j:
                product1 = products[i]
                product2 = products[j]
                combined_price = product1['price'] + product2['price']

                if (target_price - price_margin) <= combined_price <= (target_price + price_margin):
                    if not any(r['product1']['id'] == product2['id'] and
                               r['product2']['id'] == product1['id'] for r in results):
                        pair = {
                            'product1': product1,
                            'product2': product2,
                            'combined_price': combined_price,
                            'price_difference': abs(target_price - combined_price)
                        }
                        results.append(pair)

    results.sort(key=lambda x: x['price_difference'])
    return results
```

**Context:**
- Finds pairs of products whose combined price falls within a target range
- Processes 5,000+ products
- Currently takes 20-30 seconds to run
- Runs on every page load of the Product Recommendations page
- Environment: Python 3.9, web server with 4GB RAM

---

## AI Prompt Used (Prompt 1: Slow Code Analysis)
```
I have a piece of code that's running slowly. I'd like to understand why
and how to improve it.

Here's the slow-performing code:
[pasted the find_product_combinations function above]

Context about the issue:
- What this code does: Find pairs of products whose combined price is within
  target_price ± price_margin of a target amount
- Typical input size: Processing 5,000+ products
- Current performance: Takes about 25 seconds to run, making the
  product recommendations page very slow to load
- Environment: Python 3.9 on a web server with 4GB RAM

Could you:
1. Explain in simple terms why this code might be slow
2. Identify the specific operations causing the slowdown
3. Suggest 2-3 specific improvements I could make
4. Explain the performance concepts I should learn to avoid similar issues
5. Suggest tools to measure the actual bottlenecks

I am particularly interested in learning the underlying performance concepts,
not just getting a quick fix.
```

---

## Performance Analysis

### Why This Code Is Slow — Plain English Explanation

Imagine you have 5,000 products and you want to find every pair.
This code looks at EVERY product against EVERY other product.
That means 5,000 × 5,000 = **25,000,000 comparisons**.

Then, for each matching pair found, it does ANOTHER search through
all already-found results to check for duplicates. As `results` grows,
this inner duplicate check gets slower and slower.

This is called an **O(n²) algorithm** — the work grows with the
SQUARE of the input size. Double the products = four times the work.

---

### Specific Performance Problems Identified

**Problem 1: Nested loops comparing every product against every other product**
```python
for i in range(len(products)):
    for j in range(len(products)):  # This inner loop runs 5000 times for EACH outer loop
```
With 5,000 products this is 25 million iterations before any filtering.

**Problem 2: Duplicate pairs checked using linear search through results**
```python
if not any(r['product1']['id'] == product2['id'] and ...)
```
Every time a match is found, Python scans the ENTIRE results list to check
for its reverse pair. As results grows, this scan gets longer and longer.
This is O(n) inside an already O(n²) loop — making the total O(n³) in the worst case.

**Problem 3: Both directions of every pair are being compared**
The outer loop compares product[0] vs product[1], then later
product[1] vs product[0] — the same pair twice. Half the work is wasted.

---

### Suggested Optimisations

**Optimisation 1: Only compare each pair once using j = i + 1**

Instead of starting j at 0 every time, start it at i+1:
```python
for i in range(len(products)):
    for j in range(i + 1, len(products)):  # j always ahead of i
        # No need to check for duplicate pairs — they can't happen now
```
This cuts comparisons from 25,000,000 to 12,500,000 — exactly half.
No duplicate checking needed at all.

**Optimisation 2: Use a set to track seen pairs instead of scanning results**
```python
seen_pairs = set()
for i in range(len(products)):
    for j in range(i + 1, len(products)):
        pair_key = (min(products[i]['id'], products[j]['id']),
                    max(products[i]['id'], products[j]['id']))
        if pair_key not in seen_pairs:  # O(1) lookup instead of O(n)
            seen_pairs.add(pair_key)
```
Set membership checks are O(1) — instant — compared to scanning a list which is O(n).

**Optimisation 3: Sort by price first, then use two-pointer technique**

Sort all products by price once, then use a left pointer and right pointer
that move toward each other. This reduces the algorithm from O(n²) to O(n log n):
```python
def find_product_combinations_optimised(products, target_price, price_margin=10):
    # Sort once by price — O(n log n)
    sorted_products = sorted(products, key=lambda p: p['price'])
    results = []
    left = 0
    right = len(sorted_products) - 1

    while left < right:
        combined = sorted_products[left]['price'] + sorted_products[right]['price']
        lower = target_price - price_margin
        upper = target_price + price_margin

        if lower <= combined <= upper:
            results.append({
                'product1': sorted_products[left],
                'product2': sorted_products[right],
                'combined_price': combined,
                'price_difference': abs(target_price - combined)
            })
            left += 1
            right -= 1
        elif combined < lower:
            left += 1   # Sum too small, move left pointer right to increase sum
        else:
            right -= 1  # Sum too large, move right pointer left to decrease sum

    results.sort(key=lambda x: x['price_difference'])
    return results
```

---

### Performance Comparison

| Approach | Operations for 5,000 products | Estimated time |
|----------|-------------------------------|----------------|
| Original (nested loops + linear duplicate check) | ~25,000,000+ | 20-30 seconds |
| Fixed nested loops (j = i+1, set for duplicates) | ~12,500,000 | ~10-15 seconds |
| Two-pointer after sort | ~10,000 comparisons | < 1 second |

The two-pointer approach is dramatically faster because it uses the sorted
order to make intelligent decisions — if the sum is too small, move the
left pointer right; if too large, move the right pointer left.

---

### Performance Concepts Learned

**1. Big O Notation — understanding how code scales**
O(n²) means: if input doubles, time quadruples.
O(n log n) means: if input doubles, time barely more than doubles.
Always ask: "what happens to my code when the input is 10x larger?"

**2. The cost of nested loops**
Each additional nested loop multiplies the complexity. A single loop = O(n).
Two nested loops = O(n²). Always look for ways to reduce nesting.

**3. Use the right data structure for lookups**
Lists require scanning every element to find something: O(n).
Sets and dictionaries find things instantly: O(1).
When checking "have I seen this before?", always use a set, not a list.

**4. Sorting as an enabler**
Sorting data once (O(n log n)) often unlocks much faster algorithms.
The two-pointer technique only works because the data is sorted.
A small upfront cost enables huge savings later.

---

### Tools to Measure Performance
```python
# Simple timing measurement
import time
start = time.time()
result = find_product_combinations(products, 500, 50)
end = time.time()
print(f"Execution time: {end - start:.2f} seconds")

# More detailed profiling using cProfile
import cProfile
cProfile.run('find_product_combinations(products, 500, 50)')
```

---

## Reflection Questions

**How did the optimisation change your understanding of the algorithm?**
I initially thought the nested loop was unavoidable for a "compare all pairs"
problem. Learning the two-pointer technique showed me that sorting first
changes the problem entirely. It is a fundamentally different way of thinking
about the same task.

**What performance improvements were achieved?**
The two-pointer approach reduces from ~25 million operations to ~10,000.
That is a 2,500x improvement. For a page that previously took 25 seconds to
load, this would make it load in milliseconds — a dramatic user experience improvement.

**What did you learn about performance bottlenecks?**
The biggest insight is that the duplicate-checking inner scan was actually
the worst part — it was O(n) inside an O(n²) loop, making the total
effectively O(n³). The duplicate problem disappeared completely once
the j = i+1 approach was used, because duplicates become structurally impossible.

**How would you approach similar issues in the future?**
1. Measure first — confirm WHERE the slowness actually is before optimising
2. Identify the loop structure and calculate Big O complexity
3. Ask: is there a sorted-data approach that unlocks a better algorithm?
4. Ask: am I using lists where I should be using sets or dictionaries?

**What tools would you use proactively?**
- Python's `cProfile` module for identifying which functions are slowest
- `time.time()` for quick before/after measurements
- Big O analysis as a mental checklist when writing any loop
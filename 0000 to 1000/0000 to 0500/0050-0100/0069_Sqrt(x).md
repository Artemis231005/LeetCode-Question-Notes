# LeetCode 69 — Sqrt(x)

## Metadata

* **LeetCode:** 69
* **Problem:** Sqrt(x)
* **Difficulty:** Easy
* **Topics:** Math, Binary Search
* **Pattern:** Binary Search on Answer
* **Key Pattern:** Find the largest integer `mid` such that `mid² <= x`
* **Key Technique:** Binary search over the possible square-root values
* **Key Template:** Binary Search on Answer
* **Optimal Complexity:** `O(log x)` time, `O(1)` space

---

## Problem

Given a non-negative integer `x`, return the **square root of `x` rounded down** to the nearest integer.

You must not use built-in exponent functions such as `sqrt()`.

Examples:

```text
x = 4
√4 = 2
Answer = 2
```

```text
x = 8
√8 ≈ 2.828
Answer = 2
```

So the problem asks for:

```text
floor(√x)
```

---

## Approach — Binary Search

### Idea

We need to find the **largest integer `k`** satisfying:

```text
k² <= x
```

The possible answer lies between:

```text
0 and x
```

For every `mid`:

```text
mid² <= x
```

means `mid` could be the answer, but there may be a larger valid value.

If:

```text
mid² > x
```

then `mid` is too large, so search the left half.

Therefore:

```text
mid² <= x
    ↓
answer could be mid
    ↓
search right

mid² > x
    ↓
mid is too large
    ↓
search left
```

We keep the best valid value in `ans`.

---

### Dry Run

For:

```text
x = 8
```

Search range:

```text
left = 0
right = 8
```

#### Step 1

```text
mid = 4

4² = 16 > 8
```

Too large:

```text
right = 3
```

#### Step 2

```text
mid = 1

1² = 1 <= 8
```

Valid:

```text
ans = 1
left = 2
```

#### Step 3

```text
mid = 2

2² = 4 <= 8
```

Valid:

```text
ans = 2
left = 3
```

#### Step 4

```text
mid = 3

3² = 9 > 8
```

Too large:

```text
right = 2
```

Now:

```text
left > right
```

Stop.

Answer:

```text
2
```

---

### Algorithm

1. Set:

   ```text
   left = 0
   right = x
   ans = 0
   ```
2. While `left <= right`:

   * Calculate:

     ```text
     mid = left + (right - left) / 2
     ```
   * If `mid² <= x`:

     * `mid` is a valid answer.
     * Store it:

       ```text
       ans = mid
       ```
     * Search for a larger valid value:

       ```text
       left = mid + 1
       ```
   * Otherwise:

     ```text
     right = mid - 1
     ```
3. Return `ans`.

---

### Complexity

* **Time:** `O(log x)` — binary search halves the search space each iteration.
* **Space:** `O(1)`.

---

### Notes / Tips

* The key is recognizing this as **Binary Search on Answer**.
* We are not searching for `x` itself. We are searching for the answer `k` satisfying:

```text
k² <= x
```

* The important condition is:

```text
mid * mid <= x
```

* **Integer overflow warning:** `mid * mid` can overflow for large values of `x`.
* A safer comparison is:

```text
mid <= x / mid
```

instead of:

```text
mid * mid <= x
```

because `x / mid` does not overflow.

* We want the **largest valid** `mid`, so when `mid` is valid, move right rather than immediately returning.

* This is the same general pattern used in problems such as:

```text
Find minimum capacity
Find maximum feasible value
Koko Eating Bananas
Allocate Books
```

The general idea is:

```text
Search for an answer
    ↓
Check whether mid is feasible
    ↓
Move left/right based on feasibility
```

---

### Code

```cpp
class Solution {
public:
    int mySqrt(int x) {
        if (x < 2) {
            return x;
        }

        long long left = 1;
        long long right = x;
        long long ans = 0;

        while (left <= right) {
            long long mid = left + (right - left) / 2;

            if (mid <= x / mid) {
                ans = mid;
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }

        return ans;
    }
};
```

---

## Quick Revision

```text
Sqrt(x)
    ↓
Find largest k such that k² <= x
    ↓
Binary Search
    ↓
mid² <= x?
    ├── YES → ans = mid → search right
    └── NO  → search left
```

### Core Template

```text
left = 1
right = x
ans = 0

while left <= right:
    mid = left + (right - left) / 2

    if mid <= x / mid:
        ans = mid
        left = mid + 1
    else:
        right = mid - 1

return ans
```

### Key Insight

```text
We need floor(√x)

⇔

Find the largest integer k
such that:

k² <= x
```

**Pattern to remember:**
**Square Root → Binary Search on Answer → Find largest `mid` satisfying `mid² <= x`**

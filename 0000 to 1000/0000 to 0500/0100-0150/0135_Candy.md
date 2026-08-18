# LeetCode 135 — Candy

## Metadata

* **LeetCode:** 135
* **Problem:** Candy
* **Difficulty:** Hard
* **Topics:** Array, Greedy
* **Pattern:** Greedy, Two-Pass
* **Key Technique:** Left-to-right + Right-to-left
* **Optimal Complexity:** `O(n)` Time, `O(1)` Extra Space

---

## Problem

There are `n` children standing in a line.
Each child has a rating given by `ratings[i]`.

Assign candies to the children such that:
1. Every child gets **at least 1 candy**.
2. If `ratings[i] > ratings[i - 1]`, then child `i` must get **more candies** than child `i - 1`.
3. If `ratings[i] > ratings[i + 1]`, then child `i` must get **more candies** than child `i + 1`.

Return the **minimum total number of candies** required.

---

# Approach 1 — Brute Force

## Idea
Start by giving every child `1` candy.

Then repeatedly check every adjacent pair:
* If the left child has a higher rating, increase the left child's candies if necessary.
* If the right child has a higher rating, increase the right child's candies if necessary.

Continue until no candy value needs to be changed.
The repeated passes are required because increasing one child's candy count can force another child's count to increase.

### Example

```text
ratings = [1, 2, 3]

Initial:
candies = [1, 1, 1]

Pass:
1 < 2  → [1, 2, 1]
2 < 3  → [1, 2, 3]

Answer = 6
```

For more complicated patterns, several passes may be required.

---

## Dry Run

```text
ratings = [1, 3, 2, 2, 1]

Initial:
candies = [1, 1, 1, 1, 1]

After enforcing increasing relationships:
candies = [1, 2, 1, 1, 1]

Now the decreasing relationships require:
candies = [1, 2, 1, 1, 2]   // depending on scan order

Further passes may be required to propagate changes.
```

The main issue is that local corrections can propagate through the array.

---

## Notes / Tips

* This approach directly enforces the constraints.
* However, repeatedly scanning the array can take `O(n²)` time.
* It is mainly useful for understanding why a more structured greedy approach is needed.

---

## Complexity

* **Time:** `O(n²)`
* **Space:** `O(n)`

---

## Code

```cpp
class Solution {
public:
    int candy(vector<int>& ratings) {
        int n = ratings.size();
        vector<int> candies(n, 1);

        bool changed = true;

        while (changed) {
            changed = false;

            for (int i = 0; i < n; i++) {
                if (i > 0 && ratings[i] > ratings[i - 1]) {
                    if (candies[i] <= candies[i - 1]) {
                        candies[i] = candies[i - 1] + 1;
                        changed = true;
                    }
                }

                if (i + 1 < n && ratings[i] > ratings[i + 1]) {
                    if (candies[i] <= candies[i + 1]) {
                        candies[i] = candies[i + 1] + 1;
                        changed = true;
                    }
                }
            }
        }

        return accumulate(candies.begin(), candies.end(), 0);
    }
};
```

---

# Approach 2 — Two Arrays / Two-Pass Greedy

## Idea

The important observation is that each child has **two independent neighbour constraints**:

* Left neighbour constraint:

  * If `ratings[i] > ratings[i - 1]`, then `candies[i] > candies[i - 1]`.
* Right neighbour constraint:

  * If `ratings[i] > ratings[i + 1]`, then `candies[i] > candies[i + 1]`.

Handle these constraints separately.

### Step 1 — Left to Right

Give every child `1` candy initially.

Then:

```text
if ratings[i] > ratings[i - 1]:
    left[i] = left[i - 1] + 1
else:
    left[i] = 1
```

This guarantees all **left-neighbour constraints**.

### Step 2 — Right to Left

Similarly:

```text
if ratings[i] > ratings[i + 1]:
    right[i] = right[i + 1] + 1
else:
    right[i] = 1
```

This guarantees all **right-neighbour constraints**.

### Step 3 — Combine

A child must satisfy **both** constraints.

Therefore:

```text
candies[i] = max(left[i], right[i])
```

We take the maximum because the child must have enough candies to satisfy whichever side requires more.

---

## Why Not Check Both Neighbours in One Forward Pass?
Consider:
```text
ratings = [1, 2, 3, 2, 1]
```

The middle child has rating `3`.

It needs:
```text
3 > 2  → more candies than left
3 > 2  → more candies than right
```

A left-to-right pass can correctly handle the increasing slope:

```text
[1, 2, 3]
```

But when it reaches the right side, the required candy count of the right neighbour is not yet known.

The right-to-left pass is specifically needed to propagate information from the **right side toward the left**.

Therefore, instead of trying to handle both directions in one pass, we solve:

```text
Left constraint  → Left to Right
Right constraint → Right to Left
```

and combine them.

---

## Dry Run

```text
ratings = [1, 2, 3, 2, 1]
```

### Initial

```text
ratings = [1, 2, 3, 2, 1]
```

### Left to Right

```text
1 → 1
2 > 1 → 2
3 > 2 → 3
2 > 3? No → 1
1 > 2? No → 1

left = [1, 2, 3, 1, 1]
```

### Right to Left

```text
1 → 1
2 > 1 → 2
3 > 2 → 3
2 > 3? No → 1
1 > 2? No → 1

right = [1, 1, 3, 2, 1]
```

### Combine

```text
max(left[i], right[i])

i = 0 → max(1, 1) = 1
i = 1 → max(2, 1) = 2
i = 2 → max(3, 3) = 3
i = 3 → max(1, 2) = 2
i = 4 → max(1, 1) = 1

candies = [1, 2, 3, 2, 1]
```

```text
Total = 1 + 2 + 3 + 2 + 1 = 9
```

---

## Notes / Tips

* The two passes are solving **different directional constraints**.
* Do not try to directly overwrite one array in the first pass and assume the second condition is automatically satisfied.
* `max(left[i], right[i])` is essential because a child can simultaneously belong to:

  * an increasing sequence from the left, and
  * a decreasing sequence toward the right.
* Equal ratings do **not** impose any constraint.

---

## Complexity

* **Time:** `O(n)`
* **Space:** `O(n)`

---

## Code

```cpp
class Solution {
public:
    int candy(vector<int>& ratings) {
        int n = ratings.size();

        vector<int> left(n, 1);
        vector<int> right(n, 1);

        // Left to right
        for (int i = 1; i < n; i++) {
            if (ratings[i] > ratings[i - 1]) {
                left[i] = left[i - 1] + 1;
            }
        }

        // Right to left
        for (int i = n - 2; i >= 0; i--) {
            if (ratings[i] > ratings[i + 1]) {
                right[i] = right[i + 1] + 1;
            }
        }

        int total = 0;

        for (int i = 0; i < n; i++) {
            total += max(left[i], right[i]);
        }

        return total;
    }
};
```

---

# Approach 3 — Optimal Greedy / Constant Space

## Idea

The two-array solution is already `O(n)` time, but it uses `O(n)` extra space.

We can eliminate the arrays by processing the rating sequence according to its **slopes**.

There are three important situations:

1. Increasing slope
2. Decreasing slope
3. A peak between increasing and decreasing slopes

For example:

```text
ratings = [1, 2, 3, 2, 1]

             peak
              ↓
            3
          /   \
        2       2
      /           \
    1               1
```

The increasing side needs:

```text
1, 2, 3
```

The decreasing side needs:

```text
3, 2, 1
```

The peak must satisfy **both sides**, so it needs:

```text
max(increasing length, decreasing length) + 1
```

Instead of explicitly storing candy counts, count the lengths of increasing and decreasing runs.

---

## Dry Run

Consider:

```text
ratings = [1, 2, 3, 2, 1]
```

There is:

```text
Increasing run: 1 → 2 → 3
Length = 2 edges

Decreasing run: 3 → 2 → 1
Length = 2 edges
```

For the increasing run:

```text
candies contribution = 1 + 2 + 3
```

For the decreasing run:

```text
candies contribution = 2 + 1
```

The peak `3` is shared, so it must be adjusted to satisfy both sides.

Final:

```text
candies = [1, 2, 3, 2, 1]

Total = 9
```

---

## Handling Increasing Sequences

For:

```text
ratings = [1, 2, 3, 4]
```

Candy counts must be:

```text
[1, 2, 3, 4]
```

If the increasing run has length `up`, its contribution can be calculated as:

```text
1 + 2 + ... + up
```

---

## Handling Decreasing Sequences

For:

```text
ratings = [4, 3, 2, 1]
```

Candy counts must be:

```text
[4, 3, 2, 1]
```

The decreasing run contributes similarly.

---

## Handling a Peak

This is the most important part.

Consider:
```text
ratings = [1, 2, 3, 2, 1]
```

The peak belongs to both:
```text
increasing sequence
        ↓
1 → 2 → 3

decreasing sequence
        ↓
3 → 2 → 1
```

If the increasing run has length `up` and the decreasing run has length `down`, the peak needs:

```text
max(up, down) + 1
```

This prevents undercounting the peak.

---

## Handling Equal Ratings

For:

```text
ratings = [1, 2, 2, 1]
```

The two `2`s do not have a greater-than relationship.

Therefore, the increasing/decreasing run ends at equality.

The equal-rating boundary effectively resets the sequence.

---

## Notes / Tips

* Think in terms of **slopes**, not individual candy assignments.
* `up` represents the length of the current increasing slope.
* `down` represents the length of the current decreasing slope.
* The peak needs to satisfy both slopes.
* Equal ratings break both increasing and decreasing runs.
* The constant-space solution is essentially compressing the information stored explicitly in the two arrays.
* The core reason for the `max(up, down)` adjustment is the peak shared by an increasing and decreasing run.

---

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

---

## Code

```cpp
class Solution {
public:
    int candy(vector<int>& ratings) {
        int n = ratings.size();

        if (n == 0)
            return 0;

        int total = 1;
        int up = 0;
        int down = 0;
        int peak = 0;

        for (int i = 1; i < n; i++) {

            if (ratings[i] > ratings[i - 1]) {
                // Increasing slope
                up++;
                peak = up;

                // Start/continue increasing sequence
                total += up;

                // A new increasing slope cancels the previous
                // decreasing run.
                down = 0;
            }
            else if (ratings[i] < ratings[i - 1]) {
                // Decreasing slope
                down++;

                // Add candies for the decreasing side.
                total += down;

                // If the decreasing side is longer than the
                // increasing side, the peak needs one extra candy.
                if (down > peak) {
                    total++;
                }
            }
            else {
                // Equal ratings break both slopes.
                up = 0;
                down = 0;
                peak = 0;

                total += 1;
            }
        }

        return total;
    }
};
```

---

# Final Comparison

| Approach       |    Time |  Space | Main Idea                                   |
| -------------- | ------: | -----: | ------------------------------------------- |
| Brute Force    | `O(n²)` | `O(n)` | Repeatedly enforce constraints              |
| Two Arrays     |  `O(n)` | `O(n)` | Solve left and right constraints separately |
| Constant Space |  `O(n)` | `O(1)` | Track increasing/decreasing slopes          |


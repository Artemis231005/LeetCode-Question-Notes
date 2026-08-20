# LeetCode 45 — Jump Game II

## Metadata

* **LeetCode:** 45
* **Problem:** Jump Game II
* **Difficulty:** Medium
* **Topics:** Array, Greedy, Dynamic Programming
* **Pattern:** Greedy, Range Expansion
* **Key Pattern:** Track the farthest position reachable within the current jump
* **Key Technique:** Greedy BFS / Level-by-Level Range Expansion
* **Optimal Complexity:** `O(n)` time, `O(1)` space
* **Key Template:** Greedy Range / Level Traversal

---

## Problem

You are given an integer array `nums` where:

```text
nums[i] = maximum jump length from index i
```

You start at index `0`.

Return the **minimum number of jumps** required to reach the last index.

You can assume that the last index is always reachable.

Example:

```text
nums = [2,3,1,1,4]

Output = 2
```

Explanation:

```text
0 → 1 → 4
```

Only `2` jumps are required.

---

## Approach — Greedy

### Idea

At every jump, instead of choosing a specific next index immediately, consider the **entire range of positions reachable with the current number of jumps**.

For the current range, find the position that can take us **farthest**.

Example:

```text
nums = [2,3,1,1,4]
```

Starting from index `0`:

```text
index 0 can reach:
[1, 2]
```

Within this range:

```text
index 1 → can reach 4
index 2 → can reach 3
```

So index `1` is the best choice.

The important observation is:

> We do not actually need to choose index `1`. We only need to know the farthest position reachable by the next jump.

This turns the problem into a **range expansion** problem.

---

## Greedy Range Concept

Maintain three variables:

```cpp
int jumps = 0;
int currentEnd = 0;
int farthest = 0;
```

### `farthest`

The farthest position we can reach from any index in the current range.

### `currentEnd`

The last index that can be reached using the current number of jumps.

### `jumps`

Number of jumps used so far.

Think of:

```text
Current jump range
        ↓
[................]
        ↓
Find farthest reachable position
        ↓
That becomes the next range
```

---

## Dry Run

Consider:

```text
nums = [2,3,1,1,4]
```

Indices:

```text
index:  0  1  2  3  4
nums:   2  3  1  1  4
```

Initially:

```text
jumps = 0
currentEnd = 0
farthest = 0
```

### Index 0

From index `0`:

```text
0 + nums[0]
= 0 + 2
= 2
```

So:

```text
farthest = 2
```

We reached the end of our current range:

```text
i == currentEnd
0 == 0
```

Therefore, we must make another jump.

```text
jumps = 1
currentEnd = farthest = 2
```

Now one jump allows us to reach:

```text
[0 ... 2]
```

### Index 1

```text
1 + nums[1]
= 1 + 3
= 4
```

Update:

```text
farthest = 4
```

### Index 2

```text
2 + nums[2]
= 2 + 1
= 3
```

`farthest` remains:

```text
4
```

We have reached the end of the current range:

```text
i == currentEnd
2 == 2
```

Therefore:

```text
jumps = 2
currentEnd = 4
```

Now:

```text
currentEnd == last index
```

So the answer is:

```text
2
```

---

## Why Does This Greedy Strategy Work?

Suppose the current jump can take us to:

```text
[l ........ r]
```

We need to make another jump.

Any position inside this range is a possible next position.

Instead of deciding immediately which one to choose, calculate:

```text
maximum reachable position
```

among all positions in the range.

Choosing the position that gives the farthest reach can never make the answer worse because every other choice reaches a position no farther than it.

Therefore:

```text
Current range
      ↓
Find maximum reach
      ↓
Expand to that range
      ↓
Count one jump
```

This is essentially **BFS performed greedily without explicitly storing a queue**.

Each range corresponds to one BFS level:

```text
Level 0 → starting position
Level 1 → positions reachable in 1 jump
Level 2 → positions reachable in 2 jumps
...
```

The first level that reaches the last index gives the minimum number of jumps.

---

### Algorithm

1. Initialize:

   ```cpp
   jumps = 0
   currentEnd = 0
   farthest = 0
   ```
2. Traverse the array up to the second-last index.
3. For every index `i`, update:

   ```cpp
   farthest = max(farthest, i + nums[i]);
   ```
4. When `i` reaches `currentEnd`, the current jump range is exhausted.
5. Make one jump:

   ```cpp
   jumps++;
   ```
6. Expand the current range:

   ```cpp
   currentEnd = farthest;
   ```
7. Continue until the last index becomes reachable.
8. Return `jumps`.

---

### Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

Only one pass through the array is required.

---

### Notes / Tips

* Do **not** greedily choose the index with the largest `nums[i]` blindly.
* What matters is:

  ```cpp
  i + nums[i]
  ```

  because that is the actual farthest position reachable from `i`.
* Think in terms of **ranges**, not individual jumps.
* `currentEnd` represents the boundary of the current jump.
* `farthest` represents how far the next jump can take us.
* When:

  ```cpp
  i == currentEnd
  ```

  we have finished processing the current jump's range, so we increment `jumps`.
* We iterate only until `n - 2` because once the last index is reached, no additional jump is required.
* This problem is essentially a **greedy version of BFS**:

  * Current range = current BFS level
  * `farthest` = next BFS level's boundary
  * `jumps` = BFS level count

---

### Code

```cpp
class Solution {
public:
    int jump(vector<int>& nums) {
        int jumps = 0;
        int currentEnd = 0;
        int farthest = 0;

        for (int i = 0; i < nums.size() - 1; i++) {
            farthest = max(farthest, i + nums[i]);

            if (i == currentEnd) {
                jumps++;
                currentEnd = farthest;
            }
        }

        return jumps;
    }
};
```

---

## Alternative Approach — Dynamic Programming

### Idea

Define:

```text
dp[i] = minimum number of jumps required to reach index i
```

Initially:

```text
dp[0] = 0
```

For every reachable index `i`, we can jump to:

```text
i + 1
i + 2
...
i + nums[i]
```

Update:

```cpp
dp[next] = min(dp[next], dp[i] + 1);
```

This works, but it is slower than the greedy solution.

---

### Dry Run

For:

```text
nums = [2,3,1,1,4]
```

Start:

```text
dp = [0, INF, INF, INF, INF]
```

From index `0`:

```text
0 → 1
0 → 2
```

So:

```text
dp = [0, 1, 1, INF, INF]
```

From index `1`:

```text
1 → 2
1 → 3
1 → 4
```

Update:

```text
dp = [0, 1, 1, 2, 2]
```

Therefore:

```text
dp[4] = 2
```

Answer:

```text
2
```

---

### Algorithm

1. Create a `dp` array of size `n`.
2. Initialize all values to infinity.
3. Set:

   ```cpp
   dp[0] = 0;
   ```
4. For each index `i`:

   * Iterate through every reachable index from `i`.
   * Update the minimum jump count.
5. Return `dp[n - 1]`.

---

### Complexity

* **Time:** `O(n²)` worst case
* **Space:** `O(n)`

This is inferior to the greedy solution.

---

### Notes / Tips

DP is useful for understanding the problem, but the greedy solution should be preferred when asked for optimal complexity.

The key difference is:

```text
DP:
Calculate minimum cost for every position

Greedy:
Calculate the farthest boundary reachable
with the current number of jumps
```

---

### Code

```cpp
class Solution {
public:
    int jump(vector<int>& nums) {
        int n = nums.size();
        vector<int> dp(n, INT_MAX);

        dp[0] = 0;

        for (int i = 0; i < n; i++) {
            for (int j = 1; j <= nums[i] && i + j < n; j++) {
                dp[i + j] = min(dp[i + j], dp[i] + 1);
            }
        }

        return dp[n - 1];
    }
};
```

---

## Key Takeaways

### Core Greedy Template

```cpp
int jumps = 0;
int currentEnd = 0;
int farthest = 0;

for (int i = 0; i < n - 1; i++) {
    farthest = max(farthest, i + nums[i]);

    if (i == currentEnd) {
        jumps++;
        currentEnd = farthest;
    }
}
```

The three variables to remember:

```text
currentEnd
    ↓
Boundary of current jump

farthest
    ↓
Best boundary for next jump

jumps
    ↓
Number of jumps used
```

### Pattern

```text
Current Range
      ↓
Explore every index in range
      ↓
Find farthest reachable position
      ↓
Expand range
      ↓
Count one jump
```

### LeetCode 45 Mental Model

> **Treat every jump as a level. Process the entire current level, find the farthest position reachable from it, and use that as the next level's boundary.**

This gives the optimal:

```text
Time  → O(n)
Space → O(1)
```

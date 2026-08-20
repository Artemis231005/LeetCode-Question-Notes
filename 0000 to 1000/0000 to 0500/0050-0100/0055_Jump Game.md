# LeetCode 55 — Jump Game

## Metadata

* **LeetCode:** 55
* **Problem:** Jump Game
* **Difficulty:** Medium
* **Topics:** Array, Greedy, Dynamic Programming
* **Pattern:** Greedy
* **Key Pattern:** Track the farthest reachable index
* **Key Technique:** Update the maximum reachable position while traversing the array
* **Key Template:** Greedy Reachability
* **Optimal Complexity:** `O(n)` time, `O(1)` space

---

## Problem

You are given an integer array `nums`.

You start at index `0`.

`nums[i]` represents the **maximum number of steps** you can jump forward from index `i`.

Return `true` if you can reach the **last index**, otherwise return `false`.

Example:

```text
nums = [2,3,1,1,4]

Start at index 0
→ Can jump up to 2 positions
→ Reach index 1 or 2
→ From index 1, can jump up to 3 positions
→ Reach index 4

Answer = true
```

---

## Approach — Greedy

### Idea

Instead of trying every possible jump, keep track of:

```text
maxReach = farthest index we can currently reach
```

While traversing the array:

1. If the current index is greater than `maxReach`, it means this index is unreachable.
2. Otherwise, update:

```text
maxReach = max(maxReach, i + nums[i])
```

3. If `maxReach >= n - 1`, we can reach the last index.

The key idea is:

> We don't care exactly which path we take. We only care about the farthest position reachable so far.

---

### Dry Run

For:

```text
[2,3,1,1,4]
```

Start:

```text
maxReach = 0
```

| Index | `nums[i]` | `i + nums[i]` | `maxReach` |
| ----: | --------: | ------------: | ---------: |
|   `0` |       `2` |           `2` |        `2` |
|   `1` |       `3` |           `4` |        `4` |
|   `2` |       `1` |           `3` |        `4` |
|   `3` |       `1` |           `4` |        `4` |
|   `4` |       `4` |           `8` |        `8` |

At index `1`:

```text
maxReach = max(2, 1 + 3)
         = 4
```

Since:

```text
maxReach >= last index
4 >= 4
```

we can reach the end.

Answer:

```text
true
```

---

## Important Example — Cannot Reach End

```text
nums = [3,2,1,0,4]
```

Dry run:

```text
index 0 → maxReach = 3
index 1 → maxReach = 3
index 2 → maxReach = 3
index 3 → maxReach = 3
index 4 → unreachable
```

At index `4`:

```text
4 > maxReach
4 > 3
```

Therefore, index `4` cannot be reached.

Answer:

```text
false
```

The `0` at index `3` creates a dead end.

---

### Algorithm

1. Initialize:

   ```text
   maxReach = 0
   ```
2. Traverse the array from left to right.
3. For each index `i`:

   * If:

     ```text
     i > maxReach
     ```

     return `false`.
   * Otherwise update:

     ```text
     maxReach = max(maxReach, i + nums[i])
     ```
4. If `maxReach >= n - 1`, return `true`.
5. If the loop finishes without reaching the last index, return `false`.

---

### Complexity

* **Time:** `O(n)` — each index is processed once.
* **Space:** `O(1)` — only `maxReach` is maintained.

---

### Notes / Tips

* This is a classic **Greedy Reachability** problem.
* You do **not** need to simulate every possible jump.
* The most important variable is:

```text
maxReach = farthest index reachable so far
```

* The key condition for an unreachable position is:

```text
i > maxReach
```

* Compare **Jump Game (55)** with **Jump Game II (45)**:

```text
55 → Can I reach the end?
     Return true / false

45 → Minimum jumps needed?
     Return minimum number
```

* For Jump Game II, the greedy logic is slightly different because we track the current jump boundary and number of jumps.

* A useful mental model:

```text
Current reachable range
        ↓
Look at every index inside it
        ↓
Find the farthest next reach
        ↓
Expand reachable range
```

---

### Code

```cpp
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int maxReach = 0;

        for (int i = 0; i < nums.size(); i++) {
            if (i > maxReach) {
                return false;
            }

            maxReach = max(maxReach, i + nums[i]);

            if (maxReach >= nums.size() - 1) {
                return true;
            }
        }

        return true;
    }
};
```

---

## Quick Revision

```text
Jump Game
    ↓
Track farthest reachable index
    ↓
maxReach = max(maxReach, i + nums[i])
    ↓
If i > maxReach
    → unreachable
    → false
    ↓
If maxReach >= n - 1
    → true
```

### Core Template

```text
maxReach = 0

for i = 0 → n-1:
    if i > maxReach:
        return false

    maxReach = max(maxReach, i + nums[i])

    if maxReach >= n - 1:
        return true

return true
```

### Key Insight

```text
Don't ask:
"Which jump should I take?"

Ask:
"How far can I reach from everything reachable so far?"
```

**Pattern to remember:**
**Can reach end → Greedy → Track `maxReach`**

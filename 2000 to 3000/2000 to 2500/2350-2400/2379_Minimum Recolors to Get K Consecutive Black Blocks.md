# LeetCode 2379 — Minimum Recolors to Get K Consecutive Black Blocks

## Metadata

* **LeetCode:** 2379
* **Problem:** Minimum Recolors to Get K Consecutive Black Blocks
* **Difficulty:** Easy
* **Topics:** String, Sliding Window
* **Pattern:** Fixed-Size Sliding Window (Count of Whites per Window)
* **Key Technique:** The number of recolors needed for any window of length `k` is just the number of white blocks inside it — slide the window across the string, updating the white count incrementally as blocks enter and leave
* **Optimal Complexity:** `O(n)` Time, `O(1)` Space

---

## Problem Statement

Given a string `blocks` of `'W'` (white) and `'B'` (black) characters and an integer `k`, in one operation you can recolor a white block to black. Return the minimum number of operations needed so that there is at least one run of `k` consecutive black blocks.

---

## Approaches

1. **Brute Force — Count Whites in Every Window Directly**
2. **Optimal — Sliding Window with Running White Count**

---

# Approach 1 — Brute Force / Count Whites in Every Window Directly

## Idea

For every window of length `k`, scan the entire window and count how many `'W'` blocks it contains — that count is exactly how many recolors that window needs. Track the minimum across all windows.

## Dry Run

```text
blocks = "WBBWWBBWBW", k = 7
```

Window `[0..6]` = `"WBBWWBB"`:

```text
whites: W at index 0, 3, 4 → count = 3
```

Window `[1..7]` = `"BBWWBBW"`:

```text
whites: indices 3, 4, 7 → count = 3
```

Window `[2..8]` = `"BWWBBWB"`:

```text
whites: 3 → count = 3
```

Window `[3..9]` = `"WWBBWBW"`:

```text
whites: 4 → count = 4
```

Minimum: `3`.

## Algorithm

1. Initialize `best = infinity`.
2. For each start index `i` from `0` to `n - k`:

   * Count the number of `'W'` characters in `blocks[i .. i+k-1]`.
   * Update `best = min(best, count)`.
3. Return `best`.

## Complexity

* **Time:** `O(n * k)`

  * For each of the `n - k + 1` windows, scanning all `k` characters to count whites.
* **Space:** `O(1)`

  * Only a running count and minimum.

## Notes / Tips

* Recounting every character of each window from scratch is redundant — consecutive windows overlap in `k - 1` characters, so the white count can be updated by only accounting for the one character leaving and the one entering.

## Code

```cpp
class Solution {
public:
    int minimumRecolors(string blocks, int k) {
        int n = blocks.size();
        int best = INT_MAX;

        for (int i = 0; i + k <= n; i++) {
            int whites = 0;
            for (int j = i; j < i + k; j++) {
                if (blocks[j] == 'W') {
                    whites++;
                }
            }
            best = min(best, whites);
        }

        return best;
    }
};
```

---

# Approach 2 — Optimal / Sliding Window with Running White Count

## Idea

Count the whites in the first window of length `k` once. Then slide the window one position at a time: subtract `1` if the block leaving the window (on the left) was white, and add `1` if the block entering (on the right) is white. Track the minimum white count seen across all window positions — that minimum is the answer.

## Dry Run

```text
blocks = "WBBWWBBWBW", k = 7
```

First window `[0..6]` = `"WBBWWBB"`:

```text
whites = 3 → best = 3
```

Slide to `[1..7]` (remove index 0 = 'W', add index 7 = 'W'):

```text
whites = 3 - 1 + 1 = 3 → best stays 3
```

Slide to `[2..8]` (remove index 1 = 'B', add index 8 = 'B'):

```text
whites = 3 - 0 + 0 = 3 → best stays 3
```

Slide to `[3..9]` (remove index 2 = 'B', add index 9 = 'W'):

```text
whites = 3 - 0 + 1 = 4 → best stays 3
```

Final answer: `3`, matching the brute-force result.

## Algorithm

1. Count the number of `'W'` in the first `k` characters, store as `whites`, and set `best = whites`.
2. For each `i` from `k` to `n - 1`:

   * If `blocks[i - k] == 'W'`, decrement `whites` (that block left the window).
   * If `blocks[i] == 'W'`, increment `whites` (that block entered the window).
   * Update `best = min(best, whites)`.
3. Return `best`.

## Complexity

* **Time:** `O(n)`

  * One pass to build the first window (`O(k)`), then `O(1)` per slide.
* **Space:** `O(1)`

  * Only a running white count and minimum — no extra structures.

## Notes / Tips

* This is the standard fixed-size sliding window template: maintain a window's aggregate value incrementally by adding the element that enters and removing the element that leaves, instead of recomputing the aggregate from scratch at each position.
* The answer is the minimum number of whites in any window because each white block in the chosen window needs exactly one recolor to become black, and black blocks need none.


## Code

```cpp
class Solution {
public:
    int minimumRecolors(string blocks, int k) {
        int n = blocks.size();
        int whites = 0;

        for (int i = 0; i < k; i++) {
            if (blocks[i] == 'W') {
                whites++;
            }
        }

        int best = whites;

        for (int i = k; i < n; i++) {
            if (blocks[i - k] == 'W') {
                whites--;
            }
            if (blocks[i] == 'W') {
                whites++;
            }
            best = min(best, whites);
        }

        return best;
    }
};
```

---

## Key Template

```text
whites = count of 'W' in first k characters
best = whites

for i in k..n-1:
    if blocks[i - k] == 'W': whites -= 1
    if blocks[i] == 'W': whites += 1
    best = min(best, whites)

return best
```
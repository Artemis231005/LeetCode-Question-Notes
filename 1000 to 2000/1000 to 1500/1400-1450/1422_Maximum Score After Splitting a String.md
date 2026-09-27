# LeetCode 1422 — Maximum Score After Splitting a String

## Metadata

* **LeetCode:** 1422
* **Problem:** Maximum Score After Splitting a String
* **Difficulty:** Easy
* **Topics:** String, Prefix Sum
* **Pattern:** Prefix Sum (Running Left Count + Total Count)
* **Key Technique:** Track the total number of `1`s in the string and a running count of `0`s seen so far while scanning left to right — at each split point, the score is derived directly from these two running values instead of rescanning both halves
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given a binary string `s`, split it into two non-empty substrings (a left part and a right part) at some index. The score of a split is the number of `0`s in the left substring plus the number of `1`s in the right substring. Return the maximum achievable score over all possible splits.

---

## Approaches

1. **Brute Force — Count Both Halves for Every Split Point**
2. **Optimal — Running Zero Count + Total One Count**

---

# Approach 1 — Brute Force / Count Both Halves for Every Split Point

## Idea

For every possible split point, scan the left substring to count its `0`s and scan the right substring to count its `1`s, then sum the two counts and track the maximum.

## Dry Run

```text
s = "011101"
```

Split after index `0` (left=`"0"`, right=`"11101"`):

```text
zeros in left = 1
ones in right = 4
score = 5
```

Split after index `2` (left=`"011"`, right=`"101"`):

```text
zeros in left = 1
ones in right = 2
score = 3
```

Continue checking every other split point — the best found is `5`.

## Algorithm

1. Initialize `best = -infinity`.
2. For each split point `i` from `1` to `n-1` (left substring is `s[0..i-1]`, right is `s[i..n-1]`):

   * Count `0`s in `s[0..i-1]`.
   * Count `1`s in `s[i..n-1]`.
   * Update `best = max(best, zeros + ones)`.
3. Return `best`.

## Complexity

* **Time:** `O(n²)`

  * Each of the `n-1` split points triggers two fresh scans covering up to `n` characters combined.
* **Space:** `O(1)`

  * Only a couple of running counters per split point.

## Notes / Tips

* Rescanning both halves from scratch at every split point is redundant — as the split point moves one step to the right, the left half either gains a `0` (increase zero count by 1) or gains a `1` (zero count unchanged), and the right half loses whatever character just left it. Tracking these running counts avoids rescanning entirely.

## Code

```cpp
class Solution {
public:
    int maxScore(string s) {
        int n = s.size();
        int best = INT_MIN;

        for (int i = 1; i < n; i++) {
            int zeros = 0;
            for (int j = 0; j < i; j++) {
                if (s[j] == '0') zeros++;
            }

            int ones = 0;
            for (int j = i; j < n; j++) {
                if (s[j] == '1') ones++;
            }

            best = max(best, zeros + ones);
        }

        return best;
    }
};
```

---

# Approach 2 — Optimal / Running Zero Count + Total One Count

## Idea

First count the total number of `1`s in the entire string once. Then scan left to right, maintaining a running count of `0`s seen so far (the left substring's zero count). At each split point, the right substring's `1` count is simply `totalOnes` minus however many `1`s have already been counted on the left side (which can be derived as `index - zerosSoFar`, i.e. non-zero characters seen so far are all `1`s). This lets the score at every split point be computed in `O(1)` as the scan proceeds.

## Dry Run

```text
s = "011101"
```

Total `1`s in the whole string: `4`.

Scan left to right, tracking `zerosSoFar` and `onesSoFar` (characters processed into the left half):

```text
i=1 (split after index 0, left="0"):
   process s[0]='0' → zerosSoFar=1, onesSoFar=0
   onesInRight = totalOnes - onesSoFar = 4 - 0 = 4
   score = zerosSoFar + onesInRight = 1 + 4 = 5

i=2 (split after index 1, left="01"):
   process s[1]='1' → zerosSoFar=1, onesSoFar=1
   onesInRight = 4 - 1 = 3
   score = 1 + 3 = 4

i=3 (split after index 2, left="011"):
   process s[2]='1' → zerosSoFar=1, onesSoFar=2
   onesInRight = 4 - 2 = 2
   score = 1 + 2 = 3
```

Continuing through the remaining split points, the maximum found is `5`, matching the brute-force result.

## Algorithm

1. Compute `totalOnes` by counting all `1`s in `s`.
2. Initialize `zerosSoFar = 0`, `onesSoFar = 0`, `best = -infinity`.
3. For each index `i` from `0` to `n - 2` (each represents extending the left substring by one more character, then evaluating the split just after it):

   * If `s[i] == '0'`, increment `zerosSoFar`; otherwise increment `onesSoFar`.
   * `onesInRight = totalOnes - onesSoFar`.
   * `score = zerosSoFar + onesInRight`.
   * Update `best = max(best, score)`.
4. Return `best`.

## Complexity

* **Time:** `O(n)`

  * One pass to count `totalOnes`, one pass to scan and evaluate every split point.
* **Space:** `O(1)`

  * Only a handful of running counters — no extra structures.

## Notes / Tips

* The loop bound `i` from `0` to `n-2` (not `n-1`) is what ensures the right substring is always non-empty — a split after the very last character would leave nothing on the right, which the problem disallows.
* This is the same "running left value + total minus left value" identity used in LC 724 (Find Pivot Index) and LC 1991 — just applied to counting characters by type instead of summing numeric values.
* Deriving `onesInRight` from `totalOnes - onesSoFar` (rather than tracking a separate running "ones in right" counter that decrements) is a matter of style — either works, but computing it from the total avoids needing to initialize and decrement a second counter.

## Code

```cpp
class Solution {
public:
    int maxScore(string s) {
        int n = s.size();
        int totalOnes = 0;

        for (char c : s) {
            if (c == '1') totalOnes++;
        }

        int zerosSoFar = 0, onesSoFar = 0;
        int best = INT_MIN;

        for (int i = 0; i < n - 1; i++) {
            if (s[i] == '0') {
                zerosSoFar++;
            } else {
                onesSoFar++;
            }

            int onesInRight = totalOnes - onesSoFar;
            int score = zerosSoFar + onesInRight;

            best = max(best, score);
        }

        return best;
    }
};
```

---

## Key Template

```text
totalOnes = count of '1' in s

zerosSoFar = 0
onesSoFar = 0
best = -infinity

for i in 0..n-2:
    if s[i] == '0': zerosSoFar += 1
    else: onesSoFar += 1

    onesInRight = totalOnes - onesSoFar
    score = zerosSoFar + onesInRight
    best = max(best, score)

return best
```
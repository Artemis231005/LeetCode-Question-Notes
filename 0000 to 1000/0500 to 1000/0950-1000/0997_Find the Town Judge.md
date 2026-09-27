# LeetCode 997 — Find the Town Judge

## Metadata

* **LeetCode:** 997
* **Problem:** Find the Town Judge
* **Difficulty:** Easy
* **Topics:** Array, Hash Table, Graph
* **Pattern:** In-Degree / Out-Degree Score Counting
* **Key Technique:** The judge is the one person trusted by everyone else and who trusts no one — encode this as a single running score per person (`+1` when trusted, `-1` when trusting someone), and the judge is exactly whoever ends with a score of `n - 1`
* **Optimal Complexity:** `O(n + t)` Time, `O(n)` Auxiliary Space

---

## Problem Statement

In a town of `n` people labeled `1` to `n`, given a list of `trust` pairs `[a, b]` (meaning `a` trusts `b`), find the town judge — the one person trusted by everyone else, who trusts nobody themself. Return their label, or `-1` if no such person exists.

---

## Approaches

1. **Brute Force — Check Every Candidate Directly**
2. **Optimal — Single Score Array (Trusted Count Minus Trusting Count)**

---

# Approach 1 — Brute Force / Check Every Candidate Directly

## Idea

For each candidate person, scan the entire `trust` list twice: once to confirm they trust nobody (they never appear as the truster, `a`), and once to confirm every one of the other `n-1` people trusts them (they appear as the trustee, `b`, exactly `n-1` times).

## Dry Run

```text
n = 3, trust = [[1,3],[2,3]]
```

Check candidate `1`:

```text
does 1 trust anyone? scan trust for a==1 → [1,3] found → 1 trusts someone → not the judge
```

Check candidate `2`:

```text
does 2 trust anyone? scan trust for a==2 → [2,3] found → not the judge
```

Check candidate `3`:

```text
does 3 trust anyone? scan trust for a==3 → none found → passes first check
how many people trust 3? scan trust for b==3 → [1,3] and [2,3] → count=2
n-1 = 2 → matches → 3 is the judge
```

Return `3`.

## Algorithm

1. For each candidate `c` from `1` to `n`:

   * Scan `trust` to check if `c` ever appears as `a` (trusts someone) — if so, `c` isn't the judge.
   * Scan `trust` to count how many pairs have `c` as `b` (trusted by someone) — if this count equals `n - 1`, `c` is the judge.
2. If no candidate qualifies, return `-1`.

## Complexity

* **Time:** `O(n * t)`

  * For each of the `n` candidates, up to two full scans of the `trust` list (of length `t`).
* **Space:** `O(1)`

  * Only a couple of counters per candidate.

## Notes / Tips

* Rechecking the entire `trust` list from scratch for every candidate is redundant — each trust pair only needs to be processed once overall to determine everyone's "trusts someone" status and "trusted-by count" simultaneously, which a single score array (Approach 2) achieves directly.

## Code

```cpp
class Solution {
public:
    int findJudge(int n, vector<vector<int>>& trust) {
        for (int c = 1; c <= n; c++) {
            bool trustsSomeone = false;
            for (auto& t : trust) {
                if (t[0] == c) {
                    trustsSomeone = true;
                    break;
                }
            }

            if (trustsSomeone) continue;

            int trustedByCount = 0;
            for (auto& t : trust) {
                if (t[1] == c) {
                    trustedByCount++;
                }
            }

            if (trustedByCount == n - 1) {
                return c;
            }
        }

        return -1;
    }
};
```

---

# Approach 2 — Optimal / Single Score Array (Trusted Count Minus Trusting Count)

## Idea

Maintain one running `score` per person. For each trust pair `[a, b]`: increment `score[b]` (they gained someone's trust) and decrement `score[a]` (they spent their trust on someone). The town judge, if one exists, is trusted by everyone else (`n-1` increments) and trusts nobody (`0` decrements), so their final score is exactly `n - 1` — and no one else can reach that score, since anyone who trusts even one person has their own score reduced below what a "trusts nobody" person could achieve.

## Dry Run

```text
n = 3, trust = [[1,3],[2,3]]
```

Initialize `score = [0, 0, 0, 0]` (1-indexed, index `0` unused).

```text
[1,3]: score[3] += 1 → score[3]=1; score[1] -= 1 → score[1]=-1
[2,3]: score[3] += 1 → score[3]=2; score[2] -= 1 → score[2]=-1
```

Final `score = [_, -1, -1, 2]`.

Check for `score[c] == n - 1 = 2`:

```text
score[3] = 2 → matches → judge is 3
```

Return `3`, matching the brute-force result.

## Algorithm

1. Initialize a `score` array of size `n + 1`, all zeros.
2. For each trust pair `[a, b]`:

   * `score[b] += 1`.
   * `score[a] -= 1`.
3. For each person `c` from `1` to `n`:

   * If `score[c] == n - 1`, return `c`.
4. If no one qualifies, return `-1`.

## Complexity

* **Time:** `O(n + t)`

  * `O(t)` to process all trust pairs, `O(n)` to scan for the qualifying score.
* **Space:** `O(n)`

  * For the `score` array.

## Notes / Tips

* The single `+1`/`-1` scoring trick works because it captures both conditions ("trusted by everyone" and "trusts nobody") in one combined value — a person with even one outgoing trust relationship immediately drops their score below `n-1`, no matter how many people trust them, so `score[c] == n-1` alone is sufficient to identify the judge without separate checks.
* Special case worth double-checking: with `n = 1` and no trust pairs, the single person trivially satisfies "trusted by everyone else" (there's no one else) and "trusts nobody" — their score stays `0`, which correctly equals `n - 1 = 0`.
* This is conceptually the same "in-degree minus out-degree" idea used to characterize special vertices in a directed graph (like finding a source or sink) — the judge is exactly a graph "sink" with an in-degree of `n-1` and an out-degree of `0`.

## Code

```cpp
class Solution {
public:
    int findJudge(int n, vector<vector<int>>& trust) {
        vector<int> score(n + 1, 0);

        for (auto& t : trust) {
            score[t[1]] += 1;
            score[t[0]] -= 1;
        }

        for (int c = 1; c <= n; c++) {
            if (score[c] == n - 1) {
                return c;
            }
        }

        return -1;
    }
};
```

---

## Key Template

```text
score = array of size (n + 1), all 0

for [a, b] in trust:
    score[b] += 1
    score[a] -= 1

for c in 1..n:
    if score[c] == n - 1:
        return c

return -1
```
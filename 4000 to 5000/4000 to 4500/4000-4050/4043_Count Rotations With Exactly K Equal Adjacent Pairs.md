# LeetCode 4043 — Count Rotations With Exactly K Equal Adjacent Pairs

## Metadata

* **LeetCode:** 4043
* **Problem:** Count Rotations With Exactly K Equal Adjacent Pairs
* **Difficulty:** Easy
* **Topics:** String, Sliding Window, Prefix Sum
* **Pattern:** String Doubling + Prefix Sum Sliding Window
* **Key Technique:** Double the string to represent all rotations as fixed-length windows, precompute adjacent-equality flags once, then use a prefix sum to get each rotation's score in O(1)
* **Optimal Complexity:** `O(n)` Time, `O(n)` Space

---

## Problem Statement

Given a string `s` of length `n` and an integer `k`, a cyclic rotation moves some prefix of `s` to the end. For each cyclic rotation, its "score" is the number of adjacent positions with equal characters. Return the number of cyclic rotations whose score equals `k`.

---

## Approaches

1. **Brute Force — Build Each Rotation and Count Directly**
2. **Optimal — Doubled String + Prefix Sum Sliding Window**

---

# Approach 1 — Brute Force / Build Each Rotation and Count Directly

## Idea

For every possible rotation (`n` of them), physically construct the rotated string by slicing and concatenating, then scan it once to count adjacent equal-character pairs. Compare that count to `k`.

## Dry Run

```text
s = "aab", k = 1
```

Rotation `r = 0`:

```text
rotated = "aab"
count adjacent equal pairs: (a,a) equal, (a,b) not → score = 1
```

Rotation `r = 1`:

```text
rotated = "aba"
(a,b) not, (b,a) not → score = 0
```

Rotation `r = 2`:

```text
rotated = "baa"
(b,a) not, (a,a) equal → score = 1
```

Rotations with score `1`: `r=0` and `r=2` → count = `2`.

## Algorithm

1. Initialize `count = 0`.
2. For each rotation start `r` from `0` to `n-1`:

   * Build `rotated = s.substr(r) + s.substr(0, r)`.
   * Scan `rotated` and count adjacent equal-character pairs into `score`.
   * If `score == k`, increment `count`.
3. Return `count`.

## Complexity

* **Time:** `O(n²)`

  * For each of the `n` rotations, building the rotated string and scanning it both take `O(n)`.
* **Space:** `O(1)`

  * Beyond the temporary rotated string (which doesn't scale with input growth in a way that changes the algorithm's shape), only a couple of counters are tracked.

## Notes / Tips

* Rebuilding the full rotated string for every rotation is unnecessary — a rotation starting at `r` is just a fixed-length window into `s + s`, which avoids any actual string construction.
* Given `n <= 100`, this brute force comfortably fits within typical time limits, but it's still worth recognizing the redundant work being repeated across rotations.

## Code

```cpp
class Solution {
public:
    int countRotations(string s, int k) {
        int n = s.size();
        int count = 0;

        for (int r = 0; r < n; r++) {
            string rotated = s.substr(r) + s.substr(0, r);
            int score = 0;

            for (int i = 0; i < n - 1; i++) {
                if (rotated[i] == rotated[i + 1]) {
                    score++;
                }
            }

            if (score == k) {
                count++;
            }
        }

        return count;
    }
};
```

---

# Approach 2 — Optimal / Doubled String + Prefix Sum Sliding Window

## Idea

Concatenate `s` with itself to form `doubled = s + s`. Every cyclic rotation starting at index `r` corresponds exactly to the substring `doubled[r .. r+n-1]` — no actual rotation needs to be built. Precompute an `equal` array where `equal[i] = 1` if `doubled[i] == doubled[i+1]`, else `0`, covering every adjacent pair in the doubled string. A rotation's score is then just the sum of `equal[r .. r+n-2]` (the `n-1` adjacent comparisons within that rotation's window) — a fixed-size window sum, computable in `O(1)` per rotation using a prefix sum over `equal`.

## Dry Run

```text
s = "aab", k = 1
```

```text
doubled = "aabaab"
```

Build `equal` (comparing each adjacent pair in `doubled`):

```text
equal[0] = (a==a) = 1
equal[1] = (a==b) = 0
equal[2] = (b==a) = 0
equal[3] = (a==a) = 1
equal[4] = (a==b) = 0
```

```text
equal = [1, 0, 0, 1, 0]
```

Window size `n-1 = 2`. For each rotation `r`, sum `equal[r .. r+1]`:

```text
r=0: equal[0]+equal[1] = 1+0 = 1 → matches k=1 → count=1
r=1: equal[1]+equal[2] = 0+0 = 0 → no
r=2: equal[2]+equal[3] = 0+1 = 1 → matches k=1 → count=2
```

Final count: `2`, matching the brute-force result.

## Algorithm

1. Build `doubled = s + s` (length `2n`).
2. Build `equal` array of size `2n - 1`, where `equal[i] = (doubled[i] == doubled[i+1]) ? 1 : 0`.
3. Build a prefix sum array `prefix` over `equal`, where `prefix[i]` holds the sum of `equal[0..i-1]`.
4. For each rotation `r` from `0` to `n-1`:

   * `score = prefix[r + n - 1] - prefix[r]` (sum of the window `equal[r .. r+n-2]`).
   * If `score == k`, increment `count`.
5. Return `count`.

## Complexity

* **Time:** `O(n)`

  * Building `doubled`, `equal`, and `prefix` are each `O(n)`; checking all `n` rotations afterward is `O(1)` each.
* **Space:** `O(n)`

  * For the doubled string, the `equal` array, and the prefix sum array — all linear in `n`.

## Notes / Tips

* This is the standard "double the string to handle rotations/circularity" trick, combined with a prefix sum to turn repeated window-sum queries into `O(1)` lookups — the same overall shape as LC 503 (Next Greater Element II) applied to a sum instead of a monotonic stack.
* The window size is `n - 1`, not `n` — a rotation of length `n` has exactly `n - 1` adjacent character pairs to compare, which is what `equal[r .. r+n-2]` captures.
* Building the `equal` array directly on the doubled string avoids ever materializing an actual rotated substring — the rotation is entirely conceptual, expressed only as an index offset into `doubled`.

## Code

```cpp
class Solution {
public:
    int countRotations(string s, int k) {
        int n = s.size();
        string doubled = s + s;

        vector<int> equalArr(2 * n - 1, 0);
        for (int i = 0; i < 2 * n - 1; i++) {
            equalArr[i] = (doubled[i] == doubled[i + 1]) ? 1 : 0;
        }

        vector<int> prefix(2 * n, 0);
        for (int i = 0; i < 2 * n - 1; i++) {
            prefix[i + 1] = prefix[i] + equalArr[i];
        }

        int count = 0;
        for (int r = 0; r < n; r++) {
            int score = prefix[r + n - 1] - prefix[r];
            if (score == k) {
                count++;
            }
        }

        return count;
    }
};
```

---

## Key Template

```text
doubled = s + s
equal[i] = 1 if doubled[i] == doubled[i+1] else 0, for i in 0..2n-2

prefix[0] = 0
for i in 0..2n-2:
    prefix[i+1] = prefix[i] + equal[i]

count = 0
for r in 0..n-1:
    score = prefix[r + n - 1] - prefix[r]
    if score == k:
        count += 1

return count
```
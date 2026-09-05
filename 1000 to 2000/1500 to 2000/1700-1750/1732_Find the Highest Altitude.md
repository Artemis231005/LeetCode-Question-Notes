# LeetCode 1732 — Find the Highest Altitude

## Metadata

* **LeetCode:** 1732
* **Problem:** Find the Highest Altitude
* **Difficulty:** Easy
* **Topics:** Array, Prefix Sum
* **Pattern:** Prefix Sum
* **Key Technique:** Track a running altitude while scanning gains, keeping the maximum seen so far
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given an array `gain` where `gain[i]` is the net altitude change between point `i` and point `i+1`, and starting altitude is `0`, return the highest altitude reached at any point.

---

## Approaches

1. **Brute Force — Build the Full Altitude Array First**
2. **Optimal — Single-Pass Running Max**

---

# Approach 1 — Brute Force / Build the Full Altitude Array First

## Idea

Construct the entire altitude array first (starting at `0`, applying each gain in sequence), then scan it once to find the maximum.

## Dry Run

```text
gain = [-5, 1, 5, 0, -7]
```

Build altitudes:

```text
altitude[0] = 0
altitude[1] = 0 + (-5) = -5
altitude[2] = -5 + 1 = -4
altitude[3] = -4 + 5 = 1
altitude[4] = 1 + 0 = 1
altitude[5] = 1 + (-7) = -6
```

Altitudes:

```text
[0, -5, -4, 1, 1, -6]
```

Scan for max:

```text
max = 1
```

## Algorithm

1. Create an `altitude` array of size `n + 1`, with `altitude[0] = 0`.
2. For each index `i` from `0` to `n-1`: `altitude[i+1] = altitude[i] + gain[i]`.
3. Scan `altitude` to find and return the maximum value.

## Complexity

* **Time:** `O(n)`

  * One pass to build the altitude array, one pass to find the max — both linear.
* **Space:** `O(n)`

  * The full altitude array is stored, even though only its maximum is ever used.

## Notes / Tips

* Storing the entire altitude history is unnecessary — only the running value and the running maximum matter at any point.
* Still linear time, but the extra array is avoidable.

## Code

```cpp
class Solution {
public:
    int largestAltitude(vector<int>& gain) {
        int n = gain.size();
        vector<int> altitude(n + 1, 0);

        for (int i = 0; i < n; i++) {
            altitude[i + 1] = altitude[i] + gain[i];
        }

        int maxAlt = altitude[0];
        for (int alt : altitude) {
            maxAlt = max(maxAlt, alt);
        }

        return maxAlt;
    }
};
```

---

# Approach 2 — Optimal / Single-Pass Running Max

## Idea

There's no need to store every altitude — just keep a running current altitude and update a running maximum as you go, in a single pass over `gain`.

## Dry Run

```text
gain = [-5, 1, 5, 0, -7]
```

Process:

```text
start: altitude = 0, maxAlt = 0

-5 → altitude = -5 → maxAlt stays 0
1  → altitude = -4 → maxAlt stays 0
5  → altitude = 1  → maxAlt = 1
0  → altitude = 1  → maxAlt stays 1
-7 → altitude = -6 → maxAlt stays 1
```

Final:

```text
maxAlt = 1
```

## Algorithm

1. Initialize `altitude = 0` and `maxAlt = 0` (starting point counts too).
2. For each value `g` in `gain`:

   * `altitude += g`.
   * `maxAlt = max(maxAlt, altitude)`.
3. Return `maxAlt`.

## Complexity

* **Time:** `O(n)`

  * Single pass, each gain processed exactly once.
* **Space:** `O(1)`

  * Only two running variables (`altitude`, `maxAlt`) — no array needed at all.

## Notes / Tips

* Starting `maxAlt` at `0` (not the first computed altitude) is what correctly accounts for the starting point itself possibly being the highest.
* This is a running prefix-sum-with-max pattern — same accumulation idea as LC 1480 (Running Sum), just tracking the max instead of storing every value.
* Works fine with negative gains throughout — the running max only ever increases when a new prefix sum exceeds it.

## Code

```cpp
class Solution {
public:
    int largestAltitude(vector<int>& gain) {
        int altitude = 0, maxAlt = 0;

        for (int g : gain) {
            altitude += g;
            maxAlt = max(maxAlt, altitude);
        }

        return maxAlt;
    }
};
```

---

## Key Template

```text
current = 0
maxSoFar = 0

for g in gain:
    current += g
    maxSoFar = max(maxSoFar, current)

return maxSoFar
```
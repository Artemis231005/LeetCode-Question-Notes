# LeetCode 1094 — Car Pooling

## Metadata

* **LeetCode:** 1094
* **Problem:** Car Pooling
* **Difficulty:** Medium
* **Topics:** Array, Sorting, Simulation, Prefix Sum
* **Pattern:** Difference Array (Range Update, Point Query)
* **Key Technique:** Mark `+numPassengers` at each trip's pickup point and `-numPassengers` at its dropoff point, then a running prefix sum gives the car's passenger count at every point — check it never exceeds capacity
* **Optimal Complexity:** `O(n + maxTrip)` Time, `O(maxTrip)` Space

---

## Problem Statement

Given a list of trips `[numPassengersi, fromi, toi]` (each picking up `numPassengersi` passengers at location `fromi` and dropping them off at location `toi`), and an integer `capacity`, return `true` if the car can complete all trips without exceeding `capacity` at any point.

---

## Approaches

1. **Brute Force — Apply Every Trip to Every Point It Covers**
2. **Optimal — Difference Array**

---

# Approach 1 — Brute Force / Apply Every Trip to Every Point It Covers

## Idea

Use an array covering the possible location range. For each trip, directly add `numPassengers` to every location from `from` up to (but not including) `to` — since a passenger dropped off at `to` is no longer in the car at that point.

## Dry Run

```text
trips = [[2,1,5],[3,3,7]], capacity = 4
```

Apply `[2,1,5]` (add 2 to locations 1-4):

```text
count[1..4] += 2
```

Apply `[3,3,7]` (add 3 to locations 3-6):

```text
count[3..6] += 3
```

Resulting counts at each location:

```text
1: 2
2: 2
3: 2+3=5
4: 2+3=5
5: 3
6: 3
```

At location `3`, count `5` exceeds `capacity = 4` → return `false`.

## Algorithm

1. Create a `count` array of size `maxLocation + 1`, all zeros.
2. For each trip `[numPassengers, from, to]`:

   * For `i` from `from` to `to - 1`:

     * `count[i] += numPassengers`.
3. Check every value in `count`: if any exceeds `capacity`, return `false`.
4. If all values stay within `capacity`, return `true`.

## Complexity

* **Time:** `O(n * maxTrip)`

  * Each of the `n` trips can directly touch up to `maxTrip` locations.
* **Space:** `O(maxTrip)`

  * For the `count` array, sized to the maximum possible location.

## Notes / Tips

* Directly adding to every location a trip covers is redundant — a trip's passenger effect only needs to be marked once, at its pickup and dropoff points, rather than applied to every point in between.
* The exclusive `to` boundary (stopping at `to - 1`, not `to`) is what correctly models a passenger being dropped off — they're no longer in the car once the car reaches that point.

## Code

```cpp
class Solution {
public:
    bool carPooling(vector<vector<int>>& trips, int capacity) {
        vector<int> count(1001, 0);

        for (auto& trip : trips) {
            int passengers = trip[0], from = trip[1], to = trip[2];

            for (int i = from; i < to; i++) {
                count[i] += passengers;
                if (count[i] > capacity) {
                    return false;
                }
            }
        }

        return true;
    }
};
```

---

# Approach 2 — Optimal / Difference Array

## Idea

Instead of adding `numPassengers` to every location a trip covers, mark the trip's *effect* directly: add `numPassengers` at the pickup location `from`, and subtract `numPassengers` at the dropoff location `to` (not `to + 1`, since the passenger has already left by the time the car reaches `to`). Then sweep through the array once, accumulating a running sum — that running sum is the car's current passenger count at every location, which must never exceed `capacity`.

## Dry Run

```text
trips = [[2,1,5],[3,3,7]], capacity = 4
```

Build difference array `diff`:

```text
[2,1,5] → diff[1] += 2, diff[5] -= 2
[3,3,7] → diff[3] += 3, diff[7] -= 3
```

```text
diff[1] = 2
diff[3] = 3
diff[5] = -2
diff[7] = -3
```

Prefix sum sweep:

```text
i=1: running = 2 → 2 <= 4, ok
i=2: running = 2 → ok
i=3: running = 2+3 = 5 → 5 > 4 → exceeds capacity → return false
```

Matches the brute-force result.

## Algorithm

1. Create a difference array `diff` of size `1001` (bounded by the problem's location constraints).
2. For each trip `[numPassengers, from, to]`:

   * `diff[from] += numPassengers`.
   * `diff[to] -= numPassengers`.
3. Sweep `i` from `0` to `1000`, maintaining a running sum:

   * `running += diff[i]`.
   * If `running > capacity`, return `false`.
4. If the sweep completes without exceeding capacity, return `true`.

## Complexity

* **Time:** `O(n + maxTrip)`

  * `O(n)` to apply all trip markers, `O(maxTrip)` to sweep through and accumulate the running sum.
* **Space:** `O(maxTrip)`

  * For the fixed-size difference array, independent of `n`.

## Notes / Tips

* Marking the dropoff at `diff[to]` (not `diff[to + 1]`) is the key detail here — unlike LC 1109 and LC 2848 where the range is inclusive on both ends, a car pooling trip's endpoint `to` should **not** count that passenger as still onboard, so the decrement applies exactly at `to`.
* Same difference array shape as LC 1109 (Corporate Flight Bookings) and LC 2848 (Points That Intersect With Cars) — the only structural difference is where the "end" marker lands, which depends on whether the range is inclusive or exclusive at that boundary.
* Checking `running > capacity` during the sweep (rather than after fully building the array) allows early exit the moment capacity is exceeded, without needing a separate pass.

## Code

```cpp
class Solution {
public:
    bool carPooling(vector<vector<int>>& trips, int capacity) {
        vector<int> diff(1001, 0);

        for (auto& trip : trips) {
            int passengers = trip[0], from = trip[1], to = trip[2];

            diff[from] += passengers;
            diff[to] -= passengers;
        }

        int running = 0;
        for (int i = 0; i <= 1000; i++) {
            running += diff[i];
            if (running > capacity) {
                return false;
            }
        }

        return true;
    }
};
```

---

## Key Template

```text
diff = array of size (maxLocation + 1), all 0

for [passengers, from, to] in trips:
    diff[from] += passengers
    diff[to] -= passengers

running = 0
for i = 0 to maxLocation:
    running += diff[i]
    if running > capacity:
        return false

return true
```
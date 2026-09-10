# LeetCode 4020 — Elevator Requests I

## Metadata

* **LeetCode:** 4020
* **Problem:** Elevator Requests I
* **Difficulty:** Easy
* **Topics:** Array, Simulation, Math
* **Pattern:** Sum of Consecutive Absolute Differences
* **Key Technique:** Total travel time is just the sum of `|requests[i] - requests[i-1]|`, with the elevator's start at floor `0` treated as an implicit first "previous" position
* **Optimal Complexity:** `O(m)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given an integer `n` (number of floors) and an array `requests` representing a sequence of floor requests, an elevator starts at floor `0`, moves one floor per second, and serves requests in order (moving directly from its current floor to each next requested floor). Return the total time in seconds required to serve all requests.

---

## Approaches

1. **Brute Force — Simulate Second by Second**
2. **Optimal — Sum of Consecutive Absolute Differences**

---

# Approach 1 — Brute Force / Simulate Second by Second

## Idea

Actually simulate the elevator moving one floor at a time. Track its current floor, and for each request, tick a counter up or down by one floor per second until the current floor matches the requested floor.

## Dry Run

```text
n = 5, requests = [2, 1, 4, 3]
```

Start at floor `0`, `time = 0`.

Serve request `2`:

```text
0 -> 1 (time=1) -> 2 (time=2) → reached
```

Serve request `1`:

```text
2 -> 1 (time=3) → reached
```

Serve request `4`:

```text
1 -> 2 (time=4) -> 3 (time=5) -> 4 (time=6) → reached
```

Serve request `3`:

```text
4 -> 3 (time=7) → reached
```

Total time: `7`.

## Algorithm

1. Initialize `currentFloor = 0`, `time = 0`.
2. For each `target` in `requests`:

   * While `currentFloor != target`:

     * Move one floor toward `target` (increment or decrement `currentFloor`).
     * Increment `time`.
3. Return `time`.

## Complexity

* **Time:** `O(n * m)`

  * For each of the `m` requests, the elevator may need up to `n` seconds (bounded by the number of floors) to reach it, simulated one floor at a time.
* **Space:** `O(1)`

  * Only `currentFloor` and `time` are tracked.

## Notes / Tips

* Simulating floor by floor is unnecessary work — the total seconds spent between any two floors is just their absolute difference, which can be computed directly without stepping through every intermediate floor.
* Still correct and useful for confirming the direct-difference approach produces the same result.

## Code

```cpp
class Solution {
public:
    int elevatorRequests(int n, vector<int>& requests) {
        int currentFloor = 0;
        int time = 0;

        for (int target : requests) {
            while (currentFloor != target) {
                if (currentFloor < target) {
                    currentFloor++;
                } else {
                    currentFloor--;
                }
                time++;
            }
        }

        return time;
    }
};
```

---

# Approach 2 — Optimal / Sum of Consecutive Absolute Differences

## Idea

Since the elevator moves in a straight line between consecutive requests, the time spent moving between two floors is exactly the absolute difference between them — no need to simulate each second. The first leg goes from floor `0` to `requests[0]`, and every subsequent leg goes from `requests[i-1]` to `requests[i]`. Summing all these absolute differences gives the total time directly.

## Dry Run

```text
n = 5, requests = [2, 1, 4, 3]
```

```text
leg 1: |2 - 0| = 2
leg 2: |1 - 2| = 1
leg 3: |4 - 1| = 3
leg 4: |3 - 4| = 1
```

Total:

```text
2 + 1 + 3 + 1 = 7
```

## Algorithm

1. Initialize `total = requests[0]` (covers the first leg from floor `0`).
2. For each `i` from `1` to `requests.size() - 1`:

   * `total += abs(requests[i] - requests[i-1])`.
3. Return `total`.

## Complexity

* **Time:** `O(m)`

  * A single pass over the `requests` array, one constant-time subtraction per element.
* **Space:** `O(1)`

  * Only a running total is tracked — no extra structures needed.

## Notes / Tips

* The starting floor `0` is effectively treated as a "request before the first request," which is why `requests[0]` alone (i.e. `|requests[0] - 0|`) seeds the total before the loop begins.
* No need for `n` (the number of floors) at all in the optimal solution — it only bounds the valid floor range, it doesn't affect the travel-time calculation.
* This is the same "sum of consecutive absolute differences" idea used in problems computing total path length or total distance traveled along a sequence of waypoints.

## Code

```cpp
class Solution {
public:
    int elevatorRequests(int n, vector<int>& requests) {
        int total = requests[0];

        for (int i = 1; i < requests.size(); i++) {
            total += abs(requests[i] - requests[i - 1]);
        }

        return total;
    }
};
```

---

## Key Template

```text
total = requests[0]

for i in 1..m-1:
    total += abs(requests[i] - requests[i-1])

return total
```
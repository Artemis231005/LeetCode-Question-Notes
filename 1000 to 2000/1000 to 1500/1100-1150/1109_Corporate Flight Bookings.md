# LeetCode 1109 — Corporate Flight Bookings

## Metadata

* **LeetCode:** 1109
* **Problem:** Corporate Flight Bookings
* **Difficulty:** Medium
* **Topics:** Array, Prefix Sum
* **Pattern:** Difference Array (Range Update, Point Query)
* **Key Technique:** Mark `+seats` at each booking's first flight and `-seats` just after its last flight, then a running prefix sum gives the total seats booked on every flight
* **Optimal Complexity:** `O(n + m)` Time, `O(n)` Auxiliary Space

---

## Problem Statement

Given `n` flights numbered `1` to `n`, and a list of bookings `[firsti, lasti, seatsi]` (each adding `seatsi` reserved seats to every flight from `firsti` to `lasti`, inclusive), return an array where `answer[i]` is the total number of seats reserved on flight `i + 1`.

---

## Approaches

1. **Brute Force — Apply Every Booking to Every Flight in Its Range**
2. **Optimal — Difference Array**

---

# Approach 1 — Brute Force / Apply Every Booking to Every Flight in Its Range

## Idea

For each booking, directly loop through every flight number from `first` to `last` and add `seats` to each one.

## Dry Run

```text
n = 5, bookings = [[1,2,10],[2,3,20],[2,5,25]]
```

Apply `[1,2,10]`:

```text
answer = [10, 10, 0, 0, 0]
```

Apply `[2,3,20]`:

```text
answer = [10, 30, 20, 0, 0]
```

Apply `[2,5,25]`:

```text
answer = [10, 55, 45, 25, 25]
```

Final:

```text
[10, 55, 45, 25, 25]
```

## Algorithm

1. Initialize `answer` array of size `n`, all zeros.
2. For each booking `[first, last, seats]`:

   * For `i` from `first - 1` to `last - 1` (converting to 0-indexed):

     * `answer[i] += seats`.
3. Return `answer`.

## Complexity

* **Time:** `O(n * m)`

  * Each of the `m` bookings can touch up to `n` flights directly.
* **Space:** `O(1)`

  * No extra structure beyond the required output array — no additional data structures allocated.

## Notes / Tips

* Directly applying each booking to every flight it covers is redundant — the *effect* of a range update only needs to be marked once, at its boundaries, rather than propagated to every index inside it.
* Fine for small `n * m`, but degrades badly when both bookings and the flight count grow.

## Code

```cpp
class Solution {
public:
    vector<int> corpFlightBookings(vector<vector<int>>& bookings, int n) {
        vector<int> answer(n, 0);

        for (auto& booking : bookings) {
            int first = booking[0], last = booking[1], seats = booking[2];

            for (int i = first - 1; i <= last - 1; i++) {
                answer[i] += seats;
            }
        }

        return answer;
    }
};
```

---

# Approach 2 — Optimal / Difference Array

## Idea

Instead of adding `seats` to every flight in a booking's range, mark the range's *effect* directly: add `seats` at the booking's first flight, and subtract `seats` just after its last flight. Then sweep through the array once, accumulating a running sum — at each flight, that running sum is exactly the total seats booked so far, which equals the answer for that flight.

## Dry Run

```text
n = 5, bookings = [[1,2,10],[2,3,20],[2,5,25]]
```

Build difference array `diff` of size `n + 1` (0-indexed, with an extra slot to safely mark `last`):

```text
[1,2,10] → diff[0] += 10, diff[2] -= 10
[2,3,20] → diff[1] += 20, diff[3] -= 20
[2,5,25] → diff[1] += 25, diff[5] -= 25
```

```text
diff = [10, 45, -10, -20, 0, -25]
```

Prefix sum sweep:

```text
i=0: running = 10 → answer[0] = 10
i=1: running = 10+45 = 55 → answer[1] = 55
i=2: running = 55-10 = 45 → answer[2] = 45
i=3: running = 45-20 = 25 → answer[3] = 25
i=4: running = 25+0 = 25 → answer[4] = 25
```

Final:

```text
[10, 55, 45, 25, 25]
```

Matches the brute-force result.

## Algorithm

1. Create a difference array `diff` of size `n + 1`, all zeros.
2. For each booking `[first, last, seats]` (1-indexed):

   * `diff[first - 1] += seats`.
   * `diff[last] -= seats` (this naturally falls one past the 0-indexed last flight).
3. Sweep `i` from `0` to `n - 1`, maintaining a running sum:

   * `running += diff[i]`.
   * `answer[i] = running`.
4. Return `answer`.

## Complexity

* **Time:** `O(n + m)`

  * `O(m)` to apply all booking markers, `O(n)` to sweep through and accumulate the running sum.
* **Space:** `O(n)`

  * For the difference array, sized independently of `m`.

## Notes / Tips

* This is the same difference array technique as LC 1893 — mark the start and one-past-the-end of each range once, then let a prefix sum sweep apply every range's effect in a single pass.
* Using `diff[last]` (not `diff[last - 1]`) as the subtraction point is what correctly handles the 1-indexed-to-0-indexed conversion — `last` in 1-indexed terms is `last - 1` in 0-indexed terms, and the "one past the end" marker lands at `(last - 1) + 1 = last`.
* The difference array here doubles directly as the output once the prefix sum sweep is applied — no separate result array is strictly needed if `diff` is reused in place.

## Code

```cpp
class Solution {
public:
    vector<int> corpFlightBookings(vector<vector<int>>& bookings, int n) {
        vector<int> diff(n + 1, 0);

        for (auto& booking : bookings) {
            int first = booking[0], last = booking[1], seats = booking[2];

            diff[first - 1] += seats;
            diff[last] -= seats;
        }

        vector<int> answer(n, 0);
        int running = 0;

        for (int i = 0; i < n; i++) {
            running += diff[i];
            answer[i] = running;
        }

        return answer;
    }
};
```

---

## Key Template

```text
diff = array of size (n + 1), all 0

for [first, last, seats] in bookings:
    diff[first - 1] += seats
    diff[last] -= seats

answer = array of size n
running = 0
for i = 0 to n-1:
    running += diff[i]
    answer[i] = running

return answer
```
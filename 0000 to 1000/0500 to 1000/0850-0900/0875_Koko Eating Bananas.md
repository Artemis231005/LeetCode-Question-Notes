# Koko Eating Bananas

## Problem

Given an array `piles`, where `piles[i]` represents the number of bananas in a pile, and an integer `h` representing the number of hours Koko has to eat all the bananas, return the **minimum integer eating speed** `k` such that Koko can finish all the bananas within `h` hours.

Koko can eat at most `k` bananas from a pile per hour.

Example:

```text
piles = [3,6,7,11]
h = 8

Output = 4
```

---

## Approach 1: Binary Search on Answer

### Idea

The possible eating speed lies between:

```text
1 and max(piles)
```

For a chosen speed `k`, calculate how many hours are required:

```text
hours += ceil(pile / k)
```

In integer arithmetic:

```text
hours += (pile + k - 1) / k
```

If Koko can finish within `h` hours:

* `k` might be the answer.
* Try a smaller speed.

If Koko needs more than `h` hours:

* `k` is too slow.
* Increase the speed.

This is **binary search on the answer** because we are searching for the minimum valid speed.

### Dry Run

```text
piles = [3,6,7,11]
h = 8

left = 1
right = 11

mid = 6

Hours:
3 → 1
6 → 1
7 → 2
11 → 2

Total = 6 ≤ 8
→ speed 6 works
→ search smaller

right = 5

mid = 3

Hours:
3 → 1
6 → 2
7 → 3
11 → 4

Total = 10 > 8
→ speed 3 is too slow
→ search larger

Eventually:
k = 4

Hours:
1 + 2 + 2 + 3 = 8

Answer = 4
```

### Algorithm

1. Set `left = 1`.
2. Set `right = max(piles)`.
3. While `left <= right`:

   * Calculate `mid`.
   * Calculate total hours required at speed `mid`.
   * If hours `<= h`:

     * `mid` is valid.
     * Store it as a possible answer.
     * Search for a smaller speed.
   * Otherwise:

     * Search for a larger speed.
4. Return the minimum valid speed.

### Complexity

* Time: `O(n log m)`
* Space: `O(1)`

where:

* `n` = number of piles
* `m` = maximum number of bananas in a pile

### Code

```cpp
class Solution {
public:
    int minEatingSpeed(vector<int>& piles, int h) {
        int left = 1;
        int right = *max_element(piles.begin(), piles.end());

        while (left <= right) {
            int mid = left + (right - left) / 2;

            long long hours = 0;

            for (int pile : piles) {
                hours += (pile + mid - 1) / mid;
            }

            if (hours <= h) {
                right = mid - 1;
            }
            else {
                left = mid + 1;
            }
        }

        return left;
    }
};
```

### Notes / Tips

* This is **Binary Search on Answer**, not ordinary binary search on an array.
* Identify:

  * Search space → possible speeds
  * Feasibility function → can Koko finish within `h` hours?
* The feasibility is monotonic:

  * If speed `k` works, every speed greater than `k` also works.
* Use `long long` for `hours` because the total can exceed `int`.
* Ceiling division:

  ```text
  ceil(a / b) = (a + b - 1) / b
  ```
* Since we want the **minimum valid speed**, when `mid` works, move left.

### Key Template

```text
left = minimum possible answer
right = maximum possible answer

while left <= right:
    mid = left + (right - left) / 2

    if mid is feasible:
        right = mid - 1
    else:
        left = mid + 1

return left
```

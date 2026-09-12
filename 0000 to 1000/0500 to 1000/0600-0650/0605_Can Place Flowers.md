# LeetCode 605 — Can Place Flowers

## Metadata

* **LeetCode:** 605
* **Problem:** Can Place Flowers
* **Difficulty:** Easy
* **Topics:** Array, Greedy
* **Pattern:** Greedy Single-Pass with Virtual Boundary Padding
* **Key Technique:** Treat both ends of the flowerbed as bordered by an implicit empty plot, then greedily plant wherever the current plot and both its neighbors are empty
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given a flowerbed array `flowerbed` where `0` means empty and `1` means planted, and an integer `n`, return `true` if `n` new flowers can be planted without any two flowers being adjacent.

---

## Approaches

1. **Optimal — Greedy Single Pass**

---

# Approach — Optimal / Greedy Single Pass

## Idea

Scan the flowerbed once. At each plot, check whether it's empty and both its neighbors are empty (treating positions before index `0` and after the last index as implicitly empty, since there's no real plot there to conflict with). Whenever this condition holds, greedily plant a flower immediately — since planting as early as possible never blocks a future valid planting (it can only ever occupy a spot that would otherwise stay empty anyway).

## Dry Run

```text
flowerbed = [1,0,0,0,1], n = 1
```

Process:

```text
i=0: flowerbed[0]=1 → occupied, skip
i=1: flowerbed[1]=0, left=flowerbed[0]=1 (not empty) → can't plant
i=2: flowerbed[2]=0, left=flowerbed[1]=0, right=flowerbed[3]=0 → plant!
     flowerbed = [1,0,1,0,1], planted=1
i=3: flowerbed[3]=0, left=flowerbed[2]=1 (not empty) → can't plant
i=4: flowerbed[4]=1 → occupied, skip
```

`planted (1) >= n (1)` → return `true`.

## Algorithm

1. Initialize `planted = 0`.
2. For each index `i` from `0` to `flowerbed.size() - 1`:

   * Check `left = (i == 0) || flowerbed[i-1] == 0`.
   * Check `right = (i == flowerbed.size()-1) || flowerbed[i+1] == 0`.
   * If `flowerbed[i] == 0` and `left` and `right` are both true:

     * Set `flowerbed[i] = 1` (plant here).
     * Increment `planted`.
3. Return `planted >= n`.

## Complexity

* **Time:** `O(n)` (where `n` here is the flowerbed's length, not the flowers to plant)

  * A single linear pass over the flowerbed.
* **Space:** `O(1)`

  * Only a running counter and boundary checks — the flowerbed is modified in place, no extra structures needed.

## Notes / Tips

* Treating the boundaries (`i == 0` and `i == flowerbed.size() - 1`) as implicitly bordered by empty plots is what avoids special-casing the first and last positions separately.
* The greedy choice is provably safe here: planting at the earliest valid opportunity can never make a *later* valid opportunity disappear, since doing so only ever "uses up" a plot that would have stayed empty otherwise, and it can only ever block the immediate next plot from being plantable (which, if skipped, wouldn't have been usable simultaneously with the current plot anyway).

## Code

```cpp
class Solution {
public:
    bool canPlaceFlowers(vector<int>& flowerbed, int n) {
        int planted = 0;
        int size = flowerbed.size();

        for (int i = 0; i < size; i++) {
            bool leftEmpty = (i == 0) || (flowerbed[i - 1] == 0);
            bool rightEmpty = (i == size - 1) || (flowerbed[i + 1] == 0);

            if (flowerbed[i] == 0 && leftEmpty && rightEmpty) {
                flowerbed[i] = 1;
                planted++;
            }
        }

        return planted >= n;
    }
};
```

---

## Key Template

```text
planted = 0

for i in 0..size-1:
    leftEmpty = (i == 0) or (flowerbed[i-1] == 0)
    rightEmpty = (i == size-1) or (flowerbed[i+1] == 0)

    if flowerbed[i] == 0 and leftEmpty and rightEmpty:
        flowerbed[i] = 1
        planted += 1

return planted >= n
```
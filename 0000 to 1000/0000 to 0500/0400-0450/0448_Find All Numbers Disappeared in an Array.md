# LeetCode 448 — Find All Numbers Disappeared in an Array

## Metadata

* **LeetCode:** 448
* **Problem:** Find All Numbers Disappeared in an Array
* **Difficulty:** Easy
* **Topics:** Array, Hash Table
* **Pattern:** Index-as-Hash-Key (In-Place Marking)
* **Key Technique:** Since values range from `1` to `n`, use each value's corresponding index as a marker by negating it — any index still positive afterward means that number never appeared
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space (excluding the output)

---

## Problem Statement

Given an array `nums` of `n` integers where `nums[i]` is in the range `[1, n]`, return all the integers in `[1, n]` that do not appear in `nums`.

---

## Approaches

1. **Brute Force — Hash Set of Seen Values**
2. **Optimal — In-Place Negation Marking**

---

# Approach 1 — Brute Force / Hash Set of Seen Values

## Idea

Insert every value from `nums` into a hash set. Then check every number from `1` to `n` — any number not found in the set is missing.

## Dry Run

```text
nums = [4, 3, 2, 7, 8, 2, 3, 1]
```

Build set:

```text
seen = {4, 3, 2, 7, 8, 1}
```

Check `1` to `8`:

```text
1 → in set
2 → in set
3 → in set
4 → in set
5 → not in set → missing
6 → not in set → missing
7 → in set
8 → in set
```

Result: `[5, 6]`.

## Algorithm

1. Insert every value of `nums` into a hash set `seen`.
2. For each number `i` from `1` to `n`:

   * If `i` is not in `seen`, add it to the result.
3. Return the result.

## Complexity

* **Time:** `O(n)`

  * One pass to build the set, one pass to check all `n` numbers.
* **Space:** `O(n)`

  * For the hash set storing up to `n` distinct values.

## Notes / Tips

* Correct and simple, but the extra hash set is unnecessary. Since values are guaranteed to fall within `[1, n]`, the array itself can be used as the "set," using each value's position as a marker instead of a separate structure.

## Code

```cpp
class Solution {
public:
    vector<int> findDisappearedNumbers(vector<int>& nums) {
        unordered_set<int> seen(nums.begin(), nums.end());
        vector<int> result;

        for (int i = 1; i <= nums.size(); i++) {
            if (seen.find(i) == seen.end()) {
                result.push_back(i);
            }
        }

        return result;
    }
};
```

---

# Approach 2 — Optimal / In-Place Negation Marking

## Idea

Since every value is in `[1, n]`, each value `v` has a natural corresponding index `v - 1`. For every value seen in the array, negate the number stored at that corresponding index (if not already negative) — this "marks" that index as visited without needing any extra memory. After processing the whole array, any index still holding a **positive** value means its corresponding number (`index + 1`) was never seen.

## Dry Run

```text
nums = [4, 3, 2, 7, 8, 2, 3, 1]
```

Process each value, negating at its target index:

```text
nums[0]=4 → target index 3 → nums[3] = -7
nums[1]=3 → target index 2 → nums[2] = -2
nums[2]=-2 → abs=2 → target index 1 → nums[1] = -3
nums[3]=-7 → abs=7 → target index 6 → nums[6] = -3
nums[4]=8 → target index 7 → nums[7] = -1
nums[5]=2 → abs=2 → target index 1 → nums[1] already negative, skip
nums[6]=-3 → abs=3 → target index 2 → nums[2] already negative, skip
nums[7]=-1 → abs=1 → target index 0 → nums[0] = -4
```

Final array:

```text
[-4, -3, -2, -7, 8, 2, -3, -1]
```

Indices `4` and `5` (0-indexed) still hold positive values → missing numbers are `4+1=5` and `5+1=6`.

Result: `[5, 6]`, matching the brute-force output.

## Algorithm

1. For each element `nums[i]`:

   * Compute `target = abs(nums[i]) - 1`.
   * If `nums[target]` is still positive, negate it (`nums[target] = -nums[target]`).
2. For each index `i` from `0` to `n-1`:

   * If `nums[i]` is still positive, `i + 1` is a missing number — add it to the result.
3. (Optional) Restore the original array by taking the absolute value of every element, if the input must not be mutated.
4. Return the result.

## Complexity

* **Time:** `O(n)`

  * One pass to mark, one pass to collect missing numbers.
* **Space:** `O(1)` auxiliary (beyond the required output)

  * The array itself is reused as the marking structure — no hash set or extra array needed.

## Notes / Tips

* Using `abs(nums[i])` when computing the target index is essential. Once a value has been negated by an earlier marking, its original magnitude is still needed to find *its own* correct target index.
* Checking `if nums[target] is still positive` before negating avoids double-negating a value back to positive when duplicate values map to the same target index (as `2` does twice in the dry run above).
* This is a classic "index as hash key" trick — useful whenever an array's values are guaranteed to fall within a range tied to the array's own size, letting the array double as its own visited-marker structure instead of allocating a separate one.

## Code

```cpp
class Solution {
public:
    vector<int> findDisappearedNumbers(vector<int>& nums) {
        int n = nums.size();

        for (int i = 0; i < n; i++) {
            int target = abs(nums[i]) - 1;
            if (nums[target] > 0) {
                nums[target] = -nums[target];
            }
        }

        vector<int> result;
        for (int i = 0; i < n; i++) {
            if (nums[i] > 0) {
                result.push_back(i + 1);
            }
        }

        return result;
    }
};
```

---

## Key Template

```text
for i in 0..n-1:
    target = abs(nums[i]) - 1
    if nums[target] > 0:
        nums[target] = -nums[target]

result = []
for i in 0..n-1:
    if nums[i] > 0:
        result.append(i + 1)

return result
```
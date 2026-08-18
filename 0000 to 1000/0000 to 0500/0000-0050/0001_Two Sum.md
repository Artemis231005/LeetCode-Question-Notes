# LeetCode 1 — Two Sum

## Metadata

* **LeetCode:** 1
* **Problem:** Two Sum
* **Difficulty:** Easy
* **Topics:** Array, Hash Table
* **Pattern:** Complement Lookup
* **Key Pattern:** For every element `x`, look for its complement `target - x`
* **Key Template:** Hash Map — `value → index`
* **Key Technique:** Store previously seen values and check whether the required complement already exists
* **Optimal Complexity:** `O(n)` time, `O(n)` space

---

# Approaches

1. **Brute Force — Check Every Pair**
2. **Better — Sort + Two Pointers**
3. **Optimal — Hash Map**

---

# Approach 1 — Brute Force / Check Every Pair

## Idea

Try every possible pair of elements and check whether their sum equals `target`.

For every `i`, compare `nums[i]` with every element after it.

## Dry Run

```text
nums = [2, 7, 11, 15]
target = 9
```

Check pairs:

```text
2 + 7 = 9 ✓
```

Therefore:

```text
[0, 1]
```

## Algorithm

1. Choose an element using index `i`.
2. Choose every element after it using index `j`.
3. Check whether:

   ```text
   nums[i] + nums[j] == target
   ```
4. If yes, return `{i, j}`.
5. Continue until a valid pair is found.

## Complexity

* **Time:** `O(n²)`
* **Space:** `O(1)`

## Notes / Tips

* This is the most direct approach.
* No additional data structure is required.
* It becomes inefficient for large arrays.

## Code

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        for (int i = 0; i < nums.size(); i++) {
            for (int j = i + 1; j < nums.size(); j++) {
                if (nums[i] + nums[j] == target) {
                    return {i, j};
                }
            }
        }

        return {};
    }
};
```

---

# Approach 2 — Better / Sorting + Two Pointers

## Idea

Two pointers can find the required pair efficiently **if the array is sorted**.

Sort the values and maintain:

* `left` → smallest element
* `right` → largest element

Then:

* If `nums[left] + nums[right] < target`, increase `left`.
* If `nums[left] + nums[right] > target`, decrease `right`.
* If equal, the pair is found.

### Problem

The question asks for the **original indices**.

Sorting changes the positions, so we need to store each value together with its original index.

For example:

```text
Original:
[2, 7, 11, 15]

After storing indices:
[(2,0), (7,1), (11,2), (15,3)]
```

Then sort by value.

## Dry Run

```text
nums = [3, 2, 4]
target = 6
```

Store:

```text
[(3,0), (2,1), (4,2)]
```

Sort:

```text
[(2,1), (3,0), (4,2)]
```

Pointers:

```text
2 + 4 = 6 ✓
```

Return original indices:

```text
[1, 2]
```

## Algorithm

1. Store each element along with its original index.
2. Sort the pairs by value.
3. Initialize:

   ```text
   left = 0
   right = n - 1
   ```
4. Calculate:

   ```text
   sum = value[left] + value[right]
   ```
5. If `sum == target`, return their original indices.
6. If `sum < target`, increment `left`.
7. If `sum > target`, decrement `right`.
8. Continue until the pair is found.

## Complexity

* **Time:** `O(n log n)`
* **Space:** `O(n)`

## Notes / Tips

* Two pointers work naturally after sorting.
* Original indices must be preserved.
* This is better than brute force but still slower than the hash-map solution.
* Sorting is unnecessary for the standard Two Sum problem.

## Code

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        vector<pair<int, int>> arr;

        for (int i = 0; i < nums.size(); i++) {
            arr.push_back({nums[i], i});
        }

        sort(arr.begin(), arr.end());

        int left = 0;
        int right = arr.size() - 1;

        while (left < right) {
            int sum = arr[left].first + arr[right].first;

            if (sum == target) {
                return {arr[left].second, arr[right].second};
            }

            if (sum < target) {
                left++;
            }
            else {
                right--;
            }
        }

        return {};
    }
};
```

---

# Approach 3 — Optimal / Hash Map

## Idea

Instead of searching for a pair explicitly, use the **complement**.

For every number:

```text
complement = target - nums[i]
```

If the complement has already appeared, we have found the answer.

Otherwise, store the current number and its index in a hash map.

### Example

```text
nums = [2, 7, 11, 15]
target = 9
```

Start:

```text
2
complement = 9 - 2 = 7
```

`7` is not in the map.

Store:

```text
2 → 0
```

Next:

```text
7
complement = 9 - 7 = 2
```

`2` is already in the map.

Therefore:

```text
[0, 1]
```

## Algorithm

1. Create a hash map:

   ```text
   value → index
   ```
2. Traverse the array from left to right.
3. For the current value `nums[i]`, calculate:

   ```text
   complement = target - nums[i]
   ```
4. Check whether `complement` exists in the map.
5. If it exists, return:

   ```text
   {map[complement], i}
   ```
6. Otherwise, store:

   ```text
   nums[i] → i
   ```
7. Continue until the pair is found.

## Complexity

* **Time:** `O(n)` average
* **Space:** `O(n)`

## Notes / Tips

* The key idea is **complement lookup**.
* Store elements **after checking the complement**.
* This prevents using the same element twice.
* `unordered_map` provides average `O(1)` lookup.
* The problem guarantees exactly one valid answer, so returning the pair is sufficient.

## Code

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> mp;

        for (int i = 0; i < nums.size(); i++) {
            int complement = target - nums[i];

            if (mp.find(complement) != mp.end()) {
                return {mp[complement], i};
            }

            mp[nums[i]] = i;
        }

        return {};
    }
};
```

---

# Approach Comparison

| Approach               |           Time |  Space | Status      |
| ---------------------- | -------------: | -----: | ----------- |
| Brute Force            |        `O(n²)` | `O(1)` | Brute       |
| Sorting + Two Pointers |   `O(n log n)` | `O(n)` | Better      |
| Hash Map               | `O(n)` average | `O(n)` | **Optimal** |

---

# Key Template

### Hash Map — Complement Lookup

```cpp
unordered_map<int, int> mp;

for (int i = 0; i < nums.size(); i++) {
    int complement = target - nums[i];

    if (mp.find(complement) != mp.end()) {
        return {mp[complement], i};
    }

    mp[nums[i]] = i;
}
```

### Pattern Recognition

When a problem asks:

```text
Find two elements such that:
a + b = target
```

Think:

```text
b = target - a
```

Then ask:

> **Have I already seen the complement?**

This transforms a repeated search into an `O(1)` average hash-map lookup.

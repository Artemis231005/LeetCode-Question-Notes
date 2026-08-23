# LeetCode 3731 — Find Missing Elements

## Metadata

* **LeetCode:** 3731
* **Problem:** Find Missing Elements
* **Difficulty:** Easy
* **Topics:** Array, Hash Set, Sorting
* **Pattern:** Range Traversal / Missing Elements
* **Key Technique:** Hash Set for O(1) average existence checks
* **Optimal Complexity:** O(n + (max - min)) Time, O(n) Space

---

## Approach 1: Hash Set

### Idea

We are given an array and need to find the elements missing from the range between the minimum and maximum values.

* Store all elements of the array in a **set**.
* Iterate from `min(nums)` to `max(nums)`.
* If a number is not present in the set, add it to the answer.

This directly checks every number that should exist in the range.

### Dry Run

```text
nums = [1, 4, 2, 5]

min = 1
max = 5

Range: 1 2 3 4 5

1 → present
2 → present
3 → missing
4 → present
5 → present

Answer = [3]
```

### Algorithm

1. Find the minimum and maximum element.
2. Insert every array element into a hash set.
3. Iterate from `minimum` to `maximum`.
4. If the current number is not in the set, add it to the answer.
5. Return the answer.

### Complexity

* **Time:** `O(n + (max - min))`
* **Space:** `O(n)`

### Code

```cpp
class Solution {
public:
    vector<int> findMissingElements(vector<int>& nums) {
        unordered_set<int> st(nums.begin(), nums.end());

        int mn = *min_element(nums.begin(), nums.end());
        int mx = *max_element(nums.begin(), nums.end());

        vector<int> ans;

        for (int i = mn; i <= mx; i++) {
            if (st.find(i) == st.end()) {
                ans.push_back(i);
            }
        }

        return ans;
    }
};
```

---

## Approach 2: Sorting

### Idea

After sorting, consecutive elements should differ by exactly `1` if no elements are missing.

Whenever:

```text
nums[i] > nums[i - 1] + 1
```

there are missing numbers between them.

For every gap, add all the numbers between the two elements.

### Dry Run

```text
nums = [1, 4, 2, 5]

After sorting:
[1, 2, 4, 5]

1 → 2 : no gap
2 → 4 : 3 is missing
4 → 5 : no gap

Answer = [3]
```

### Algorithm

1. Sort the array.
2. Traverse adjacent elements.
3. For every pair `nums[i - 1]` and `nums[i]`:

   * Start from `nums[i - 1] + 1`.
   * Add every value smaller than `nums[i]`.
4. Return the answer.

### Complexity

* **Time:** `O(n log n + k)`
* **Space:** `O(1)` auxiliary space, excluding the answer.
* `k` = number of missing elements.

### Code

```cpp
class Solution {
public:
    vector<int> findMissingElements(vector<int>& nums) {
        sort(nums.begin(), nums.end());

        vector<int> ans;

        for (int i = 1; i < nums.size(); i++) {
            for (int x = nums[i - 1] + 1; x < nums[i]; x++) {
                ans.push_back(x);
            }
        }

        return ans;
    }
};
```

---

## Notes / Tips

* The required range is **from the minimum element to the maximum element**, not from `1` to `n`.
* A **set** gives the most direct solution: check whether every number in the range exists.
* With sorting, missing elements appear as **gaps between consecutive values**.
* Since the problem asks for missing values only inside the existing range, elements smaller than the minimum or larger than the maximum are irrelevant.
* If duplicates are possible, the **set approach naturally handles them**; the sorting approach also works because duplicates simply create no gap.

## Key Template

### Hash Set — Find Missing Values in a Range

```cpp
unordered_set<int> st(nums.begin(), nums.end());

int mn = *min_element(nums.begin(), nums.end());
int mx = *max_element(nums.begin(), nums.end());

vector<int> ans;

for (int i = mn; i <= mx; i++) {
    if (st.find(i) == st.end()) {
        ans.push_back(i);
    }
}
```

### Sorting — Find Gaps

```cpp
sort(nums.begin(), nums.end());

vector<int> ans;

for (int i = 1; i < nums.size(); i++) {
    for (int x = nums[i - 1] + 1; x < nums[i]; x++) {
        ans.push_back(x);
    }
}
```

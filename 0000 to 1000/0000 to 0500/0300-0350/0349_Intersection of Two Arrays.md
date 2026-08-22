# 349. Intersection of Two Arrays

## Metadata

* **Topic:** Array, Hash Set
* **Difficulty:** Easy
* **Key Pattern:** Set for Uniqueness
* **Key Template:** Store one array in a set, then check elements of the other array
* **Goal:** Return the unique elements that appear in both arrays.

---

## Approach 1: Hash Set

### Idea

We need the **distinct intersection** of two arrays.

Use a set:

1. Insert all elements of `nums1` into a set.
2. Traverse `nums2`.
3. If an element exists in the set, add it to the answer.
4. Remove it from the set so that it cannot be added again.

This automatically ensures that every answer appears only once.

### Dry Run

```text
nums1 = [1, 2, 2, 1]
nums2 = [2, 2]
```

Put `nums1` into a set:

```text
set = {1, 2}
```

Traverse `nums2`:

```text
2 → found → add 2 → erase 2
2 → not found
```

Final answer:

```text
[2]
```

### Algorithm

1. Create an unordered set.
2. Insert every element of `nums1`.
3. Create an answer array.
4. Traverse `nums2`.
5. If the current element exists in the set:

   * Add it to the answer.
   * Erase it from the set.
6. Return the answer.

### Complexity

* **Time:** `O(n + m)` average
* **Space:** `O(n)`
* Where `n = nums1.size()` and `m = nums2.size()`.

### Notes / Tips

* The word **unique** is the key clue.
* Use a `set`/`unordered_set` instead of a frequency map because we only care whether an element exists.
* Erasing after finding an element prevents duplicates in the answer.
* Do not confuse this with **Intersection of Two Arrays II**, where duplicate occurrences matter.
* `unordered_set` gives average `O(1)` lookup and insertion.

### Code

```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        unordered_set<int> st;

        for (int num : nums1) {
            st.insert(num);
        }

        vector<int> ans;

        for (int num : nums2) {
            if (st.find(num) != st.end()) {
                ans.push_back(num);
                st.erase(num);
            }
        }

        return ans;
    }
};
```

---

## Approach 2: Sorting + Two Pointers

### Idea

Sort both arrays and use two pointers.

If:

```text
nums1[i] < nums2[j]
```

move `i`.

If:

```text
nums1[i] > nums2[j]
```

move `j`.

If they are equal, we found a common element.

To maintain uniqueness, only add the element if it is different from the previously added answer.

### Dry Run

```text
nums1 = [1, 2, 2, 1]
nums2 = [2, 2]
```

After sorting:

```text
nums1 = [1, 1, 2, 2]
nums2 = [2, 2]
```

Pointers:

```text
1 < 2 → move i
1 < 2 → move i
2 == 2 → add 2
```

Continue moving past duplicates.

Answer:

```text
[2]
```

### Algorithm

1. Sort both arrays.
2. Initialize `i = 0`, `j = 0`.
3. While both pointers are inside their arrays:

   * If `nums1[i] < nums2[j]`, increment `i`.
   * If `nums1[i] > nums2[j]`, increment `j`.
   * Otherwise:

     * Add the value if it is not already the last answer.
     * Increment both pointers.
4. Return the answer.

### Complexity

* **Time:** `O(n log n + m log m)`
* **Space:** `O(1)` extra space, excluding the output array.

### Notes / Tips

* Two pointers work naturally after sorting.
* This approach avoids hash-table memory.
* Always handle duplicates when the problem asks for a **unique** intersection.
* Compared to the hash-set approach, this is slower asymptotically but has deterministic complexity.

### Code

```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        sort(nums1.begin(), nums1.end());
        sort(nums2.begin(), nums2.end());

        vector<int> ans;

        int i = 0;
        int j = 0;

        while (i < nums1.size() && j < nums2.size()) {
            if (nums1[i] < nums2[j]) {
                i++;
            }
            else if (nums1[i] > nums2[j]) {
                j++;
            }
            else {
                if (ans.empty() || ans.back() != nums1[i]) {
                    ans.push_back(nums1[i]);
                }

                i++;
                j++;
            }
        }

        return ans;
    }
};
```

---

## Key Template

```cpp
unordered_set<int> st;

for (int x : nums1) {
    st.insert(x);
}

vector<int> ans;

for (int x : nums2) {
    if (st.find(x) != st.end()) {
        ans.push_back(x);
        st.erase(x);
    }
}
```

### Pattern to Remember

```text
Unique Intersection
        ↓
Need existence only?
        ↓
Hash Set
        ↓
Store first array
        ↓
Check second array
        ↓
Found → add + erase
```

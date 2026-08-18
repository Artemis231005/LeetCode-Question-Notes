# LeetCode 26 — Remove Duplicates from Sorted Array

## Metadata

* **LeetCode:** 26
* **Problem:** Remove Duplicates from Sorted Array
* **Difficulty:** Easy
* **Topics:** Array, Two Pointers
* **Pattern:** Two Pointers — Read/Write
* **Key Pattern:** Maintain a write pointer for the next unique element while scanning with a read pointer
* **Key Template:** Slow/Fast Pointer — `write` for placement, `read` for traversal
* **Key Technique:** Since the array is sorted, a new unique element is found whenever `nums[read] != nums[write - 1]`
* **Optimal Complexity:** `O(n)` time, `O(1)` space

---

# Approaches

1. **Brute Force — Use an Extra Array**
2. **Better — Use a Set**
3. **Optimal — Two Pointers / In-Place**

---

# Approach 1 — Brute Force / Extra Array

## Idea

Create a separate array containing only unique elements.

Because the input array is sorted, duplicates are adjacent, so we can simply compare each element with the last element added to the result.

After collecting all unique elements, copy them back into the original array.

## Dry Run

```text
nums = [1, 1, 2, 2, 3]
```

Start:

```text
unique = []
```

Process:

```text
1 → [1]
1 → duplicate → skip
2 → [1, 2]
2 → duplicate → skip
3 → [1, 2, 3]
```

Copy back:

```text
nums = [1, 2, 3, 2, 3]
```

Only the first `k = 3` elements matter.

## Algorithm

1. Create an empty `unique` array.
2. Traverse `nums`.
3. If `unique` is empty or the current value differs from the last unique value:

   * Add it to `unique`.
4. Copy `unique` back into `nums`.
5. Return the number of unique elements.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(n)`

## Notes / Tips

* The array is already sorted, so a hash set is unnecessary.
* This violates the problem's `O(1)` extra-space requirement.

## Code

```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        vector<int> unique;

        for (int x : nums) {
            if (unique.empty() || unique.back() != x) {
                unique.push_back(x);
            }
        }

        for (int i = 0; i < unique.size(); i++) {
            nums[i] = unique[i];
        }

        return unique.size();
    }
};
```

---

# Approach 2 — Better / Set

## Idea

Use a set to automatically store only unique values.

Since `set` stores elements in sorted order, the resulting unique elements remain sorted.

Then overwrite the beginning of `nums` with the values from the set.

## Dry Run

```text
nums = [1, 1, 2, 2, 3]
```

Insert:

```text
1 → {1}
1 → {1}
2 → {1, 2}
2 → {1, 2}
3 → {1, 2, 3}
```

Copy back:

```text
nums = [1, 2, 3, ...]
```

Return:

```text
3
```

## Algorithm

1. Create a set.
2. Insert every element of `nums`.
3. Traverse the set.
4. Copy each unique element into `nums`.
5. Return the number of unique elements.

## Complexity

Using `set`:

* **Time:** `O(n log n)`
* **Space:** `O(n)`

## Notes / Tips

* This works, but the sorted input makes the set unnecessary.
* A hash set would provide average `O(n)` time, but would still use `O(n)` space.
* The optimal solution exploits the fact that duplicates are adjacent.

## Code

```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        set<int> st;

        for (int x : nums) {
            st.insert(x);
        }

        int index = 0;

        for (int x : st) {
            nums[index] = x;
            index++;
        }

        return index;
    }
};
```

---

# Approach 3 — Optimal / Two Pointers

## Idea

Because the array is **sorted**, all duplicates are next to each other.

We can modify the array in-place using two pointers:

* `read` → scans every element.
* `write` → points to the position where the next unique element should be placed.

The first element is always unique.

For every subsequent element:

```text
if nums[read] != nums[write - 1]
```

then it is a new unique element.

Place it at `nums[write]` and increment `write`.

## Dry Run

```text
nums = [1, 1, 2, 2, 3]
```

Initially:

```text
write = 1
```

The first `1` is already in the correct position.

### `read = 1`

```text
nums[read] = 1
nums[write - 1] = 1
```

Duplicate → skip.

### `read = 2`

```text
nums[read] = 2
nums[write - 1] = 1
```

New unique element.

Place:

```text
nums[write] = 2
```

Array:

```text
[1, 2, 2, 2, 3]
```

Then:

```text
write = 2
```

### `read = 3`

```text
2 == nums[write - 1]
```

Duplicate → skip.

### `read = 4`

```text
3 != 2
```

Place `3`:

```text
[1, 2, 3, 2, 3]
```

Final:

```text
write = 3
```

Therefore:

```text
k = 3
```

The first three elements are:

```text
[1, 2, 3]
```

## Algorithm

1. If `nums` is empty, return `0`.
2. Set:

   ```text
   write = 1
   ```
3. Start `read` from index `1`.
4. For every `nums[read]`:

   * If it differs from `nums[write - 1]`:

     1. Store it at `nums[write]`.
     2. Increment `write`.
5. Return `write`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

## Notes / Tips

* The sorted property is the key observation.
* We never need to physically delete elements.
* Only the first `k` elements matter after the operation.
* `write` represents the number of unique elements found so far.
* This is a classic **Slow/Fast Pointer** problem.
* Similar pattern appears in:

  * Remove Element
  * Move Zeroes
  * Remove Duplicates from Sorted Array II
  * Partitioning problems

## Code

```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        if (nums.empty()) {
            return 0;
        }

        int write = 1;

        for (int read = 1; read < nums.size(); read++) {
            if (nums[read] != nums[write - 1]) {
                nums[write] = nums[read];
                write++;
            }
        }

        return write;
    }
};
```

---

# Approach Comparison

| Approach     |         Time |  Space | Status      |
| ------------ | -----------: | -----: | ----------- |
| Extra Array  |       `O(n)` | `O(n)` | Brute       |
| Set          | `O(n log n)` | `O(n)` | Better      |
| Two Pointers |       `O(n)` | `O(1)` | **Optimal** |

---

# Key Template

### Slow/Fast Pointer — Read/Write

```cpp
int write = 0;

for (int read = 0; read < nums.size(); read++) {
    if (condition_for_valid_element) {
        nums[write] = nums[read];
        write++;
    }
}

return write;
```

### For Sorted Array — Remove Duplicates

```cpp
int write = 1;

for (int read = 1; read < nums.size(); read++) {
    if (nums[read] != nums[write - 1]) {
        nums[write] = nums[read];
        write++;
    }
}

return write;
```

## Pattern Recognition

When you see:

```text
Sorted array
+
Remove/filter elements
+
Modify in-place
+
O(1) extra space
```

Think:

```text
Two Pointers
    ↓
read  → scan elements
write → place valid elements
```

The key observation is:

> **In a sorted array, duplicates are adjacent, so every change in value identifies a new unique element.**

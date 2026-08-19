# LeetCode 35 — Search Insert Position

## Metadata

* **LeetCode:** 35
* **Problem:** Search Insert Position
* **Difficulty:** Easy
* **Topics:** Array, Binary Search
* **Pattern:** Lower Bound
* **Key Technique:** Binary Search
* **Key Pattern:** First Position `>= target`
* **Optimal Complexity:** `O(log n)` time, `O(1)` space

---

## Problem

Given a sorted array of **distinct integers** and a `target`, return the index if the target is found.

If the target is not present, return the index where it would be inserted so that the array remains sorted.

### Example 1

```text id="x2r6qa"
nums = [1, 3, 5, 6]
target = 5

Output = 2
```

### Example 2

```text id="s5n7fc"
nums = [1, 3, 5, 6]
target = 2

Output = 1
```

Because inserting `2` at index `1` gives:

```text id="9q6m1d"
[1, 2, 3, 5, 6]
```

### Example 3

```text id="n8f3yd"
nums = [1, 3, 5, 6]
target = 7

Output = 4
```

---

# Approach 1 — Linear Search

## Idea

Traverse the array from left to right.

The first element that is **greater than or equal to** the target is exactly where the target should be inserted.

If no such element exists, the target belongs at the end.

## Dry Run

```text id="c7m2vz"
nums = [1, 3, 5, 6]
target = 2
```

Check:

```text id="8w5zqk"
1 < 2 → continue
3 >= 2 → stop
```

Therefore:

```text id="a0k9jp"
answer = 1
```

For:

```text id="0w4kbf"
target = 7
```

all elements are smaller:

```text id="4x9qke"
1 < 7
3 < 7
5 < 7
6 < 7
```

So insert at:

```text id="c1y6ga"
index = 4
```

## Algorithm

1. Traverse the array from left to right.
2. Find the first index `i` such that:

   ```cpp
   nums[i] >= target
   ```
3. Return `i`.
4. If no such index exists, return `nums.size()`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

## Notes / Tips

* This approach directly reveals the key idea of the problem.
* Since the array is sorted, we can find this position using binary search.

## Code

```cpp id="m6s1qo"
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] >= target) {
                return i;
            }
        }

        return nums.size();
    }
};
```

---

# Approach 2 — Binary Search

## Idea

The answer is the **first position where `nums[i] >= target`**.

This is exactly the **lower bound** of `target`.

Use binary search:

* If `nums[mid] < target`, the answer must be to the right.
* Otherwise, `nums[mid] >= target`, so `mid` could be the answer.
  Move left to find an earlier valid position.

At the end, `left` is the insertion position.

## Dry Run

Consider:

```text id="k0c5x8"
nums = [1, 3, 5, 6]
target = 2
```

Initial:

```text id="9h7r4v"
left = 0
right = 3
```

### Step 1

```text id="7v5m3n"
mid = 1
nums[mid] = 3
```

Since:

```text id="kq3z7p"
3 >= 2
```

`mid = 1` could be the answer.

Move left:

```text id="a2r7w8"
right = 0
```

### Step 2

```text id="5z0b6q"
mid = 0
nums[mid] = 1
```

Since:

```text id="x9w1da"
1 < 2
```

the answer must be to the right:

```text id="p5g8kl"
left = 1
```

Now:

```text id="r7w2mq"
left > right
```

Therefore:

```text id="j1v5pc"
answer = left = 1
```

### Another Example

```text id="h0s4jx"
nums = [1, 3, 5, 6]
target = 5
```

Binary search eventually finds:

```text id="8cq1dn"
nums[2] = 5
```

Since we want the first position where:

```text id="l4t9qz"
nums[i] >= 5
```

we continue toward the left.

The answer remains:

```text id="v2r8ks"
2
```

## Algorithm

1. Initialize:

   ```cpp
   left = 0;
   right = nums.size();
   ```
2. While:

   ```cpp
   left < right
   ```
3. Calculate:

   ```cpp
   mid = left + (right - left) / 2;
   ```
4. If:

   ```cpp
   nums[mid] < target
   ```

   move right:

   ```cpp
   left = mid + 1;
   ```
5. Otherwise:

   ```cpp
   right = mid;
   ```
6. Return `left`.

## Complexity

* **Time:** `O(log n)`
* **Space:** `O(1)`

## Notes / Tips

### The Most Important Observation

This is not really a "search for target" problem.

It is a **boundary search** problem:

```text
Find the first index where:

nums[i] >= target
```

That is exactly **Lower Bound**.

### Why `right = nums.size()`?

We use a half-open search range:

```text
[left, right)
```

This allows the answer to be:

```text
nums.size()
```

which happens when the target is greater than every element.

Example:

```text id="j4w2qt"
nums = [1, 3, 5, 6]
target = 7

answer = 4
```

### Key Pattern

```text
nums[mid] < target
        ↓
  answer is RIGHT
        ↓
left = mid + 1

nums[mid] >= target
        ↓
mid could be answer
        ↓
right = mid
```

## Code

```cpp id="x7f4ma"
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size();

        while (left < right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] < target) {
                left = mid + 1;
            }
            else {
                right = mid;
            }
        }

        return left;
    }
};
```

---

# Approach 3 — Standard Binary Search with Exact Match

## Idea

We can also write this using the more familiar binary-search structure with:

```cpp
left <= right
```

If the target is found, return immediately.

If the target is not found, eventually `left` will point to the correct insertion position.

## Dry Run

```text id="o1h5pc"
nums = [1, 3, 5, 6]
target = 4
```

Start:

```text id="j9k2lm"
left = 0
right = 3
```

### Step 1

```text id="x4v8qa"
mid = 1
nums[mid] = 3
```

Since:

```text id="3 < 4"
```

move right:

```text id="w6z1pf"
left = 2
```

### Step 2

```text id="7g2kcd"
mid = 2
nums[mid] = 5
```

Since:

```text id="5 > 4"
```

move left:

```text id="z3p8vn"
right = 1
```

Now:

```text id="q6r1ab"
left = 2
right = 1
```

Search ends.

`left = 2` is the insertion position.

```text id="m9k5xy"
[1, 3, 4, 5, 6]
       ↑
      index 2
```

## Algorithm

1. Set `left = 0` and `right = n - 1`.
2. While `left <= right`:

   * Calculate `mid`.
   * If `nums[mid] == target`, return `mid`.
   * If `nums[mid] < target`, move `left` right.
   * Otherwise, move `right` left.
3. When the loop ends, return `left`.

## Complexity

* **Time:** `O(log n)`
* **Space:** `O(1)`

## Notes / Tips

* This approach is perfectly valid.
* However, the **lower-bound approach is more reusable** because it directly captures the actual requirement.
* The final value of `left` after binary search is the insertion position.

## Code

```cpp id="q8n3ws"
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] == target) {
                return mid;
            }
            else if (nums[mid] < target) {
                left = mid + 1;
            }
            else {
                right = mid - 1;
            }
        }

        return left;
    }
};
```

---

# Comparison of Approaches

| Approach               |       Time |  Space | Main Idea                              |
| ---------------------- | ---------: | -----: | -------------------------------------- |
| Linear Search          |     `O(n)` | `O(1)` | Find first `>= target`                 |
| Lower Bound            | `O(log n)` | `O(1)` | Find first position `>= target`        |
| Standard Binary Search | `O(log n)` | `O(1)` | Search target, return `left` if absent |

---

# Key Takeaway

The most important realization is:

> **Search Insert Position = Lower Bound**

You are looking for:

```text
first index where nums[i] >= target
```

### Key Template

```cpp
int left = 0;
int right = nums.size();

while (left < right) {
    int mid = left + (right - left) / 2;

    if (nums[mid] < target) {
        left = mid + 1;
    }
    else {
        right = mid;
    }
}

return left;
```

### Mental Model

```text
Sorted Array
     ↓
Need insertion position
     ↓
Find first element >= target
     ↓
Lower Bound
     ↓
Binary Search
     ↓
Return left
```

**Key Pattern:** **Lower Bound / Boundary Binary Search**

**Remember:**

```text
nums[mid] < target
    → go RIGHT

nums[mid] >= target
    → mid can be answer
    → go LEFT
```

This exact lower-bound pattern is reusable in problems involving **first occurrence, insertion position, frequency ranges, and boundary finding in sorted arrays**.

# 287. Find the Duplicate Number

## Metadata

* **Topic:** Array, Two Pointer, Binary Search
* **Difficulty:** Medium
* **Key Pattern:** Floyd's Cycle Detection
* **Key Template:** Treat array values as pointers and find the cycle entrance
* **Goal:** Find the only duplicate number in an array containing `n + 1` integers where each integer is in `[1, n]`.

---

## Approach 1: Floyd's Cycle Detection — Fast & Slow Pointers

### Idea

Treat each array index as a node and `nums[i]` as the next pointer.

Because every value is between `1` and `n`, every pointer stays within a valid index. Since there are `n + 1` elements but only `n` possible values, a duplicate must exist, which creates a **cycle**.

Use:

* `slow` → moves one step
* `fast` → moves two steps

First, find a meeting point inside the cycle.

Then:

* Reset `slow` to the starting point.
* Move both `slow` and `fast` one step at a time.
* Their next meeting point is the duplicate number.

### Dry Run

`nums = [1,3,4,2,2]`

Pointers:

```text
0 → 1 → 3 → 2 → 4
        ↑       ↓
        └───────┘
```

The cycle is:

```text
2 → 4 → 2
```

First phase:

```text
slow = nums[slow]
fast = nums[nums[fast]]
```

They eventually meet inside the cycle.

Second phase:

```text
slow = 0
fast = meeting point
```

Move both one step at a time.

They meet at `2`.

Therefore:

```text
Answer = 2
```

### Algorithm

1. Initialize `slow = nums[0]` and `fast = nums[0]`.
2. Move `slow` one step and `fast` two steps until they meet.
3. Reset `slow = nums[0]`.
4. Move both pointers one step at a time.
5. When they meet again, return that value.
6. The meeting point is the duplicate number.

### Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

### Notes / Tips

* The key trick is to **view the array as a linked list**.
* `nums[i]` acts as the next node/index.
* The duplicate creates a cycle because two indices point toward the same value.
* Do not modify the array.
* Do not use a hash set because the optimal solution requires `O(1)` extra space.
* This is a classic application of **Floyd's Tortoise and Hare algorithm**.
* The first meeting point is **not necessarily the duplicate**; the second meeting gives the cycle entrance.
* Whenever you see a problem involving a duplicate and constraints like `1...n`, think about **cycle detection**.

### Code

```cpp id="f2m8kq"
class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        int slow = nums[0];
        int fast = nums[0];

        // Find meeting point inside the cycle
        do {
            slow = nums[slow];
            fast = nums[nums[fast]];
        } while (slow != fast);

        // Find entrance of the cycle
        slow = nums[0];

        while (slow != fast) {
            slow = nums[slow];
            fast = nums[fast];
        }

        return slow;
    }
};
```

---

## Key Template

```cpp id="q3v7xa"
int slow = nums[0];
int fast = nums[0];

do {
    slow = nums[slow];
    fast = nums[nums[fast]];
} while (slow != fast);

slow = nums[0];

while (slow != fast) {
    slow = nums[slow];
    fast = nums[fast];
}

return slow;
```

### Pattern to Remember

```text
Array values → next pointers
        ↓
Duplicate → cycle
        ↓
Floyd's Cycle Detection
        ↓
First meeting → inside cycle
        ↓
Reset one pointer
        ↓
Second meeting → duplicate
```

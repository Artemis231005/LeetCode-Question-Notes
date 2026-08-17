# LeetCode 496 — Next Greater Element I

## Metadata

* **LeetCode:** 496
* **Problem:** Next Greater Element I
* **Difficulty:** Easy
* **Topics:** Array, Hash Table, Stack, Monotonic Stack
* **Pattern:** Next Greater Element, Monotonic Decreasing Stack
* **Optimal Complexity:** `O(n)` Time, `O(n)` Space

---

## Problem

Given two arrays `nums1` and `nums2` where `nums1` is a subset of `nums2` and all elements are unique.
For every element in `nums1`, find its **next greater element** in `nums2`.
The next greater element of `x` is the **first element to the right of `x` that is greater than `x`**.

If no such element exists, return `-1`.

### Example

```text
nums1 = [4, 1, 2]
nums2 = [1, 3, 4, 2]

4 → -1
1 → 3
2 → -1

answer = [-1, 3, -1]
```

---

# Approach 1 — Brute Force

## Idea
For every element in `nums1`:

1. Find its position in `nums2`.
2. Starting from the next position, scan to the right.
3. Stop at the first element greater than it.
4. If no greater element is found, return `-1`.

Because `nums1` is a subset of `nums2`, every element of `nums1` will have a position in `nums2`.

## Dry Run

```text
nums1 = [4, 1, 2]
nums2 = [1, 3, 4, 2]

For 4:
position = 2
right side = [2]
2 > 4? No
→ -1

For 1:
position = 0
right side = [3, 4, 2]
3 > 1
→ 3

For 2:
position = 3
nothing to the right
→ -1

answer = [-1, 3, -1]
```

## Notes / Tips

* This directly follows the definition of the next greater element.
* The same elements may be scanned repeatedly for different queries.

## Complexity

* **Time:** `O(n * m)` where `n = nums1.length`, `m = nums2.length`
* **Space:** `O(1)` extra space apart from the result

## Code

```cpp
class Solution {
public:
    vector<int> nextGreaterElement(vector<int>& nums1, vector<int>& nums2) {
        vector<int> ans;

        for (int x : nums1) {
            int index = 0;

            while (nums2[index] != x)
                index++;

            int greater = -1;

            for (int i = index + 1; i < nums2.size(); i++) {
                if (nums2[i] > x) {
                    greater = nums2[i];
                    break;
                }
            }

            ans.push_back(greater);
        }

        return ans;
    }
};
```

---

# Approach 2 — Hash Map + Brute Force

## Idea

The expensive part of the previous approach is repeatedly finding the position of each `nums1` element in `nums2`.

Since all elements are unique, store:

```text
value → index in nums2
```

Then for every element in `nums1`, directly access its position and scan to the right.

## Dry Run

```text
nums2 = [1, 3, 4, 2]

map:
1 → 0
3 → 1
4 → 2
2 → 3

nums1 = [4, 1, 2]

4:
index = map[4] = 2
right side = [2]
no greater element
→ -1

1:
index = map[1] = 0
3 > 1
→ 3

2:
index = map[2] = 3
nothing to the right
→ -1
```

## Notes / Tips

* Hashing removes the need to search for each element's position.
* However, the right-side scan can still take `O(n)` for every element.
* The main remaining problem is repeated scanning.

## Complexity

* **Time:** `O(n * m)` in the worst case
* **Space:** `O(m)` for the hash map

## Code

```cpp
class Solution {
public:
    vector<int> nextGreaterElement(vector<int>& nums1, vector<int>& nums2) {
        unordered_map<int, int> index;

        for (int i = 0; i < nums2.size(); i++) {
            index[nums2[i]] = i;
        }

        vector<int> ans;

        for (int x : nums1) {
            int i = index[x];
            int greater = -1;

            for (int j = i + 1; j < nums2.size(); j++) {
                if (nums2[j] > x) {
                    greater = nums2[j];
                    break;
                }
            }

            ans.push_back(greater);
        }

        return ans;
    }
};
```

---

# Approach 3 — Monotonic Stack + Hash Map

## Idea

Instead of searching to the right separately for every element, process `nums2` **once** and find the next greater element for every element simultaneously.

Use a **monotonic decreasing stack**.
The stack stores elements whose next greater element has **not been found yet**.

For the current element `x`:
* While the stack is not empty and `x > stack.top()`:
  * `x` is the next greater element of `stack.top()`.
  * Store this relationship in a hash map.
  * Pop the smaller element.
* Push `x` onto the stack.

### Why does this work?

Suppose:

```text
stack = [5, 3, 2]
current = 4
```

Since:

```text
4 > 2
```

`4` is the first greater element to the right of `2`.

So:

```text
2 → 4
```

Pop `2`.

Now:

```text
4 > 3
```

Therefore:

```text
3 → 4
```

Pop `3`.

But:

```text
4 < 5
```

So `5` stays on the stack.

The stack becomes:

```text
[5, 4]
```

It remains **decreasing from bottom to top**.

## Dry Run

```text
nums2 = [1, 3, 4, 2]

Start:
stack = []
map = {}

1:
stack = [1]

3:
3 > 1
1 → 3
pop 1
stack = [3]

4:
4 > 3
3 → 4
pop 3
stack = [4]

2:
2 > 4? No
stack = [4, 2]

End:
map = {
    1 → 3,
    3 → 4
}
```

Elements remaining in the stack have no greater element to their right.

Therefore:

```text
4 → -1
2 → -1
```

For:

```text
nums1 = [4, 1, 2]
```

Look up each value:

```text
4 → -1
1 → 3
2 → -1
```

Answer:

```text
[-1, 3, -1]
```

## Why Is the Stack Monotonically Decreasing?

When a new element `x` arrives:

```text
while stack.top() < x
```

all smaller elements are removed.

Therefore, after processing `x`:

```text
stack[bottom] > stack[...] > stack[top]
```

The stack contains decreasing values.

This is why it is called a **monotonic decreasing stack**.

## Important Observation

An element is pushed onto the stack **once** and popped from the stack **at most once**.

Therefore, although there is a `while` loop, the total number of stack operations across the entire array is `O(n)`.

This is the key reason the solution is linear.

## Notes / Tips

* This is the standard **Next Greater Element** pattern.
* For a **next greater** problem, a decreasing monotonic stack is commonly used.
* The stack contains elements waiting for their next greater element.
* When `x > stack.top()`, `x` resolves the answer for the top element.
* Elements left in the stack at the end have answer `-1`.
* The hash map is used because `nums1` only asks for answers for a subset of `nums2`.
* Since values are unique, `value → next greater element` is sufficient.
* The stack stores values rather than indices because only the value is needed here.

## Complexity

Let `n = nums2.length` and `m = nums1.length`.

* **Time:** `O(n + m)`
* **Space:** `O(n)`

Each element of `nums2` is pushed once and popped at most once.

## Code

```cpp
class Solution {
public:
    vector<int> nextGreaterElement(vector<int>& nums1, vector<int>& nums2) {
        unordered_map<int, int> nextGreater;
        stack<int> st;

        for (int x : nums2) {
            while (!st.empty() && x > st.top()) {
                nextGreater[st.top()] = x;
                st.pop();
            }

            st.push(x);
        }

        vector<int> ans;

        for (int x : nums1) {
            if (nextGreater.count(x))
                ans.push_back(nextGreater[x]);
            else
                ans.push_back(-1);
        }

        return ans;
    }
};
```

---

# Key Pattern

```text
Next Greater Element
        ↓
Monotonic Decreasing Stack
        ↓
Current element is greater than stack.top()
        ↓
Current element becomes stack.top()'s answer
        ↓
Pop
```

General template:

```cpp
for (int x : nums) {
    while (!st.empty() && x > st.top()) {
        // x is the next greater element of st.top()
        st.pop();
    }

    st.push(x);
}
```

This pattern extends directly to problems such as **Next Greater Element II**, **Daily Temperatures**, and other monotonic-stack problems.

# LeetCode 128 — Longest Consecutive Sequence

## Metadata

* **LeetCode:** 128
* **Problem:** Longest Consecutive Sequence
* **Difficulty:** Medium
* **Topics:** Array, Hash Table, Union Find
* **Pattern:** Hash Set, Sequence Expansion
* **Key Technique:** Start counting only from sequence beginnings
* **Key Pattern:** Hash Set + sequence start detection
* **Key Template:** `if (x - 1 not in set) → expand forward`
* **Optimal Complexity:** `O(n)`

---

## Problem

Given an unsorted array of integers `nums`, return the length of the **longest consecutive elements sequence**.

The sequence must contain consecutive integers, but the elements do **not** need to be adjacent in the original array.

The solution must run in **`O(n)`** time.

Example:

```text
nums = [100, 4, 200, 1, 3, 2]
```

The longest consecutive sequence is:

```text
1 → 2 → 3 → 4
```

Therefore:

```text
answer = 4
```

---

## Idea

Use a **Hash Set** to store all numbers.

For every number `x`, check whether:

```text
x - 1
```

exists in the set.

### If `x - 1` exists

Then `x` is **not the beginning** of a sequence.

Skip it.

### If `x - 1` does not exist

Then `x` is the **start of a consecutive sequence**.

Start expanding:

```text
x
x + 1
x + 2
x + 3
...
```

until the next number is not present.

This ensures that every sequence is expanded only from its starting point.

### Why Does This Give `O(n)`?

We do not blindly expand from every number.

For example:

```text
1, 2, 3, 4, 5
```

Only `1` starts a sequence because:

```text
0 → not present
```

The other numbers are skipped because their previous number exists.

---

## Dry Run

Consider:

```text
nums = [100, 4, 200, 1, 3, 2]
```

Hash Set:

```text
{100, 4, 200, 1, 3, 2}
```

### `100`

Check:

```text
99 → not present
```

So `100` starts a sequence.

```text
100 → 101 not found
```

Length:

```text
1
```

### `4`

Check:

```text
3 → present
```

So `4` is not a starting point.

Skip.

### `200`

Check:

```text
199 → not present
```

Sequence:

```text
200
```

Length:

```text
1
```

### `1`

Check:

```text
0 → not present
```

So `1` starts a sequence.

Expand:

```text
1 → 2 → 3 → 4
```

Next:

```text
5 → not present
```

Length:

```text
4
```

### `3`

Check:

```text
2 → present
```

Skip.

### `2`

Check:

```text
1 → present
```

Skip.

Final answer:

```text
4
```

---

## Algorithm

1. Insert every element of `nums` into an `unordered_set`.
2. Initialize `longest = 0`.
3. For every number `x` in the set:

   * If `x - 1` exists, skip `x`.
   * Otherwise, `x` is the start of a sequence.
4. Set:

   ```text
   current = x
   length = 1
   ```
5. While `current + 1` exists:

   * Increment `current`.
   * Increment `length`.
6. Update:

   ```text
   longest = max(longest, length)
   ```
7. Return `longest`.

---

## Complexity

* **Time:** `O(n)` average

  * Building the set: `O(n)`
  * Each sequence is expanded from its starting point.
* **Space:** `O(n)`

  * Hash set stores all elements.

---

## Notes / Tips

* **Do not sort** the array because sorting takes `O(n log n)`.
* The key observation is:

  ```text
  x - 1 not present → x is a sequence start
  ```
* `unordered_set` provides average `O(1)` lookup.
* Duplicates do not cause a problem because the set automatically removes them.
* Negative numbers work naturally.
* The order of elements in the input does not matter.

### Common Mistake

Incorrect approach:

```cpp
for every x:
    while (x + 1 exists):
        expand
```

This can repeatedly scan the same sequence.

For:

```text
[1, 2, 3, 4, 5]
```

you would expand from `1`, then again from `2`, then from `3`, etc.

Instead, expand **only when `x - 1` is absent**.

### Key Observation

```text
Sequence:
1 → 2 → 3 → 4

Starting point:
1

Why?
0 is not present.
```

But:

```text
2 → 3 → 4
```

does not need to be checked separately because `1` already starts the sequence.

---

## Code

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> st(nums.begin(), nums.end());

        int longest = 0;

        for (int x : st) {
            // x is not the start of a sequence
            if (st.find(x - 1) != st.end()) {
                continue;
            }

            int current = x;
            int length = 1;

            while (st.find(current + 1) != st.end()) {
                current++;
                length++;
            }

            longest = max(longest, length);
        }

        return longest;
    }
};
```

---

## Basic Template

```cpp
int longestConsecutive(vector<int>& nums) {
    unordered_set<int> st(nums.begin(), nums.end());

    int longest = 0;

    for (int x : st) {
        // Start only from the beginning of a sequence
        if (st.find(x - 1) != st.end()) {
            continue;
        }

        int current = x;
        int length = 1;

        while (st.find(current + 1) != st.end()) {
            current++;
            length++;
        }

        longest = max(longest, length);
    }

    return longest;
}
```

### Reusable Pattern

```text
Put elements in Hash Set
        ↓
For each element x
        ↓
Is x - 1 absent?
   ↙           ↘
 No             Yes
 ↓               ↓
Skip        Start sequence
                ↓
          Expand x + 1, x + 2...
                ↓
          Track maximum length
```

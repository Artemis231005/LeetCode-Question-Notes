# LeetCode 3794 — Reverse String Prefix

## Metadata

* **LeetCode:** 3794
* **Problem:** Reverse String Prefix
* **Difficulty:** Easy
* **Topics:** String, Two Pointers
* **Pattern:** Prefix Reversal
* **Key Technique:** Find the required prefix boundary, then reverse only that prefix using two pointers.
* **Optimal Complexity:** **O(n)** time, **O(1)** extra space

---

## Approach 1 — Find Boundary + Reverse Prefix

### Idea

We are given a string `s` and a character `ch`.

* Find the **first occurrence** of `ch`.
* Reverse the substring from index `0` to that position.
* Leave the remaining suffix unchanged.

Instead of creating a separate substring, reverse the prefix **in-place** using two pointers.

### Dry Run

**Input:** `s = "abcdefd", ch = 'd'`

First occurrence of `d` is at index `3`.

Prefix:

`"abcd"`

Reverse it:

`"dcba"`

Remaining suffix:

`"efd"`

Result:

`"dcbaefd"`

### Algorithm

1. Find the first index `i` where `s[i] == ch`.
2. Set:

   * `left = 0`
   * `right = i`
3. While `left < right`:

   * Swap `s[left]` and `s[right]`.
   * Increment `left`.
   * Decrement `right`.
4. Return `s`.

### Complexity

* **Time:** `O(n)` — finding the character takes `O(n)` and reversing the prefix takes at most `O(n)`.
* **Space:** `O(1)` extra space.

### Code

```cpp
class Solution {
public:
    string reversePrefix(string s, char ch) {
        int index = -1;

        for (int i = 0; i < s.size(); i++) {
            if (s[i] == ch) {
                index = i;
                break;
            }
        }

        if (index == -1) {
            return s;
        }

        int left = 0;
        int right = index;

        while (left < right) {
            swap(s[left], s[right]);
            left++;
            right--;
        }

        return s;
    }
};
```

---

## Notes / Tips

* **Only the prefix is reversed**, not the entire string.
* Use the **first occurrence** of `ch`.
* If `ch` does not exist, return the original string unchanged.
* The two-pointer reversal is the standard way to reverse a range in-place.
* Remember that the right boundary is **inclusive**.

---

## Key Template

```cpp
index = -1

for i = 0 to s.size() - 1
    if s[i] == ch
        index = i
        break

if index == -1
    return s

left = 0
right = index

while left < right
    swap(s[left], s[right])
    left++
    right--
```

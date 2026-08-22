# Assign Cookies

## Problem

Given an array `g` where `g[i]` represents the **greed factor** of a child and an array `s` where `s[j]` represents the **size** of a cookie, assign cookies to children such that each child gets at most one cookie.

A child is satisfied if:

```text
cookie size >= child's greed factor
```

Return the **maximum number of satisfied children**.

---

## Approach 1: Greedy + Two Pointers

### Idea

Sort both arrays.

Use:

* `i` → current child
* `j` → current cookie

For each cookie:

* If the cookie can satisfy the current child, give it to them and move both pointers.
* Otherwise, the cookie is too small, so move only the cookie pointer.

The smallest cookie that can satisfy a child should be used for that child, preserving larger cookies for greedier children.

### Dry Run

```text
g = [1,2,3]
s = [1,1]

Sorted:
g = [1,2,3]
s = [1,1]

cookie 1 >= greed 1 → satisfy
cookie 1 < greed 2  → cannot satisfy

Answer = 1
```

Another example:

```text
g = [1,2]
s = [1,2,3]

1 >= 1 → satisfy
2 >= 2 → satisfy

Answer = 2
```

### Algorithm

1. Sort `g` and `s`.
2. Initialize `i = 0` for children.
3. Traverse cookies using `j`.
4. If `s[j] >= g[i]`:

   * The child is satisfied.
   * Increment `i`.
5. Always increment `j`.
6. Return `i`.

### Complexity

* Time: `O(n log n + m log m)`
* Space: `O(1)` excluding sorting space

where:

* `n = g.length()`
* `m = s.length()`

### Code

```cpp
class Solution {
public:
    int findContentChildren(vector<int>& g, vector<int>& s) {
        sort(g.begin(), g.end());
        sort(s.begin(), s.end());

        int i = 0;

        for (int j = 0; j < s.size() && i < g.size(); j++) {
            if (s[j] >= g[i]) {
                i++;
            }
        }

        return i;
    }
};
```

### Notes / Tips

* This is a classic **Greedy + Two Pointer** problem.
* Always try to satisfy the **least greedy child with the smallest possible cookie**.
* If a cookie is too small for the current child, it cannot satisfy any greedier child either, so safely discard it.
* Sorting is what makes the greedy choice possible.
* Think: **"Use the smallest sufficient resource."**

### Key Template

```text
sort requirements
sort resources

i = 0

for each resource:
    if resource >= requirement[i]:
        satisfy requirement
        i++

return i
```

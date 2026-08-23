# LeetCode 3803 — Count Residue Prefixes

## Metadata

* **LeetCode:** 3803
* **Problem:** Count Residue Prefixes
* **Difficulty:** Easy
* **Topics:** String, Hash Table, Simulation
* **Pattern:** Prefix Processing + Set for Distinct Elements
* **Optimal Complexity:** O(n) time, O(1) space

---

## Idea

For every prefix, we need to check:

```text
number of distinct characters == length of prefix % 3
```

Since prefixes are built one character at a time, maintain a **set of distinct characters** seen so far.

At each character:

1. Add it to the set.
2. Current prefix length is `i + 1`.
3. Check whether `set.size() == (i + 1) % 3`.
4. If yes, increment the answer.

Important observation:

* Length `1` → `1 % 3 = 1` → always a residue.
* Length `2` → `2 % 3 = 2` → residue only if both characters are distinct.
* Length `3` → `3 % 3 = 0` → impossible for a non-empty prefix because it always has at least 1 distinct character.
* The same pattern repeats for larger lengths.

---

## Dry Run

### `s = "abc"`

| Prefix  | Distinct | Length % 3 | Residue? | Count |
| ------- | -------: | ---------: | -------- | ----: |
| `"a"`   |        1 |          1 | Yes      |     1 |
| `"ab"`  |        2 |          2 | Yes      |     2 |
| `"abc"` |        3 |          0 | No       |     2 |

**Answer = 2**

### `s = "dd"`

| Prefix | Distinct | Length % 3 | Residue? | Count |
| ------ | -------: | ---------: | -------- | ----: |
| `"d"`  |        1 |          1 | Yes      |     1 |
| `"dd"` |        1 |          2 | No       |     1 |

**Answer = 1**

---

## Algorithm

1. Create an empty set `seen`.
2. Initialize `ans = 0`.
3. Traverse the string from left to right.
4. Insert the current character into `seen`.
5. Let the current prefix length be `i + 1`.
6. If `seen.size() == (i + 1) % 3`, increment `ans`.
7. Return `ans`.

---

## Complexity

* **Time:** `O(n)` — each character is processed once.
* **Space:** `O(1)` — the string contains only lowercase English letters, so the set can contain at most 26 characters.

---

## Code

```cpp
class Solution {
public:
    int residuePrefixes(string s) {
        unordered_set<char> seen;
        int ans = 0;

        for (int i = 0; i < s.size(); i++) {
            seen.insert(s[i]);

            if (seen.size() == (i + 1) % 3) {
                ans++;
            }
        }

        return ans;
    }
};
```

---

## Notes / Tips

* The key is that **prefixes overlap**, so don't recompute the distinct characters for every prefix.
* Maintain the distinct-character information incrementally.
* `unordered_set` automatically ignores duplicate characters.
* Always use `(i + 1)` for prefix length because `i` is zero-indexed.
* A prefix of length divisible by `3` can **never** be a residue because `length % 3 = 0`, while every non-empty prefix has at least one distinct character.

---

## Key Template

```text
seen = empty set
ans = 0

for every character:
    add character to seen

    if distinct_count == prefix_length % k:
        ans++
```

**Pattern:** When a problem asks about a property of every prefix, check whether the required information can be maintained incrementally while scanning left to right.

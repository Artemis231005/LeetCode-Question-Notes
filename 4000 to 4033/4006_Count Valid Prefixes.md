# LeetCode 4006 — Count Valid Prefixes

## Metadata

* **LeetCode:** 4006
* **Problem:** Count Valid Prefixes
* **Difficulty:** Easy
* **Topics:** String, Prefix, Counting
* **Pattern:** Prefix Counting + Frequency Difference
* **Optimal Complexity:** O(n) time, O(1) space

---

## Idea

A prefix is **valid** if its characters can be rearranged to form an alternating binary string.

For a binary string to be rearranged into an alternating string:

```text
number of 0s and number of 1s must differ by at most 1
```

So while traversing the string, maintain:

```text
diff = count(1) - count(0)
```

For every prefix:

```text
if |diff| <= 1
    prefix is valid
```

There is no need to actually rearrange the prefix.

---

## Dry Run

### `s = "00101"`

| Prefix    | #0 | #1 | `diff` | Valid? |
| --------- | -: | -: | -----: | ------ |
| `"0"`     |  1 |  0 |     -1 | Yes    |
| `"00"`    |  2 |  0 |     -2 | No     |
| `"001"`   |  2 |  1 |     -1 | Yes    |
| `"0010"`  |  3 |  1 |     -2 | No     |
| `"00101"` |  3 |  2 |     -1 | Yes    |

**Answer = 3**

The valid prefixes are:

```text
"0"
"001"
"00101"
```

---

## Algorithm

1. Initialize `diff = 0` and `ans = 0`.
2. Traverse the string character by character.
3. If the character is `'1'`, increment `diff`.
4. Otherwise, decrement `diff`.
5. If `abs(diff) <= 1`, increment `ans`.
6. Return `ans`.

---

## Complexity

* **Time:** `O(n)` — one traversal of the string.
* **Space:** `O(1)` — only a few variables are required.

---

## Code

```cpp
class Solution {
public:
    int countValidPrefixes(string s) {
        int diff = 0;
        int ans = 0;

        for (char c : s) {
            if (c == '1') {
                diff++;
            } else {
                diff--;
            }

            if (abs(diff) <= 1) {
                ans++;
            }
        }

        return ans;
    }
};
```

---

## Notes / Tips

* **Key observation:** A binary string can be rearranged into an alternating string iff `|count(0) - count(1)| <= 1`.
* We only need the **difference**, not the two individual frequencies.
* This is a classic **prefix invariant**: update the state as the prefix grows and check the condition immediately.
* Don't confuse this with checking whether the prefix is **already** alternating. Rearrangement is allowed.
* `diff = count(1) - count(0)` makes the condition simply `abs(diff) <= 1`.

---

## Key Template

```text
diff = 0
ans = 0

for every character:
    if character == '1':
        diff++
    else:
        diff--

    if abs(diff) <= 1:
        ans++
```

**Pattern:** When a prefix is valid based on the relative frequency of two characters, maintain their **count difference** while scanning.

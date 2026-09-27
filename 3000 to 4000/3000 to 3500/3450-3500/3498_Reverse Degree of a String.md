# LeetCode 3498 — Reverse Degree of a String

## Metadata

* **LeetCode:** 3498
* **Problem:** Reverse Degree of a String
* **Difficulty:** Easy
* **Topics:** String, Math
* **Pattern:** Direct Arithmetic Accumulation
* **Key Technique:** A character's position in the reversed alphabet is just `26 - (c - 'a')`, computable directly with arithmetic — no lookup table is needed
* **Optimal Complexity:** `O(n)` Time, `O(1)` Space

---

## Problem Statement

Given a string `s`, for each character multiply its position in the reversed alphabet (`'a'=26, 'b'=25, ..., 'z'=1`) by its 1-indexed position in the string, and return the sum of all these products.

---

## Approaches

1. **Brute Force — Precompute a Reversed-Alphabet Lookup Table**
2. **Optimal — Direct Arithmetic, No Lookup Table**

---

# Approach 1 — Brute Force / Precompute a Reversed-Alphabet Lookup Table

## Idea

Build an explicit array mapping each letter to its reversed-alphabet position (`'a'` → `26`, `'b'` → `25`, ..., `'z'` → `1`) by filling it in from `'z'` down to `'a'`. Then scan the string once, looking up each character's reversed value from the table and multiplying it by its 1-indexed position.

## Dry Run

```text
s = "abc"
```

Build lookup table:

```text
table['a'] = 26, table['b'] = 25, table['c'] = 24, ..., table['z'] = 1
```

Scan `s`:

```text
i=0, 'a': table['a']=26, position=1 → product=26
i=1, 'b': table['b']=25, position=2 → product=50
i=2, 'c': table['c']=24, position=3 → product=72
```

Sum: `26 + 50 + 72 = 148`, matching the expected output.

## Algorithm

1. Build a `reversedValue` array of size `26`, where `reversedValue[i] = 26 - i` for `i` from `0` (`'a'`) to `25` (`'z'`).
2. Initialize `total = 0`.
3. For each index `i` and character `c` in `s`:

   * `total += reversedValue[c - 'a'] * (i + 1)`.
4. Return `total`.

## Complexity

* **Time:** `O(n)`

  * Building the fixed-size lookup table is `O(26)`, and the main scan is `O(n)`.
* **Space:** `O(1)`

  * The lookup table is a fixed size of `26`, independent of `n`.

## Notes / Tips

* Building a lookup table here is unnecessary overhead — the reversed-alphabet value for any character can be computed directly with a single subtraction (`26 - (c - 'a')`), which is exactly what Approach 2 does instead.
* This pattern (precomputing a small fixed-size table vs. computing the value inline) is a stylistic choice more than a real optimization for a 26-letter alphabet, but recognizing when a lookup table is genuinely unnecessary avoids adding needless setup code.

## Code

```cpp
class Solution {
public:
    int reverseDegree(string s) {
        vector<int> reversedValue(26);
        for (int i = 0; i < 26; i++) {
            reversedValue[i] = 26 - i;
        }

        long long total = 0;

        for (int i = 0; i < s.size(); i++) {
            total += (long long)reversedValue[s[i] - 'a'] * (i + 1);
        }

        return (int)total;
    }
};
```

---

# Approach 2 — Optimal / Direct Arithmetic, No Lookup Table

## Idea

A character's reversed-alphabet position is simply `26 - (c - 'a')` — there's no need to precompute or store this in a table at all, since it's a one-line arithmetic expression. Scan the string once, computing each character's reversed value inline and accumulating the product with its 1-indexed position directly.

## Dry Run

```text
s = "zaza"
```

Process:

```text
i=0, 'z': reversedValue = 26 - ('z'-'a') = 26 - 25 = 1, position=1 → product=1
i=1, 'a': reversedValue = 26 - ('a'-'a') = 26 - 0 = 26, position=2 → product=52
i=2, 'z': reversedValue = 1, position=3 → product=3
i=3, 'a': reversedValue = 26, position=4 → product=104
```

Sum: `1 + 52 + 3 + 104 = 160`, matching the expected output.

## Algorithm

1. Initialize `total = 0`.
2. For each index `i` and character `c` in `s`:

   * `reversedValue = 26 - (c - 'a')`.
   * `total += reversedValue * (i + 1)`.
3. Return `total`.

## Complexity

* **Time:** `O(n)`

  * A single pass over the string, constant work per character.
* **Space:** `O(1)`

  * Only a running total — no lookup table or extra structures.

## Notes / Tips

* The formula `26 - (c - 'a')` directly encodes the reversed-alphabet mapping: `'a'` (`c - 'a' = 0`) maps to `26`, and `'z'` (`c - 'a' = 25`) maps to `1` — no table needed since it's a simple linear relationship.
* Using a wider type (`long long`) for the running total is a safe habit here even though `n <= 1000` keeps the actual maximum sum well within a 32-bit `int`'s range — worth doing by default whenever summing many multiplied terms.
## Code

```cpp
class Solution {
public:
    int reverseDegree(string s) {
        long long total = 0;

        for (int i = 0; i < s.size(); i++) {
            int reversedValue = 26 - (s[i] - 'a');
            total += (long long)reversedValue * (i + 1);
        }

        return (int)total;
    }
};
```

---

## Key Template

```text
total = 0

for i, c in enumerate(s):
    reversedValue = 26 - (c - 'a')
    total += reversedValue * (i + 1)

return total
```
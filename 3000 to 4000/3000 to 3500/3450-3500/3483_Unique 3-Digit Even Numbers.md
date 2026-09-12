# LeetCode 3483 — Unique 3-Digit Even Numbers

## Metadata

* **LeetCode:** 3483
* **Problem:** Unique 3-Digit Even Numbers
* **Difficulty:** Easy
* **Topics:** Array, Hash Table, Math, Enumeration, Counting
* **Pattern:** Frequency Counting + Fixed-Range Enumeration
* **Key Technique:** Since only 900 possible 3-digit numbers exist, check each one directly against the available digit counts instead of generating and deduplicating permutations
* **Optimal Complexity:** `O(1)` Time (bounded by the fixed range of 3-digit numbers), `O(1)` Space

---

## Problem Statement

Given an array of digits `digits`, return the number of distinct 3-digit even numbers that can be formed using them, where each copy of a digit can only be used once per number, and leading zeros aren't allowed.

---

## Approaches

1. **Brute Force — Generate Every Ordered Triple of Indices**
2. **Optimal — Frequency Count + Enumerate All 3-Digit Even Numbers**

---

# Approach 1 — Brute Force / Generate Every Ordered Triple of Indices

## Idea

Directly simulate "pick three digits and arrange them": try every ordered triple of **distinct indices** `(i, j, k)` from the `digits` array, form the 3-digit number `digits[i]*100 + digits[j]*10 + digits[k]`, and check whether it's a valid result (no leading zero, last digit even). Insert every valid number into a hash set to automatically deduplicate numbers formed from different index combinations that happen to produce the same value.

## Dry Run

```text
digits = [1, 2, 3, 4]
```

Try indices `(0, 1, 2)` → digits `1, 2, 3` → number `123`:

```text
first digit 1 != 0 → ok
last digit 3 is odd → invalid, skip
```

Try indices `(0, 1, 3)` → digits `1, 2, 4` → number `124`:

```text
first digit 1 != 0 → ok
last digit 4 is even → valid → add 124 to set
```

Continue through all `4 * 3 * 2 = 24` ordered index triples, inserting every valid formed number into the set.

Final set size: `12`, matching the expected output.

## Algorithm

1. Initialize an empty hash set `results`.
2. For each ordered triple of distinct indices `(i, j, k)` in `digits` (i.e. `i != j`, `j != k`, `i != k`):

   * If `digits[i] == 0`, skip (leading zero not allowed).
   * If `digits[k] % 2 != 0`, skip (must be even).
   * Compute `number = digits[i]*100 + digits[j]*10 + digits[k]`.
   * Insert `number` into `results`.
3. Return `results.size()`.

## Complexity

* **Time:** `O(n³)`

  * Three nested loops over indices, with `n <= 10` per the problem's constraints, so this is at most `10 * 9 * 8 = 720` iterations — trivially fast in practice despite the cubic shape.
* **Space:** `O(n³)`

  * For the hash set, which in the worst case could hold up to `n * (n-1) * (n-2)` distinct formed numbers (bounded further by the fixed range of 900 possible 3-digit numbers).

## Notes / Tips

* Using **indices** rather than values is essential here — treating repeated digit values (like the two `2`s in `[0,2,2]`) as literally different array slots is what correctly allows a digit to be reused up to as many times as it physically appears, without allowing more reuse than that.
* The hash set handles deduplication naturally (e.g. two different index triples producing the same number, like using either copy of a repeated digit), but this is somewhat wasteful given how few distinct 3-digit numbers can ever exist in total.

## Code

```cpp
class Solution {
public:
    int totalNumbers(vector<int>& digits) {
        unordered_set<int> results;
        int n = digits.size();

        for (int i = 0; i < n; i++) {
            if (digits[i] == 0) continue;

            for (int j = 0; j < n; j++) {
                if (j == i) continue;

                for (int k = 0; k < n; k++) {
                    if (k == i || k == j) continue;
                    if (digits[k] % 2 != 0) continue;

                    int number = digits[i] * 100 + digits[j] * 10 + digits[k];
                    results.insert(number);
                }
            }
        }

        return results.size();
    }
};
```

---

# Approach 2 — Optimal / Frequency Count + Enumerate All 3-Digit Even Numbers

## Idea

There are only `900` three-digit numbers in total (`100` to `999`), and only about half of those are even — a fixed, tiny range regardless of how large `digits` is. Instead of generating permutations from `digits`, build a frequency count of each available digit (`0`-`9`), then directly check every even 3-digit number `x`: does the available digit count cover the digits `x` requires? This sidesteps permutation generation and deduplication entirely.

## Dry Run

```text
digits = [0, 2, 2]
```

Build frequency count:

```text
count[0] = 1, count[2] = 2
```

Check `x = 202`:

```text
digits needed: 2 (hundreds), 0 (tens), 2 (units)
needed count: {2: 2, 0: 1}
available: count[2]=2 >= 2 ✓, count[0]=1 >= 1 ✓ → valid
```

Check `x = 220`:

```text
digits needed: 2, 2, 0 → needed count: {2: 2, 0: 1}
same as above → valid
```

Check `x = 222`:

```text
needed count: {2: 3}
available count[2] = 2 < 3 → invalid
```

Continuing through all even 3-digit numbers, only `202` and `220` pass → final count `2`, matching the expected output.

## Algorithm

1. Build a frequency array `count` of size `10`, counting occurrences of each digit in `digits`.
2. Initialize `total = 0`.
3. For each even number `x` from `100` to `998` (stepping by `2`):

   * Extract its three digits: `hundreds = x/100`, `tens = (x/10)%10`, `units = x%10`.
   * Build a small "needed" count for these three digits (accounting for repeats, e.g. `222` needs three `2`s).
   * Check if `count` has enough of each needed digit; if so, increment `total`.
4. Return `total`.

## Complexity

* **Time:** `O(1)`

  * Bounded by the fixed range of `450` even 3-digit numbers to check, each requiring only a few constant-time count comparisons — independent of `digits.length`.
* **Space:** `O(1)`

  * The frequency array is a fixed size of `10`, and the per-number "needed" count is also fixed size.

## Notes / Tips

* Instead of generating numbers *from* the available digits, it checks every *possible* number against the available digits — since the output range (900 numbers) is far smaller and more fixed than the input's potential permutation count, this direction is strictly cheaper.
* Correctly handling repeated required digits (e.g. `222` needing three `2`s, or `202` needing two `2`s) requires building a small per-candidate frequency count rather than a naive "does digit X exist somewhere" check — otherwise a digit appearing once in `digits` could incorrectly satisfy a number that needs it multiple times.
* Given the problem's extremely small constraints (`digits.length <= 10`), both approaches run instantly, but this approach's complexity doesn't change at all even if the input array were allowed to grow much larger, unlike Approach 1's cubic dependence on `n`.

## Code

```cpp
class Solution {
public:
    int totalNumbers(vector<int>& digits) {
        vector<int> count(10, 0);
        for (int d : digits) {
            count[d]++;
        }

        int total = 0;

        for (int x = 100; x <= 998; x += 2) {
            int hundreds = x / 100;
            int tens = (x / 10) % 10;
            int units = x % 10;

            vector<int> needed(10, 0);
            needed[hundreds]++;
            needed[tens]++;
            needed[units]++;

            bool feasible = true;
            for (int d = 0; d < 10; d++) {
                if (needed[d] > count[d]) {
                    feasible = false;
                    break;
                }
            }

            if (feasible) {
                total++;
            }
        }

        return total;
    }
};
```

---

## Key Template

```text
count = array of size 10, all 0
for d in digits: count[d] += 1

total = 0
for x = 100 to 998, step 2:
    hundreds = x / 100
    tens = (x / 10) % 10
    units = x % 10

    needed = array of size 10, all 0
    needed[hundreds] += 1
    needed[tens] += 1
    needed[units] += 1

    if needed[d] <= count[d] for all d in 0..9:
        total += 1

return total
```
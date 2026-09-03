# LeetCode 60 — Permutation Sequence

## Metadata

* **LeetCode:** 60
* **Problem:** Permutation Sequence
* **Difficulty:** Hard
* **Topics:** Math, Recursion
* **Pattern:** Factorial Number System (Cantor's Algorithm)
* **Key Technique:** Divide `k` by decreasing factorials to directly pick each digit, removing it from the candidate list each time
* **Optimal Complexity:** `O(n²)` Time, `O(n)` Space

---

## Approaches

1. **Brute Force — Generate Permutations in Order**
2. **Optimal — Factorial Number System (Cantor's Algorithm)**

---

# Approach 1 — Brute Force / Generate Permutations in Order

## Idea

Generate permutations of `1..n` one at a time in lexicographic order (e.g. using `next_permutation`), starting from the smallest. Stop once the `k`th one is reached.

## Dry Run

For `n = 3, k = 3`:

```text
start: [1, 2, 3]   (1st permutation)
next:  [1, 3, 2]   (2nd)
next:  [2, 1, 3]   (3rd) ← stop, this is k = 3
```

Result:

```text
"213"
```

## Algorithm

1. Build the initial sorted sequence `[1, 2, ..., n]`.
2. Call `next_permutation` a total of `k - 1` times to advance from the 1st permutation to the `k`th.
3. Convert the final sequence to a string and return it.

## Complexity

* **Time:** `O(k * n)`
  * Each `next_permutation` call is `O(n)`, done `k - 1` times.
* **Space:** `O(n)`
  * For the sequence itself.

## Notes / Tips

* Simple and directly uses the standard library, but degrades badly when `k` is large — worst case `k` can be close to `n!`, making this effectively `O(n! * n)`.
* Fine only because constraints cap `n <= 9`, so `n!` is at most `362880`.

## Code

```cpp
class Solution {
public:
    string getPermutation(int n, int k) {
        vector<int> seq;
        for (int i = 1; i <= n; i++) {
            seq.push_back(i);
        }

        for (int i = 0; i < k - 1; i++) {
            next_permutation(seq.begin(), seq.end());
        }

        string result = "";
        for (int num : seq) {
            result += to_string(num);
        }

        return result;
    }
};
```

---

# Approach 2 — Optimal / Factorial Number System (Cantor's Algorithm)

## Idea

There are `n!` total permutations of `1..n`. If we fix the first digit, the remaining `n-1` digits can be arranged in `(n-1)!` ways. So the first `(n-1)!` permutations all start with digit `1`, the next `(n-1)!` all start with digit `2`, and so on.

This means:

```text
index of first digit = (k - 1) / (n - 1)!
```

Once the first digit is picked and removed from the candidate list, the same logic applies to the remaining `n-1` digits with `(n-2)!` as the new group size — and so on, shrinking by one digit and one factorial level each step.

## Dry Run

For `n = 4, k = 9`:

```text
digits = [1, 2, 3, 4]
k = 9 → k-- → k = 8   (0-indexed)
```

### Step 1

```text
group size = 3! = 6
index = 8 / 6 = 1 → digits[1] = 2 → pick '2'
digits = [1, 3, 4]
k = 8 % 6 = 2
```

### Step 2

```text
group size = 2! = 2
index = 2 / 2 = 1 → digits[1] = 3 → pick '3'
digits = [1, 4]
k = 2 % 2 = 0
```

### Step 3

```text
group size = 1! = 1
index = 0 / 1 = 0 → digits[0] = 1 → pick '1'
digits = [4]
k = 0 % 1 = 0
```

### Step 4

```text
group size = 0! = 1
index = 0 / 1 = 0 → digits[0] = 4 → pick '4'
```

Result:

```text
"2314"
```

## Algorithm

1. Precompute factorials `0!` through `(n-1)!`.
2. Build a list of candidate digits `[1, 2, ..., n]`.
3. Decrement `k` by `1` to make it 0-indexed.
4. For `i` from `n` down to `1`:

   * `factorial = (i - 1)!`
   * `index = k / factorial`
   * Append `digits[index]` to the result.
   * Remove `digits[index]` from the candidate list.
   * `k = k % factorial`
5. Return the result.

## Why Do We Need `k--`?

`k` is given as **1-indexed** (the 1st permutation, the 2nd, etc.), but the factorial grouping logic is naturally **0-indexed** — the first group of `(n-1)!` permutations corresponds to index `0`, not index `1`.

```text
k = 1 → should land in the very first group → index 0
```

Without `k--`, dividing `k` directly by the factorial would misalign every group boundary by one. Subtracting `1` upfront fixes this once, so the rest of the algorithm can treat `k` as a plain 0-indexed offset.

## Complexity

* **Time:** `O(n²)`
  * `n` steps, and removing an element from the candidate list takes `O(n)` each time.
* **Space:** `O(n)`
  * For the candidate digit list and the output string.

## Notes / Tips

* This is known as **Cantor's algorithm** — converting `k` into the factorial number system directly gives the permutation without generating any others.
* Constraints keep `n <= 9`, so `9!` comfortably fits in a normal `int`.
* Common mistake: forgetting `k--` at the start, which shifts every digit choice off by one group.
* Removing from a `vector` is `O(n)` per step; a `list` or Fenwick-tree-based index structure can bring this down to `O(n log n)` total, but isn't necessary at this problem's constraints.

## Code

```cpp
class Solution {
public:
    string getPermutation(int n, int k) {
        vector<int> factorial(n + 1, 1);
        for (int i = 1; i <= n; i++) {
            factorial[i] = factorial[i - 1] * i;
        }

        vector<char> digits;
        for (int i = 1; i <= n; i++) {
            digits.push_back('0' + i);
        }

        k--;
        string result = "";

        for (int i = n; i >= 1; i--) {
            int index = k / factorial[i - 1];
            result += digits[index];
            digits.erase(digits.begin() + index);
            k %= factorial[i - 1];
        }

        return result;
    }
};
```

---

## Key Template

```text
factorial[0..n] precomputed
digits = [1, 2, ..., n]
k--

for i = n down to 1:
    index = k / factorial[i - 1]
    result += digits[index]
    digits.erase(index)
    k %= factorial[i - 1]

return result
```
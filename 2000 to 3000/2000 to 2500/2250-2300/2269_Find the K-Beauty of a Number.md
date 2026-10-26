# LeetCode 2269 — Find the K-Beauty of a Number

## Metadata

* **LeetCode:** 2269
* **Problem:** Find the K-Beauty of a Number
* **Difficulty:** Easy
* **Topics:** Math, String, Sliding Window
* **Pattern:** Fixed-Size Sliding Window over Digits
* **Key Technique:** Slide a length-`k` window across the number's digits, treating each window as a number and checking whether it's a nonzero divisor of the original — maintaining the window value incrementally instead of re-parsing a substring each time
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space (where `n` is the number of digits)

---

## Problem Statement

Given an integer `num` and an integer `k`, the **k-beauty** of `num` is the number of substrings of `num` (viewed as a string of digits) with length exactly `k` that, when interpreted as integers, are **nonzero divisors** of `num`. Return `num`'s k-beauty.

---

## Approaches

1. **Brute Force — Convert and Parse Every Substring**
2. **Optimal — Rolling Window Value Over Digits**

---

# Approach 1 — Brute Force / Convert and Parse Every Substring

## Idea

Convert `num` to a string, then for every window of length `k`, extract the substring, parse it back into an integer, and check whether it's nonzero and divides `num` evenly.

## Dry Run

```text
num = 430043, k = 2
```

String: `"430043"`.

Window `"43"` (indices 0-1): parse → `43`. `430043 % 43 == 0` → valid, count = 1.

Window `"30"` (indices 1-2): parse → `30`. `430043 % 30 != 0` → not valid.

Window `"00"` (indices 2-3): parse → `0`. Zero is not a valid divisor → skip.

Window `"04"` (indices 3-4): parse → `4`. `430043 % 4 != 0` → not valid.

Window `"43"` (indices 4-5): parse → `43`. `430043 % 43 == 0` → valid, count = 2.

Final k-beauty: `2`.

## Algorithm

1. Convert `num` to a string `s`.
2. Initialize `count = 0`.
3. For each start index `i` from `0` to `s.length() - k`:

   * Extract `sub = s.substr(i, k)` and parse it into an integer `val`.
   * If `val != 0` and `num % val == 0`, increment `count`.
4. Return `count`.

## Complexity

* **Time:** `O(n * k)`

  * For each of the `n - k + 1` windows, extracting and parsing a length-`k` substring takes `O(k)`.
* **Space:** `O(k)`

  * For the temporary substring built at each step.

## Notes / Tips

* Re-extracting and re-parsing each window from scratch repeats overlapping work — consecutive windows share `k - 1` digits, so the new window's value can be derived from the previous one with a constant number of arithmetic operations instead.
* Skipping `val == 0` before taking the modulo is essential — dividing by zero is undefined, and the problem explicitly states a divisor must be nonzero.

## Code

```cpp
class Solution {
public:
    int divisorSubstrings(int num, int k) {
        string s = to_string(num);
        int count = 0;

        for (int i = 0; i + k <= (int)s.size(); i++) {
            int val = stoi(s.substr(i, k));

            if (val != 0 && num % val == 0) {
                count++;
            }
        }

        return count;
    }
};
```

---

# Approach 2 — Optimal / Rolling Window Value Over Digits

## Idea

Maintain the current window's numeric value incrementally as the window slides: when moving one step right, remove the leading digit's contribution (by taking the value modulo `10^(k-1)`), shift everything left by one place (multiply by `10`), and add the new trailing digit. This updates the window value in `O(1)` instead of re-parsing the whole substring every time.

## Dry Run

```text
num = 430043, k = 2
```

Digits: `4,3,0,0,4,3`. Precompute `pow = 10^(k-1) = 10`.

Initialize the first window `"43"`:

```text
windowVal = 4*10 + 3 = 43
430043 % 43 == 0 → valid, count = 1
```

Slide to window `"30"` (drop leading `4`, add trailing `0`):

```text
windowVal = (43 % 10) * 10 + 0 = 3*10 + 0 = 30
430043 % 30 != 0 → not valid
```

Slide to window `"00"`:

```text
windowVal = (30 % 10) * 10 + 0 = 0
windowVal == 0 → skip (zero not a valid divisor)
```

Slide to window `"04"`:

```text
windowVal = (0 % 10) * 10 + 4 = 4
430043 % 4 != 0 → not valid
```

Slide to window `"43"`:

```text
windowVal = (4 % 10) * 10 + 3 = 43
430043 % 43 == 0 → valid, count = 2
```

Final k-beauty: `2`, matching the brute-force result.

## Algorithm

1. Convert `num` to a string `s` (or extract digits via arithmetic).
2. Compute `pow = 10^(k-1)`.
3. Compute the first window's value from the first `k` digits.
4. Check the first window: if `windowVal != 0` and `num % windowVal == 0`, increment `count`.
5. For each subsequent position `i` from `k` to `s.length() - 1`:

   * Update `windowVal = (windowVal % pow) * 10 + (s[i] - '0')`.
   * If `windowVal != 0` and `num % windowVal == 0`, increment `count`.
6. Return `count`.

## Complexity

* **Time:** `O(n)`

  * One pass to build the first window (`O(k)`) and one `O(1)` update per remaining window.
* **Space:** `O(1)`

  * Only a running window value and a precomputed power of `10` — no substrings created.

## Notes / Tips

* The expression `(windowVal % pow) * 10 + newDigit` is the numeric equivalent of "drop the leading digit, shift left, append the new digit" — `% pow` strips the leading digit (since `pow = 10^(k-1)` is the place value of the leading digit in a `k`-digit window), and `* 10` shifts the remaining digits up one place.
* This is the same rolling-window idea behind rolling hash techniques (e.g. Rabin-Karp), just applied to an exact numeric value instead of a hash — maintaining a window's value incrementally avoids recomputing from scratch at every position.
* Given this problem's small constraints (`num <= 10^9`, so at most `10` digits), both approaches run essentially instantly in practice, but the rolling approach avoids all string slicing and parsing overhead entirely.

## Code

```cpp
class Solution {
public:
    int divisorSubstrings(int num, int k) {
        string s = to_string(num);
        int n = s.size();

        long long pow = 1;
        for (int i = 0; i < k - 1; i++) {
            pow *= 10;
        }

        long long windowVal = 0;
        for (int i = 0; i < k; i++) {
            windowVal = windowVal * 10 + (s[i] - '0');
        }

        int count = 0;
        if (windowVal != 0 && num % windowVal == 0) {
            count++;
        }

        for (int i = k; i < n; i++) {
            windowVal = (windowVal % pow) * 10 + (s[i] - '0');

            if (windowVal != 0 && num % windowVal == 0) {
                count++;
            }
        }

        return count;
    }
};
```

---

## Key Template

```text
s = digits of num
pow = 10^(k-1)

windowVal = value of first k digits
count = 1 if windowVal != 0 and num % windowVal == 0 else 0

for i in k..n-1:
    windowVal = (windowVal % pow) * 10 + digit(s[i])
    if windowVal != 0 and num % windowVal == 0:
        count += 1

return count
```
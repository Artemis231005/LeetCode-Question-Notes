# LeetCode 9 — Palindrome Number

## Metadata

* **LeetCode:** 9
* **Problem:** Palindrome Number
* **Difficulty:** Easy
* **Topics:** Math
* **Pattern:** Number Reversal
* **Key Pattern:** Compare a number with its reversed form
* **Key Template:** Reverse Half of Number
* **Key Technique:** Reverse only half the digits to avoid unnecessary work and overflow
* **Optimal Complexity:** `O(log n)` time, `O(1)` space

---

# Approaches

1. **Brute Force — Convert to String**
2. **Better — Reverse the Entire Number**
3. **Optimal — Reverse Half of the Number**

---

# Approach 1 — Brute Force / String Conversion

## Idea

Convert the integer into a string and check whether the string reads the same forward and backward.

Use two pointers:

* `left` → beginning
* `right` → end

If characters differ, the number is not a palindrome.

## Dry Run

```text
x = 121
```

Convert:

```text
"121"
```

Compare:

```text
'1' == '1' ✓
'2' == '2' ✓
```

Therefore:

```text
true
```

For:

```text
x = 123
```

```text
'1' != '3'
```

Therefore:

```text
false
```

## Algorithm

1. Convert `x` to a string.
2. Initialize `left = 0` and `right = length - 1`.
3. Compare characters at `left` and `right`.
4. If they differ, return `false`.
5. Move both pointers inward.
6. If all characters match, return `true`.

## Complexity

Let `d` be the number of digits.

* **Time:** `O(d)`
* **Space:** `O(d)`

## Notes / Tips

* Very simple and readable.
* Uses extra memory for the string.
* The problem can be solved using pure arithmetic with `O(1)` space.

## Code

```cpp
class Solution {
public:
    bool isPalindrome(int x) {
        string s = to_string(x);

        int left = 0;
        int right = s.size() - 1;

        while (left < right) {
            if (s[left] != s[right]) {
                return false;
            }

            left++;
            right--;
        }

        return true;
    }
};
```

---

# Approach 2 — Better / Reverse Entire Number

## Idea

A number is a palindrome if it is equal to its reversed form.

For example:

```text
121 → 121
123 → 321
```

So:

```text
x == reverse(x)
```

means the number is a palindrome.

## Dry Run

```text
x = 121
```

Reverse the number digit by digit:

```text
121
 ↓
1
12
121
```

Therefore:

```text
reverse = 121
```

Since:

```text
121 == 121
```

return `true`.

### Example

```text
x = 123
```

```text
reverse = 321
```

Since:

```text
123 != 321
```

return `false`.

## Algorithm

1. Store the original number.
2. Initialize `reverse = 0`.
3. Extract the last digit:

   ```text
   digit = x % 10
   ```
4. Append it to `reverse`:

   ```text
   reverse = reverse × 10 + digit
   ```
5. Remove the last digit from `x`:

   ```text
   x /= 10
   ```
6. Repeat until `x == 0`.
7. Compare `reverse` with the original number.

## Complexity

Let `d` be the number of digits.

* **Time:** `O(d)`
* **Space:** `O(1)`

## Notes / Tips

* Pure arithmetic solution.
* Negative numbers are automatically rejected because their reverse will not equal the original.
* For example:

  ```text
  -121 → -121
  ```

  but the problem defines negative numbers as not palindromes.
* In general integer-reversal problems, full reversal can potentially cause overflow.

## Code

```cpp
class Solution {
public:
    bool isPalindrome(int x) {
        if (x < 0) {
            return false;
        }

        int original = x;
        long long reverse = 0;

        while (x > 0) {
            int digit = x % 10;
            reverse = reverse * 10 + digit;
            x /= 10;
        }

        return reverse == original;
    }
};
```

---

# Approach 3 — Optimal / Reverse Half of the Number

## Idea

We do not need to reverse the entire number.

For a palindrome, the first half of the digits must match the reversed second half.

Therefore, reverse only the **last half** of the number.

### Example

```text
x = 1221
```

Split conceptually:

```text
12 | 21
```

Reverse the second half:

```text
21 → 12
```

Now compare:

```text
12 == 12
```

Therefore it is a palindrome.

For an odd number of digits:

```text
12321
```

we get:

```text
12 | 3 | 21
```

The middle digit does not matter.

After reversing the second half:

```text
12321
      ↓
12 | 123
```

Remove the middle digit from the reversed half:

```text
123 / 10 = 12
```

Then compare:

```text
12 == 12
```

## Important Observations

### Negative numbers

A negative number cannot be a palindrome:

```text
-121 → false
```

### Numbers ending in zero

If:

```text
x % 10 == 0
```

then the number cannot be a palindrome unless:

```text
x == 0
```

For example:

```text
10 → 01
```

which is not a valid representation of `10`.

So:

```cpp
if (x % 10 == 0 && x != 0) {
    return false;
}
```

### When to stop?

Keep reversing digits until:

```text
reversedHalf >= x
```

At this point, half the digits have been processed.

## Dry Run

```text
x = 1221
```

Initial:

```text
x = 1221
reversedHalf = 0
```

### Step 1

Last digit:

```text
1
```

```text
reversedHalf = 1
x = 122
```

### Step 2

Last digit:

```text
2
```

```text
reversedHalf = 12
x = 12
```

Now:

```text
reversedHalf >= x
12 >= 12
```

Stop.

Compare:

```text
x == reversedHalf
12 == 12
```

Therefore:

```text
true
```

### Odd-Length Example

```text
x = 12321
```

After reversing half:

```text
x = 12
reversedHalf = 123
```

Since the reversed half contains the middle digit, remove it:

```text
reversedHalf / 10 = 12
```

Compare:

```text
x == reversedHalf
12 == 12
```

Therefore:

```text
true
```

## Algorithm

1. If `x < 0`, return `false`.
2. If `x` ends in `0` and `x != 0`, return `false`.
3. Initialize:

   ```text
   reversedHalf = 0
   ```
4. While:

   ```text
   reversedHalf < x
   ```

   do:

   * Extract the last digit:

     ```text
     digit = x % 10
     ```
   * Add it to `reversedHalf`:

     ```text
     reversedHalf = reversedHalf × 10 + digit
     ```
   * Remove the last digit from `x`:

     ```text
     x /= 10
     ```
5. For an even number of digits:

   ```text
   x == reversedHalf
   ```
6. For an odd number of digits:

   ```text
   x == reversedHalf / 10
   ```
7. Return whether either condition is true.

## Complexity

Let `d` be the number of digits.

* **Time:** `O(d)`
* **Space:** `O(1)`

Only half the digits are processed, so more precisely the number of iterations is approximately `d / 2`.

## Notes / Tips

* This is the preferred arithmetic solution.
* Only half the number needs to be reversed.
* It avoids constructing a string.
* It avoids reversing the complete number.
* The key comparison is:

  ```text
  x == reversedHalf
  ```

  for even digits, or:

  ```text
  x == reversedHalf / 10
  ```

  for odd digits.
* The `x % 10 == 0` check handles numbers ending in zero.

## Code

```cpp
class Solution {
public:
    bool isPalindrome(int x) {
        if (x < 0) {
            return false;
        }

        if (x % 10 == 0 && x != 0) {
            return false;
        }

        int reversedHalf = 0;

        while (reversedHalf < x) {
            reversedHalf = reversedHalf * 10 + x % 10;
            x /= 10;
        }

        return x == reversedHalf || x == reversedHalf / 10;
    }
};
```

---

# Approach Comparison

| Approach              |   Time |  Space | Status      |
| --------------------- | -----: | -----: | ----------- |
| String Conversion     | `O(d)` | `O(d)` | Brute       |
| Reverse Entire Number | `O(d)` | `O(1)` | Better      |
| Reverse Half          | `O(d)` | `O(1)` | **Optimal** |

Where `d` is the number of digits.

---

# Key Template

### Reverse Digits

```cpp
int reversed = 0;

while (x > 0) {
    int digit = x % 10;
    reversed = reversed * 10 + digit;
    x /= 10;
}
```

### Reverse Half of a Number

```cpp
int reversedHalf = 0;

while (reversedHalf < x) {
    reversedHalf = reversedHalf * 10 + x % 10;
    x /= 10;
}

return x == reversedHalf || x == reversedHalf / 10;
```

## Pattern Recognition

When you see:

```text
Number
+
Palindrome
+
O(1) extra space
```

Think:

```text
Reverse digits
        ↓
Better: reverse entire number
        ↓
Optimal: reverse only half
```

The key observation is:

> **A palindrome's first half must equal the reverse of its second half, so reversing only half the digits is sufficient.**

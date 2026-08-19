# LeetCode 32 — Longest Valid Parentheses

## Metadata

* **LeetCode:** 32
* **Problem:** Longest Valid Parentheses
* **Difficulty:** Hard
* **Topics:** String, Dynamic Programming, Stack
* **Pattern:** Balanced Parentheses, Matching Pairs
* **Key Technique:** Stack / Dynamic Programming / Two Pass
* **Key Pattern:** Prefix Balance + Boundary Tracking
* **Optimal Complexity:** `O(n)` time, `O(1)` space

---

## Problem

Given a string `s` containing only `'('` and `')'`, find the length of the **longest valid (well-formed) parentheses substring**.

A valid parentheses string must:

1. Have every opening parenthesis matched with a closing parenthesis.
2. Never have more closing parentheses than opening parentheses at any point.

### Examples

```text id="e3bq3x"
s = "(()"

Output = 2
```

The longest valid substring is:

```text
"()"
```

Another example:

```text id="x7pj2v"
s = ")()())"

Output = 4
```

The longest valid substring is:

```text
"()()"
```

---

# Approach 1 — Brute Force

## Idea

Check every possible substring and determine whether it is a valid parentheses string.

For a substring to be valid:

* The number of `(` must equal the number of `)`.
* While scanning from left to right, the number of `)` must never exceed the number of `(`.

This approach is straightforward but inefficient because there are `O(n²)` substrings.

## Dry Run

Consider:

```text id="v6cr0u"
s = "(()"
```

Possible substrings include:

```text
"("
"(("
"(()"
"()"
")"
```

Check:

```text
"("  → invalid
"((" → invalid
"(()" → invalid
"()" → valid → length 2
```

Therefore:

```text
answer = 2
```

## Algorithm

1. Generate every possible substring using two indices `i` and `j`.
2. For each substring:

   * Maintain a balance.
   * Increment balance for `(`.
   * Decrement balance for `)`.
3. If balance becomes negative, the substring is invalid.
4. If the final balance is `0`, the substring is valid.
5. Track the maximum valid length.

## Complexity

* **Time:** `O(n³)` with direct validation of every substring
* **Space:** `O(1)`

## Notes / Tips

* There are `O(n²)` substrings.
* Checking each substring can take `O(n)`.
* Therefore, this approach is too slow for large inputs.
* It is useful mainly for understanding the definition of a valid substring.

## Code

```cpp id="y9v3ke"
class Solution {
public:
    int longestValidParentheses(string s) {
        int n = s.size();
        int ans = 0;

        for (int i = 0; i < n; i++) {
            int balance = 0;

            for (int j = i; j < n; j++) {
                if (s[j] == '(') {
                    balance++;
                }
                else {
                    balance--;
                }

                if (balance < 0) {
                    break;
                }

                if (balance == 0) {
                    ans = max(ans, j - i + 1);
                }
            }
        }

        return ans;
    }
};
```

---

# Approach 2 — Stack

## Idea

Use a stack to keep track of indices of unmatched parentheses.

The stack stores **indices**, not just characters.

The key idea is to keep a boundary index representing the position immediately before the current possible valid substring.

Initialize:

```text
stack = [-1]
```

When we see:

### `'('`

Push its index onto the stack.

### `')'`

Pop the top element.

* If the stack becomes empty:

  * The current `)` cannot be matched.
  * Push its index as the new boundary.
* Otherwise:

  * The current valid substring has length:

    ```text
    i - stack.top()
    ```

## Dry Run

Consider:

```text id="h8v2qa"
s = ")()())"
```

Indices:

```text
0 1 2 3 4 5
) ( ) ( ) )
```

Start:

```text id="d7v5f2"
stack = [-1]
ans = 0
```

### i = 0 → `)`

Pop `-1`.

Stack becomes empty.

Push `0` as the new invalid boundary:

```text id="wz3z2x"
stack = [0]
ans = 0
```

### i = 1 → `(`

Push `1`:

```text id="6o5v8e"
stack = [0, 1]
```

### i = 2 → `)`

Pop `1`:

```text id="qg2w1u"
stack = [0]
```

Stack is not empty.

Valid length:

```text
2 - 0 = 2
```

So:

```text id="b5q2au"
ans = 2
```

The valid substring is:

```text
"()"
```

### i = 3 → `(`

Push `3`:

```text id="2f2p8k"
stack = [0, 3]
```

### i = 4 → `)`

Pop `3`:

```text id="o2q8mg"
stack = [0]
```

Valid length:

```text
4 - 0 = 4
```

So:

```text id="7h3gqk"
ans = 4
```

The valid substring is:

```text
"()()"
```

### i = 5 → `)`

Pop `0`.

Stack becomes empty.

Push `5` as the new boundary:

```text id="s8k5jd"
stack = [5]
```

Final answer:

```text id="v9r4au"
4
```

## Algorithm

1. Initialize a stack with `-1`.
2. Traverse the string from left to right.
3. If the current character is `'('`:

   * Push its index.
4. Otherwise:

   * Pop the top index.
   * If the stack becomes empty:

     * Push the current index as a new boundary.
   * Otherwise:

     * Calculate:

       ```text
       i - stack.top()
       ```
     * Update the maximum.
5. Return the maximum length.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(n)`

## Notes / Tips

* The stack stores **indices** because we need lengths.
* `-1` acts as a virtual boundary before the string.
* When an unmatched `)` is encountered, its index becomes the new boundary.
* The formula:

  ```text
  i - stack.top()
  ```

  gives the length of the current valid substring.
* This is one of the most intuitive solutions for the problem.

## Code

```cpp id="j4n2qs"
class Solution {
public:
    int longestValidParentheses(string s) {
        stack<int> st;
        st.push(-1);

        int ans = 0;

        for (int i = 0; i < s.size(); i++) {
            if (s[i] == '(') {
                st.push(i);
            }
            else {
                st.pop();

                if (st.empty()) {
                    st.push(i);
                }
                else {
                    ans = max(ans, i - st.top());
                }
            }
        }

        return ans;
    }
};
```

---

# Approach 3 — Dynamic Programming

## Idea

Let:

```text
dp[i] = length of the longest valid parentheses substring ending at index i
```

We only need to calculate `dp[i]` when:

```text
s[i] == ')'
```

There are two important cases.

### Case 1 — `s[i - 1] == '('`

We have:

```text
"...()"
```

So the current pair contributes `2`.

Therefore:

```text
dp[i] = dp[i - 2] + 2
```

Example:

```text id="g7a2ke"
s = "(())"
```

At `i = 3`:

```text
s[2] = '('
s[3] = ')'
```

The pair `()` contributes `2`, plus the valid substring before it.

---

### Case 2 — `s[i - 1] == ')'`

We may have:

```text
"...))"
```

The current `)` can match an earlier `(` if one exists.

The index of that potential matching `(` is:

```text
i - dp[i - 1] - 1
```

If that index is valid and contains `'('`, then:

```text
dp[i] = dp[i - 1] + 2
```

plus any valid substring immediately before that matching `(`:

```text
dp[i - dp[i - 1] - 2]
```

Therefore:

```text
dp[i] = dp[i - 1] + 2 + dp[i - dp[i - 1] - 2]
```

## Dry Run

Consider:

```text id="3y5e3f"
s = ")()())"
```

Indices:

```text
0 1 2 3 4 5
) ( ) ( ) )
```

Initialize:

```text id="l2j3w8"
dp = [0, 0, 0, 0, 0, 0]
ans = 0
```

### i = 2

`s[2] = ')'` and:

```text
s[1] == '('
```

Therefore:

```text
dp[2] = dp[0] + 2
      = 0 + 2
      = 2
```

```text id="7d3c3x"
dp = [0, 0, 2, 0, 0, 0]
```

### i = 4

`s[4] = ')'` and:

```text
s[3] == '('
```

Therefore:

```text
dp[4] = dp[2] + 2
      = 2 + 2
      = 4
```

```text id="k1x4d6"
dp = [0, 0, 2, 0, 4, 0]
```

### i = 5

`s[5] = ')'` and:

```text
s[4] == ')'
```

We check the possible matching `(`:

```text
i - dp[i - 1] - 1
= 5 - 4 - 1
= 0
```

But:

```text
s[0] == ')'
```

so there is no matching `(`.

Therefore:

```text
dp[5] = 0
```

Final:

```text
dp = [0, 0, 2, 0, 4, 0]
```

Answer:

```text
4
```

## Algorithm

1. Create a `dp` array of size `n`, initialized to `0`.
2. Traverse from `i = 1` to `n - 1`.
3. Only process positions where `s[i] == ')'`.
4. If `s[i - 1] == '('`:

   ```text
   dp[i] = dp[i - 2] + 2
   ```

   where `dp[i - 2]` is considered `0` if `i < 2`.
5. Otherwise, if `s[i - 1] == ')'`:

   * Calculate:

     ```text
     openIndex = i - dp[i - 1] - 1
     ```
   * If `openIndex >= 0` and `s[openIndex] == '('`:

     ```text
     dp[i] = dp[i - 1] + 2
     ```

     * If there is a valid substring before `openIndex`, add it:

       ```text
       dp[i] += dp[openIndex - 1]
       ```
6. Track the maximum value in `dp`.
7. Return the maximum.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(n)`

## Notes / Tips

* `dp[i]` represents a valid substring **ending exactly at `i`**.
* The DP solution is useful for understanding how valid substrings connect to previously solved substrings.
* Be careful with negative indices when accessing `dp[i - 2]` or `dp[openIndex - 1]`.

## Code

```cpp id="r8j3mt"
class Solution {
public:
    int longestValidParentheses(string s) {
        int n = s.size();

        if (n == 0) {
            return 0;
        }

        vector<int> dp(n, 0);
        int ans = 0;

        for (int i = 1; i < n; i++) {
            if (s[i] == ')') {
                // Case 1: "()"
                if (s[i - 1] == '(') {
                    dp[i] = 2;

                    if (i >= 2) {
                        dp[i] += dp[i - 2];
                    }
                }
                // Case 2: "(...))"
                else {
                    int openIndex = i - dp[i - 1] - 1;

                    if (openIndex >= 0 && s[openIndex] == '(') {
                        dp[i] = dp[i - 1] + 2;

                        if (openIndex >= 1) {
                            dp[i] += dp[openIndex - 1];
                        }
                    }
                }

                ans = max(ans, dp[i]);
            }
        }

        return ans;
    }
};
```

---

# Approach 4 — Two-Pass Counting / Optimal

## Idea

We can solve the problem using only two counters:

```text
left  = number of '('
right = number of ')'
```

Scan from **left to right**.

Whenever:

```text
left == right
```

we have a balanced valid substring, so its length is:

```text
2 * right
```

However, there is an important problem.

Consider:

```text
"(()"
```

From left to right:

```text
left = 2
right = 1
```

The counts never become equal, so we would miss `"()"`.

To handle this, perform a second scan from **right to left**.

The two scans handle the two possible imbalance directions.

## Dry Run

Consider:

```text id="d8x0zm"
s = ")()())"
```

### Left → Right

Initialize:

```text id="x3q6a1"
left = 0
right = 0
ans = 0
```

Scan:

```text
) → left=0, right=1
```

Since:

```text
right > left
```

the current substring cannot become valid from this point, so reset:

```text
left = 0
right = 0
```

Next:

```text
( → left=1, right=0
) → left=1, right=1
```

Counts are equal:

```text
length = 2
ans = 2
```

Next:

```text
( → left=2, right=1
) → left=2, right=2
```

Counts are equal:

```text
length = 4
ans = 4
```

Next:

```text
) → left=2, right=3
```

Again:

```text
right > left
```

Reset.

Left-to-right result:

```text
ans = 4
```

For some strings, the left-to-right scan alone misses valid suffixes such as:

```text
"(()"
```

So we perform the reverse scan.

### Right → Left

Now count:

```text
left = 0
right = 0
```

For:

```text
"(()"
```

from right to left:

```text
) → right=1
( → left=1
( → left=2
```

At the end:

```text
left > right
```

The reverse scan handles this type of imbalance and finds the valid `"()"`.

## Algorithm

### Left-to-Right Pass

1. Set `left = 0` and `right = 0`.
2. Traverse from left to right.
3. For `(`, increment `left`.
4. For `)`, increment `right`.
5. If:

   ```text
   left == right
   ```

   update:

   ```text
   ans = max(ans, 2 * right)
   ```
6. If:

   ```text
   right > left
   ```

   reset both counters to `0`.

### Right-to-Left Pass

1. Reset `left` and `right`.
2. Traverse from right to left.
3. For `(`, increment `left`.
4. For `)`, increment `right`.
5. If:

   ```text
   left == right
   ```

   update the answer.
6. If:

   ```text
   left > right
   ```

   reset both counters to `0`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

## Notes / Tips

* This is the **optimal space solution**.
* Why two passes?

  * Left-to-right handles cases where there are too many `)`.
  * Right-to-left handles cases where there are too many `(`.
* Simply checking `left == right` in one direction is **not enough**.
* This technique works specifically because the string contains only parentheses.
* This is often the best solution to remember when the interviewer asks for `O(1)` extra space.

## Code

```cpp id="w4q6nz"
class Solution {
public:
    int longestValidParentheses(string s) {
        int ans = 0;
        int left = 0;
        int right = 0;

        // Left to right
        for (char ch : s) {
            if (ch == '(') {
                left++;
            }
            else {
                right++;
            }

            if (left == right) {
                ans = max(ans, 2 * right);
            }
            else if (right > left) {
                left = 0;
                right = 0;
            }
        }

        // Right to left
        left = 0;
        right = 0;

        for (int i = s.size() - 1; i >= 0; i--) {
            if (s[i] == '(') {
                left++;
            }
            else {
                right++;
            }

            if (left == right) {
                ans = max(ans, 2 * left);
            }
            else if (left > right) {
                left = 0;
                right = 0;
            }
        }

        return ans;
    }
};
```

---

# Comparison of Approaches

| Approach    |    Time |  Space | Main Idea                            |
| ----------- | ------: | -----: | ------------------------------------ |
| Brute Force | `O(n³)` | `O(1)` | Check every substring                |
| Stack       |  `O(n)` | `O(n)` | Store unmatched indices              |
| DP          |  `O(n)` | `O(n)` | `dp[i]` = valid length ending at `i` |
| Two-Pass    |  `O(n)` | `O(1)` | Count parentheses in both directions |

---

# Key Takeaway

There are **two important patterns** to remember.

### Stack Pattern

```text
'(' → push index

')' → pop

if stack becomes empty:
    current index becomes boundary
else:
    valid length = current index - stack.top()
```

The key template is:

```cpp
stack<int> st;
st.push(-1);

for (int i = 0; i < s.size(); i++) {
    if (s[i] == '(') {
        st.push(i);
    }
    else {
        st.pop();

        if (st.empty()) {
            st.push(i);
        }
        else {
            ans = max(ans, i - st.top());
        }
    }
}
```

### Optimal `O(1)` Pattern

```text
Left → Right
    ↓
Too many ')' → reset

Right → Left
    ↓
Too many '(' → reset
```

The most important thing to remember:

> **One-direction counting is not enough. You need both directions to catch both types of imbalance.**

**Best approaches to remember:**

* **Stack:** easiest to reason about and very reusable for parentheses problems.
* **Two-pass counting:** optimal `O(n)` time and `O(1)` space.

**Key Pattern:** Balanced Parentheses + Boundary Tracking.

**Mental Template:**

```text
Longest Valid Parentheses
        ↓
Track balance
        ↓
Invalid imbalance → reset
        ↓
Balanced → calculate length
        ↓
Need O(1) space?
        ↓
Scan both directions
```

# LeetCode 150 — Evaluate Reverse Polish Notation

## Metadata

- **LeetCode:** 150
- **Problem:** Evaluate Reverse Polish Notation
- **Difficulty:** Medium
- **Topics:** Array, Math, Stack
- **Pattern:** Stack Evaluation
- **Key Technique:** Push operands, and on an operator pop the last two, apply it, push the result back

---

# Approaches

1. **Brute Force — Repeated List Scan + In-Place Reduction**
2. **Optimal — Single-Pass Stack Evaluation**

---

# Approach 1 — Brute Force / Repeated List Scan + In-Place Reduction

## Idea

Repeatedly scan the token list to find the **first** operator. Evaluate it using the two tokens immediately before it, replace those three tokens with the single result, and scan again from the start. Keep doing this until only one token remains.

## Dry Run

```text
tokens = ["2", "1", "+", "3", "*"]
```

Scan 1 — first operator found is `"+"` at index 2:
```text
2 + 1 = 3
tokens = ["3", "3", "*"]
```

Scan 2 — first operator found is `"*"` at index 2:
```text
3 * 3 = 9
tokens = ["9"]
```

Only one token left:
```text
result = 9
```

## Algorithm

1. While the token list has more than one element:
   - Scan from the start to find the first operator (`+`, `-`, `*`, `/`).
   - Take the two tokens immediately before it as operands.
   - Compute the result and replace those three tokens with the single result token.
2. Return the remaining token as an integer.

## Complexity

- **Time:** `O(n^2)` — each reduction step requires a fresh scan, and list shrinks/shifts each time
- **Space:** `O(n)` — for the mutable token list

## Notes / Tips

- Correct but wasteful — repeatedly rescanning and shifting list elements is unnecessary.
- Demonstrates why a stack is a natural fit here: operands always sit immediately before their operator once RPN order is respected.

## Code

```cpp
class Solution {
public:
    int evalRPN(vector<string>& tokens) {
        vector<string> list = tokens;

        while (list.size() > 1) {
            int opIndex = -1;
            for (int i = 0; i < list.size(); i++) {
                if (list[i] == "+" || list[i] == "-" || list[i] == "*" || list[i] == "/") {
                    opIndex = i;
                    break;
                }
            }

            int a = stoi(list[opIndex - 2]);
            int b = stoi(list[opIndex - 1]);
            string op = list[opIndex];
            int result;

            if (op == "+") {
                result = a + b;
            }
            else if (op == "-") {
                result = a - b;
            }
            else if (op == "*") {
                result = a * b;
            }
            else {
                result = a / b;
            }

            list.erase(list.begin() + opIndex - 2, list.begin() + opIndex + 1);
            list.insert(list.begin() + opIndex - 2, to_string(result));
        }

        return stoi(list[0]);
    }
};
```

---

# Approach 2 — Optimal / Single-Pass Stack Evaluation

## Idea

RPN is designed for stack evaluation: scan tokens left to right, push numbers onto a stack. When an operator is seen, pop the top two values (the second-to-last is the left operand, the last is the right operand), apply the operator, and push the result back.

## Dry Run

```text
tokens = ["4", "13", "5", "/", "+"]
```

Process:
```text
"4"  → push → stack = [4]
"13" → push → stack = [4, 13]
"5"  → push → stack = [4, 13, 5]
"/"  → pop 5, pop 13 → 13 / 5 = 2 → push → stack = [4, 2]
"+"  → pop 2, pop 4 → 4 + 2 = 6 → push → stack = [6]
```

Final:
```text
stack = [6] → result = 6
```

## Algorithm

1. Initialize an empty stack.
2. For each token:
   - If it's a number, push it onto the stack.
   - If it's an operator, pop the top two values `b` (last pushed) and `a` (second-to-last), apply `a op b`, push the result.
3. Return the single remaining value on the stack.

## Complexity

- **Time:** `O(n)` — each token processed once
- **Space:** `O(n)` — for the stack

## Notes / Tips

- Operand order matters for `-` and `/`: pop `b` first, then `a`, and compute `a op b`, not `b op a`.
- Integer division in RPN problems truncates toward zero — in C++, plain `/` on `int` already does this correctly.
- This is the canonical stack-evaluation pattern — same idea applies to any postfix/prefix expression evaluator or a basic calculator that needs to respect operator precedence.

## Code

```cpp
class Solution {
public:
    int evalRPN(vector<string>& tokens) {
        stack<int> st;

        for (string& token : tokens) {
            if (token == "+" || token == "-" || token == "*" || token == "/") {
                int b = st.top(); st.pop();
                int a = st.top(); st.pop();

                if (token == "+") st.push(a + b);
                else if (token == "-") {
                    st.push(a - b);
                }
                else if (token == "*") {
                    st.push(a * b);
                }
                else {
                    st.push(a / b);
                }
            } else {
                st.push(stoi(token));
            }
        }

        return st.top();
    }
};
```

---

# Key Template

### Postfix Expression Evaluation via Stack

```text
stack = []

for token in tokens:
    if token is operator:
        b = stack.pop()
        a = stack.pop()
        stack.push(apply(op, a, b))
    else:
        stack.push(number(token))

return stack.pop()
```

## Pattern Recognition

When you see:

```text
Postfix / Reverse Polish notation
+
Expression evaluation
```

Think:

```text
Operands always precede their operator
        ↓
Push numbers, pop two + apply on operator, push result back
```

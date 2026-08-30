# LeetCode 682 — Baseball Game

## Metadata

- **LeetCode:** 682
- **Problem:** Baseball Game
- **Difficulty:** Easy
- **Topics:** Array, String, Stack, Simulation
- **Pattern:** Stack Simulation
- **Key Technique:** Push valid scores, pop/peek for "C"/"D"/"+" based on the most recent record(s)

---

# Approaches

1. **Brute Force — Array with Manual Index Tracking**
2. **Optimal — Stack Simulation**

---

# Approach 1 — Brute Force / Array with Manual Index Tracking

## Idea

Use a dynamic array and a pointer to the current "top" index. For each operation, look back at the array using the pointer to find the needed previous score(s), then either append a new score or remove the last one by decrementing the pointer.

## Dry Run

```text
ops = ["5", "2", "C", "D", "+"]
```

Process:
```text
"5" → append → record = [5], top = 0
"2" → append → record = [5, 2], top = 1
"C" → remove last → record = [5], top = 0
"D" → look at record[top]=5, new score = 10 → append → record = [5, 10], top = 1
"+" → look at record[top]=10, record[top-1]=5, new score = 15 → append → record = [5, 10, 15], top = 2
```

Sum:
```text
5 + 10 + 15 = 30
```

## Algorithm

1. Initialize an empty array `record` and `top = -1`.
2. For each operation `op`:
   - If `op` is a number, append it, increment `top`.
   - If `op == "C"`, remove the last element, decrement `top`.
   - If `op == "D"`, compute `2 * record[top]`, append, increment `top`.
   - If `op == "+"`, compute `record[top] + record[top-1]`, append, increment `top`.
3. Sum all elements in `record` and return.

## Complexity

- **Time:** `O(n)` — one pass, each operation `O(1)` amortized
- **Space:** `O(n)` — for the record array

## Notes / Tips

- Functionally identical to a stack — this version just manages the "top" manually instead of using stack push/pop.
- Slightly more error-prone since it's easy to mismanage the `top` pointer or forget to decrement it on `"C"`.

## Code

```cpp
class Solution {
public:
    int calPoints(vector<string>& ops) {
        vector<int> record;
        int top = -1;

        for (string& op : ops) {
            if (op == "C") {
                record.pop_back();
                top--;
            } else if (op == "D") {
                int val = 2 * record[top];
                record.push_back(val);
                top++;
            } else if (op == "+") {
                int val = record[top] + record[top - 1];
                record.push_back(val);
                top++;
            } else {
                record.push_back(stoi(op));
                top++;
            }
        }

        int sum = 0;
        for (int score : record) {
            sum += score;
        }

        return sum;
    }
};
```

---

# Approach 2 — Optimal / Stack Simulation

## Idea

This problem is a direct stack simulation: each operation only ever needs the most recent one or two scores, which is exactly what a stack's `top()` gives for free. Use built-in stack operations instead of manual index tracking.

## Dry Run

```text
ops = ["5", "-2", "4", "C", "D", "9", "+", "+"]
```

Process:
```text
"5"  → push → stack = [5]
"-2" → push → stack = [5, -2]
"4"  → push → stack = [5, -2, 4]
"C"  → pop → stack = [5, -2]
"D"  → top=-2, 2*-2=-4 → push → stack = [5, -2, -4]
"9"  → push → stack = [5, -2, -4, 9]
"+"  → top=9, second=-4, 9+(-4)=5 → push → stack = [5, -2, -4, 9, 5]
"+"  → top=5, second=9, 5+9=14 → push → stack = [5, -2, -4, 9, 5, 14]
```

Sum:
```text
5 + (-2) + (-4) + 9 + 5 + 14 = 27
```

## Algorithm

1. Initialize an empty stack.
2. For each operation `op`:
   - If `op == "C"`, pop the stack.
   - If `op == "D"`, push `2 * stack.top()`.
   - If `op == "+"`, let `a = stack.top()`, pop, `b = stack.top()`, push `b`, push `a`, push `a + b` (or peek both without popping, depending on implementation).
   - Otherwise, push the parsed integer.
3. Sum all values in the stack and return.

## Complexity

- **Time:** `O(n)` — one pass, `O(1)` per operation
- **Space:** `O(n)` — for the stack

## Notes / Tips

- For `"+"`, don't actually remove the two previous scores — they still count individually toward the final sum, only the new sum is added on top.
- This is a good "template" easy problem for stack simulation: recognize that any op needing "the last k items" maps directly to `top()`/`pop()`/`push()`.
- Summing while popping everything at the end (instead of keeping a running total) also works and avoids a second pass — either is fine at this scale.

## Code

```cpp
class Solution {
public:
    int calPoints(vector<string>& ops) {
        stack<int> st;

        for (string& op : ops) {
            if (op == "C") {
                st.pop();
            } else if (op == "D") {
                st.push(2 * st.top());
            } else if (op == "+") {
                int a = st.top();
                st.pop();
                int b = st.top();
                st.push(a);
                st.push(a + b);
            } else {
                st.push(stoi(op));
            }
        }

        int sum = 0;
        while (!st.empty()) {
            sum += st.top();
            st.pop();
        }

        return sum;
    }
};
```

---

# Key Template

### Stack Simulation for "Last K Records" Operations

```text
stack = []

for op in ops:
    if op == "C": stack.pop()
    elif op == "D": stack.push(2 * stack.top())
    elif op == "+":
        a = stack.pop()
        b = stack.top()
        stack.push(a)
        stack.push(a + b)
    else: stack.push(int(op))

return sum(stack)
```

## Pattern Recognition

When you see:

```text
Sequence of operations
+
Each op depends only on the last 1-2 results
+
Undo / recompute based on recent history
```

Think:

```text
Manual array + index tracking
        ↓
Optimal: plain stack, top()/pop()/push() map directly to each operation
```

The key observation is:

> **Any operation that only looks at the most recent one or two entries is a stack problem — no need to track indices manually.**
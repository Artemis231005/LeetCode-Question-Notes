# LeetCode 155 — Min Stack

## Metadata

* **LeetCode:** 155
* **Problem:** Min Stack
* **Difficulty:** Medium
* **Topics:** Stack, Design
* **Pattern:** Auxiliary Stack
* **Key Technique:** Maintain minimum at every stack state
* **Key Pattern:** Two Stacks
* **Key Template:** Main Stack + Min Stack
* **Optimal Complexity:** `O(1)` per operation

---

## Problem

Design a stack that supports the following operations in **`O(1)` time**:

```text
push(val)
pop()
top()
getMin()
```

`getMin()` should return the minimum element currently present in the stack.

Example:

```text
push(-2)
push(0)
push(-3)

getMin() → -3

pop()

top() → 0

getMin() → -2
```

---

## Idea

A normal stack can give us:

```text
top()
push()
pop()
```

in `O(1)`.

But finding the minimum by scanning the entire stack would take:

```text
O(n)
```

for every `getMin()`.

So maintain a **second stack** called `minStack`.

### Main Stack

Stores all elements normally.

### Min Stack

Stores the minimum value corresponding to each stack state.

For every `push`:

```text
minStack.push(min(val, currentMin))
```

This means the top of `minStack` always contains the minimum of the entire main stack.

---

## Dry Run

Perform:

```text
push(-2)
push(0)
push(-3)
```

### Push `-2`

Main stack:

```text
[-2]
```

Min stack:

```text
[-2]
```

Minimum:

```text
-2
```

### Push `0`

Minimum remains `-2`.

```text
Main: [-2, 0]
Min:  [-2, -2]
```

### Push `-3`

New minimum is `-3`.

```text
Main: [-2, 0, -3]
Min:  [-2, -2, -3]
```

Therefore:

```text
getMin() → -3
```

### Pop

Remove `-3` from both stacks:

```text
Main: [-2, 0]
Min:  [-2, -2]
```

Now:

```text
getMin() → -2
```

The previous minimum is automatically restored.

---

## Algorithm

### `push(val)`

1. Push `val` into the main stack.
2. If `minStack` is empty, push `val`.
3. Otherwise push:

   ```text
   min(val, minStack.top())
   ```

### `pop()`

1. Pop from the main stack.
2. Pop from the min stack.

### `top()`

Return:

```text
mainStack.top()
```

### `getMin()`

Return:

```text
minStack.top()
```

---

## Complexity

| Operation  |   Time |  Space |
| ---------- | -----: | -----: |
| `push()`   | `O(1)` | `O(n)` |
| `pop()`    | `O(1)` | `O(n)` |
| `top()`    | `O(1)` | `O(n)` |
| `getMin()` | `O(1)` | `O(n)` |

Overall:

* **Time:** `O(1)` per operation
* **Space:** `O(n)`

---

## Notes / Tips

* The key idea is to **store the minimum at every level of the stack**.
* Both stacks always have the same number of elements.
* The top of `minStack` is always the current minimum.
* When the minimum is popped, the previous minimum automatically becomes available.
* Duplicate minimum values are handled correctly.

### Example With Duplicate Minimum

```text
push(2)
push(1)
push(1)
```

Stacks:

```text
Main: [2, 1, 1]
Min:  [2, 1, 1]
```

After one `pop()`:

```text
Main: [2, 1]
Min:  [2, 1]
```

Minimum is still:

```text
1
```

This is why we should push the current minimum **even when it is equal** to the new value.

### Common Mistake

Do not store a new minimum only when:

```cpp
val < minStack.top()
```

because duplicate minimum values would be lost.

Instead:

```cpp
minStack.push(min(val, minStack.top()));
```

---

## Code

```cpp
class MinStack {
public:
    stack<int> st;
    stack<int> minSt;

    MinStack() {
    }

    void push(int val) {
        st.push(val);

        if (minSt.empty()) {
            minSt.push(val);
        } else {
            minSt.push(min(val, minSt.top()));
        }
    }

    void pop() {
        st.pop();
        minSt.pop();
    }

    int top() {
        return st.top();
    }

    int getMin() {
        return minSt.top();
    }
};
```

---

## Basic Template

```cpp
class MinStack {
public:
    stack<int> st;
    stack<int> minSt;

    void push(int val) {
        st.push(val);

        if (minSt.empty()) {
            minSt.push(val);
        } else {
            minSt.push(min(val, minSt.top()));
        }
    }

    void pop() {
        st.pop();
        minSt.pop();
    }

    int top() {
        return st.top();
    }

    int getMin() {
        return minSt.top();
    }
};
```

### Reusable Pattern

```text
Main Stack              Min Stack

  val                      min
   ↓                        ↓
push(val)          push(min(val, previousMin))
   ↓                        ↓
pop()              pop()
   ↓                        ↓
top()              getMin()
```

### Core Idea

```text
Every position in minStack
        ↓
stores the minimum
        ↓
of all elements up to that position
```

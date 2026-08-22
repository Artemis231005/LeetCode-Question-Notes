# 232. Implement Queue using Stacks

## Metadata

* **Topic:** Queue, Stack
* **Difficulty:** Easy
* **Pattern:** Two Stacks
* **Key Pattern:** Use one stack for input and one for output to simulate FIFO.

---

## Idea

A **queue** follows:

```text
FIFO → First In, First Out
```

A **stack** follows:

```text
LIFO → Last In, First Out
```

Use two stacks:

* `in` → stores newly pushed elements.
* `out` → provides elements for `pop()` and `peek()`.

When `out` is empty, move everything from `in` to `out`.

Example:

```text
in:  [1, 2, 3]
out: []
```

Transfer:

```text
in:  []
out: [3, 2, 1]
```

Now `1` is at the top of `out`, so it behaves like a queue.

---

## Dry Run

Operations:

```text
push(1)
push(2)
push(3)
pop()
pop()
```

After pushes:

```text
in  = [1,2,3]
out = []
```

First `pop()`:

```text
in  = []
out = [3,2,1]
```

Pop `1`.

Second `pop()`:

```text
out = [3,2]
```

Pop `2`.

So the order is:

```text
1 → 2 → 3
```

which is FIFO.

---

## Algorithm

### Push

Push the element into `in`.

### Pop

1. If `out` is empty, transfer all elements from `in` to `out`.
2. Pop from `out`.

### Peek

1. If `out` is empty, transfer all elements from `in` to `out`.
2. Return `out.top()`.

### Empty

Return whether both stacks are empty.

---

## Complexity

* **Push:** `O(1)`
* **Pop:** `O(1)` amortized
* **Peek:** `O(1)` amortized
* **Empty:** `O(1)`
* **Space:** `O(n)`

The transfer can take `O(n)` for one operation, but each element is transferred from `in` to `out` only once before being removed, giving **O(1) amortized** time.

---

## Code

```cpp
class MyQueue {
public:
    stack<int> in;
    stack<int> out;

    void push(int x) {
        in.push(x);
    }

    int pop() {
        shift();

        int value = out.top();
        out.pop();

        return value;
    }

    int peek() {
        shift();

        return out.top();
    }

    bool empty() {
        return in.empty() && out.empty();
    }

private:
    void shift() {
        if (out.empty()) {
            while (!in.empty()) {
                out.push(in.top());
                in.pop();
            }
        }
    }
};
```

---

## Notes / Tips

* `in` handles **pushes**.
* `out` handles **pops/peeks**.
* Only transfer when `out` is empty.
* Do **not** transfer every time; that would make operations unnecessarily expensive.
* This is an example of **amortized analysis**.
* Each element moves:

  ```text
  in → out → removed
  ```

  only once.

---

## Key Template

```text
push(x):
    in.push(x)

pop/peek:
    if out is empty:
        move all in → out

    use out.top()
```

Remember:

```text
Stack  → LIFO
Queue  → FIFO

Two stacks:
in → push side
out → pop/peek side
```

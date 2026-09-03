# LeetCode 24 — Swap Nodes in Pairs

## Metadata

* **LeetCode:** 24
* **Problem:** Swap Nodes in Pairs
* **Difficulty:** Medium
* **Topics:** Linked List, Recursion
* **Pattern:** Pairwise Pointer Manipulation
* **Key Technique:** Use a dummy node and rewire three pointers per pair so the swap works in-place without extra nodes
* **Optimal Complexity:** `O(n)` Time, `O(1)` Space

---

## Approaches

1. **Brute Force — Extract Values into an Array, Swap, Rewrite**
2. **Better — Recursive Swap**
3. **Optimal — Iterative Swap with Dummy Node**

---

# Approach 1 — Brute Force / Extract Values, Swap, Rewrite

## Idea

Traverse the list once and collect all node values into an array. Swap adjacent values in the array in pairs. Traverse the list a second time and overwrite each node's value from the array.

## Dry Run

```text
list = 1 -> 2 -> 3 -> 4
```

Extract:
```text
values = [1, 2, 3, 4]
```

Swap adjacent pairs:
```text
values = [2, 1, 4, 3]
```

Rewrite nodes:
```text
1 -> 2, 2 -> 1, 3 -> 4, 4 -> 3
```

Result:
```text
2 -> 1 -> 4 -> 3
```

## Algorithm

1. Traverse the list, storing all values in an array.
2. Swap every adjacent pair of values in the array.
3. Traverse the list again, writing values back from the array in order.
4. Return the head.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(n)`
  * For the values array.

## Notes / Tips

* Doesn't actually swap nodes — just swaps values, which technically violates the spirit of the problem (some variants explicitly disallow modifying node values).
* Simple to write but not the intended solution.

## Code

```cpp
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        vector<int> values;
        ListNode* curr = head;

        while (curr) {
            values.push_back(curr->val);
            curr = curr->next;
        }

        for (int i = 0; i + 1 < values.size(); i += 2) {
            swap(values[i], values[i + 1]);
        }

        curr = head;
        int i = 0;
        while (curr) {
            curr->val = values[i++];
            curr = curr->next;
        }

        return head;
    }
};
```

---

# Approach 2 — Better / Recursive Swap

## Idea

Swap the first two nodes, then recursively solve the rest of the list starting from the third node, and attach the recursive result to the back of the swapped pair.

## Dry Run

```text
list = 1 -> 2 -> 3 -> 4
```

Call `swapPairs(1)`:
```text
first = 1, second = 2
first->next = swapPairs(3)
```

Call `swapPairs(3)`:
```text
first = 3, second = 4
first->next = swapPairs(nullptr) = nullptr
second->next = first → 4 -> 3
return 4 (new head of this sub-list)
```

Back in outer call:
```text
first->next = 4 -> 3   (so 1 -> 4 -> 3)
second->next = first  → 2 -> 1 -> 4 -> 3
return 2 (new head)
```

Result:
```text
2 -> 1 -> 4 -> 3
```

## Algorithm

1. If `head` is `null` or `head->next` is `null`, return `head` (0 or 1 node, nothing to swap).
2. Let `first = head`, `second = head->next`.
3. Recursively call `swapPairs(second->next)` and assign it to `first->next`.
4. Set `second->next = first`.
5. Return `second` as the new head of this pair.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(n)`
  * For the recursion call stack (`n/2` recursive calls deep).

## Notes / Tips

* Cleaner to write than the iterative version since each call only handles one pair and trusts recursion for the rest.
* Recursion depth is `n/2`, which can risk a stack overflow on very long lists.

## Code

```cpp
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        if (!head || !head->next) {
            return head;
        }

        ListNode* first = head;
        ListNode* second = head->next;

        first->next = swapPairs(second->next);
        second->next = first;

        return second;
    }
};
```

---

# Approach 3 — Optimal / Iterative Swap with Dummy Node

## Idea

Use a dummy node pointing to `head` so the very first pair can be swapped without special-casing the head pointer. Keep a `prev` pointer that always points to the node just before the current pair, and rewire three links per pair: `prev`, `first`, and `second`.

## Dry Run

```text
list = 1 -> 2 -> 3 -> 4
dummy -> 1 -> 2 -> 3 -> 4
prev = dummy
```

### Pair 1: (1, 2)

```text
first = 1, second = 2
first->next = second->next → 1->next = 3
second->next = first → 2->next = 1
prev->next = second → dummy->next = 2
```

State:
```text
dummy -> 2 -> 1 -> 3 -> 4
prev = first = 1
```

### Pair 2: (3, 4)

```text
first = 3, second = 4
first->next = second->next → 3->next = null
second->next = first → 4->next = 3
prev->next = second → 1->next = 4
```

State:
```text
dummy -> 2 -> 1 -> 4 -> 3
```

`prev = first = 3`, `prev->next = null` → loop ends.

Result:
```text
2 -> 1 -> 4 -> 3
```

## Algorithm

1. Create a `dummy` node with `dummy->next = head`.
2. Set `prev = dummy`.
3. While `prev->next` and `prev->next->next` both exist:

   * `first = prev->next`, `second = first->next`.
   * `first->next = second->next`.
   * `second->next = first`.
   * `prev->next = second`.
   * `prev = first` (move `prev` to the end of the just-swapped pair).
4. Return `dummy->next`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

## Notes / Tips

* The dummy node removes the need to special-case swapping the first pair (which would otherwise require updating the external `head` reference).
* Order of the three pointer rewires matters — `first->next` must be reassigned before `second->next`, otherwise the link to the rest of the list is lost.
* This is the standard **dummy node + pointer rewiring** template used across most in-place linked list manipulation problems (reversing, reordering, etc.).

## Code

```cpp
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        ListNode* dummy = new ListNode(0);
        dummy->next = head;
        ListNode* prev = dummy;

        while (prev->next && prev->next->next) {
            ListNode* first = prev->next;
            ListNode* second = first->next;

            first->next = second->next;
            second->next = first;
            prev->next = second;

            prev = first;
        }

        return dummy->next;
    }
};
```

---

## Key Template

```text
dummy -> head
prev = dummy

while prev.next and prev.next.next:
    first = prev.next
    second = first.next

    first.next = second.next
    second.next = first
    prev.next = second

    prev = first

return dummy.next
```
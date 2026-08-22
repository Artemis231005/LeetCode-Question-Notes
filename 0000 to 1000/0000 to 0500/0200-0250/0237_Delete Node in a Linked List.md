# 237. Delete Node in a Linked List

## Metadata

* **Topic:** Linked List
* **Difficulty:** Medium
* **Pattern:** Node Value Copying
* **Key Pattern:** Copy the next node's value into the current node, then skip the next node.

---

## Idea

Normally, to delete a node from a linked list, we need access to its **previous node**.

But here, we are only given the node that needs to be deleted, not the head.

Example:

```text
4 → 5 → 1 → 9
    ↑
  delete
```

We cannot directly change the previous node (`4`).

Instead:

1. Copy the next node's value into the current node.
2. Skip the next node.

```text
4 → 1 → 9
```

For deleting `5`:

```text
node->val = node->next->val
node->next = node->next->next
```

---

## Dry Run

```text
4 → 5 → 1 → 9
    ↑
  node
```

Copy next value:

```text
4 → 1 → 1 → 9
    ↑
  node
```

Skip the next node:

```text
4 → 1 → 9
```

So `5` is effectively deleted.

---

## Algorithm

1. Copy the value of `node->next` into `node`.
2. Make `node->next` point to `node->next->next`.
3. The original next node is no longer part of the list.

---

## Complexity

* **Time:** `O(1)`
* **Space:** `O(1)`

---

## Code

```cpp
class Solution {
public:
    void deleteNode(ListNode* node) {
        node->val = node->next->val;
        node->next = node->next->next;
    }
};
```

---

## Notes / Tips

* We are **not actually deleting the given node**.
* We make it take the identity/value of the next node and remove the next node instead.
* This works because the problem guarantees the given node is **not the tail**.
* If the node were the tail, there would be no `node->next` to copy from.

### Key Trick

```text
node->val = node->next->val
node->next = node->next->next
```

Think:

```text
Copy next → Skip next
```

---

## Key Template

```text
given node ≠ tail

node->val = node->next->val
node->next = node->next->next
```

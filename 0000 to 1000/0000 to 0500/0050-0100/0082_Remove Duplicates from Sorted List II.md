# LeetCode 82 — Remove Duplicates from Sorted List II

## Metadata

* **LeetCode:** 82
* **Problem:** Remove Duplicates from Sorted List II
* **Difficulty:** Medium
* **Topics:** Linked List, Two Pointers
* **Pattern:** Dummy Node + Lookahead Skip
* **Key Technique:** Use a dummy node and a `prev` pointer that only advances past a run of values when that run has no duplicates
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given the head of a sorted linked list, delete all nodes that have duplicate values, leaving only distinct values from the original list. Return the modified list's head.

---

## Approaches

1. **Brute Force — Count Occurrences, Then Rebuild**
2. **Optimal — Dummy Node + Lookahead Skip**

---

# Approach 1 — Brute Force / Count Occurrences, Then Rebuild

## Idea

Traverse the list once and count how many times each value appears (using a hash map, since values could repeat non-adjacently in general, though here the list is sorted so duplicates are adjacent). Traverse again, building a new list that only includes nodes whose value has a count of exactly `1`.

## Dry Run

```text
list = 1 -> 2 -> 3 -> 3 -> 4 -> 4 -> 5
```

Count occurrences:

```text
1:1, 2:1, 3:2, 4:2, 5:1
```

Rebuild, keeping only count-1 values:

```text
1 -> 2 -> 5
```

## Algorithm

1. Traverse the list once, storing each value's frequency in a hash map.
2. Create a dummy node and a `tail` pointer for building the result.
3. Traverse the list again:

   * If the current node's value has frequency `1`, append a new node (or the same node) with that value to the result.
4. Return `dummy.next`.

## Complexity

* **Time:** `O(n)`

  * Two linear passes: one to count, one to rebuild.
* **Space:** `O(n)`

  * For the hash map storing value frequencies.

## Notes / Tips

* The hash map is overkill here since the list is already sorted — duplicates are always adjacent, so a single pass with lookahead (Approach 2) can do the same job without extra memory.
* Still useful conceptually if the list weren't sorted, though this specific problem guarantees sorted input.

## Code

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        unordered_map<int, int> freq;
        ListNode* curr = head;

        while (curr) {
            freq[curr->val]++;
            curr = curr->next;
        }

        ListNode* dummy = new ListNode(0);
        ListNode* tail = dummy;
        curr = head;

        while (curr) {
            if (freq[curr->val] == 1) {
                tail->next = new ListNode(curr->val);
                tail = tail->next;
            }
            curr = curr->next;
        }

        return dummy->next;
    }
};
```

---

# Approach 2 — Optimal / Dummy Node + Lookahead Skip

## Idea

Since the list is sorted, all duplicates of a value sit consecutively. Use a dummy node before `head` so the very first node can be removed safely if needed. Keep a `prev` pointer that always points to the last confirmed-unique node. At each step, look ahead: if the current node's value matches the next node's value, skip the entire run of matching nodes without advancing `prev`; otherwise, advance `prev` to include this node.

## Dry Run

```text
list = 1 -> 2 -> 3 -> 3 -> 4 -> 4 -> 5
dummy -> 1 -> 2 -> 3 -> 3 -> 4 -> 4 -> 5
prev = dummy, curr = 1
```

### curr = 1

```text
1->next = 2, values differ → not a duplicate run
prev = curr (1), curr = curr->next (2)
```

### curr = 2

```text
2->next = 3, values differ → not a duplicate run
prev = curr (2), curr = curr->next (3)
```

### curr = 3

```text
3->next = 3, values match → duplicate run detected
skip all nodes with value 3:
   curr = curr->next → second 3
   curr = curr->next → 4
prev->next = curr → 2->next = 4 (skips both 3's)
curr = 4 (do not advance prev)
```

### curr = 4

```text
4->next = 4, values match → duplicate run detected
skip all nodes with value 4:
   curr = curr->next → second 4
   curr = curr->next → 5
prev->next = curr → 2->next = 5 (skips both 4's)
curr = 5
```

### curr = 5

```text
5->next = null → not a duplicate run
prev = curr (5), curr = curr->next (null)
```

Loop ends.

Result:

```text
dummy -> 1 -> 2 -> 5
```

## Algorithm

1. Create a `dummy` node with `dummy->next = head`.
2. Set `prev = dummy`, `curr = head`.
3. While `curr` is not null:

   * If `curr->next` exists and `curr->next->val == curr->val`:

     * Skip forward through `curr` while the next node has the same value.
     * Set `prev->next = curr->next` (removes the entire duplicate run).
     * Advance `curr = curr->next` (do not move `prev`).
   * Else:

     * Advance `prev = curr`.
     * Advance `curr = curr->next`.
4. Return `dummy->next`.

## Complexity

* **Time:** `O(n)`

  * Each node is visited a constant number of times across the traversal and duplicate-skipping steps.
* **Space:** `O(1)`

  * Only a constant number of pointers used — no hash map or extra data structures.

## Notes / Tips

* The dummy node is essential here since the very first node(s) in the list could themselves be duplicates that need removing — without it, updating the external `head` reference would need special-casing.
* `prev` only advances when the current node is confirmed to have no duplicates — this is what correctly excludes an entire run rather than leaving one copy behind (unlike LC 83, which keeps one copy of each duplicate value instead of removing all of them).
* Relies entirely on the list being sorted — if it weren't, duplicates could be scattered non-adjacently and this lookahead technique wouldn't catch them all.

## Code

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode* dummy = new ListNode(0);
        dummy->next = head;
        ListNode* prev = dummy;
        ListNode* curr = head;

        while (curr) {
            if (curr->next && curr->next->val == curr->val) {
                int dupVal = curr->val;
                while (curr && curr->val == dupVal) {
                    curr = curr->next;
                }
                prev->next = curr;
            } else {
                prev = curr;
                curr = curr->next;
            }
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
curr = head

while curr:
    if curr.next and curr.next.val == curr.val:
        dupVal = curr.val
        while curr and curr.val == dupVal:
            curr = curr.next
        prev.next = curr
    else:
        prev = curr
        curr = curr.next

return dummy.next
```
# LeetCode 2 — Add Two Numbers

## Metadata

* **LeetCode:** 2
* **Problem:** Add Two Numbers
* **Difficulty:** Medium
* **Topics:** Linked List, Math, Recursion
* **Pattern:** Simulated Digit-by-Digit Addition with Carry
* **Key Technique:** Since digits are stored in reverse order (least significant first), traverse both lists simultaneously like elementary-school column addition, carrying over into the next position as needed
* **Optimal Complexity:** `O(max(n, m))` Time, `O(max(n, m))` Auxiliary Space (for the output list)

---

## Problem Statement

Given two non-empty linked lists representing two non-negative integers, where digits are stored in **reverse order** (each node contains a single digit), add the two numbers and return the sum as a linked list, also in reverse order.

---

## Approaches

1. **Brute Force — Convert to Integers, Add, Convert Back**
2. **Optimal — Simulated Digit-by-Digit Addition with Carry**

---

# Approach 1 — Brute Force / Convert to Integers, Add, Convert Back

## Idea

Traverse each list to reconstruct the full number it represents (reading digits from least to most significant, so building the number by multiplying by increasing powers of 10 as you go). Add the two resulting numbers directly, then convert the sum back into a linked list of digits in reverse order.

## Dry Run

```text
l1 = 2 -> 4 -> 3   (represents 342)
l2 = 5 -> 6 -> 4   (represents 465)
```

Reconstruct `l1`:

```text
2 + 4*10 + 3*100 = 342
```

Reconstruct `l2`:

```text
5 + 6*10 + 4*100 = 465
```

Add:

```text
342 + 465 = 807
```

Convert back to reversed digit list:

```text
807 → digits reversed: 7 -> 0 -> 8
```

## Algorithm

1. Traverse `l1`, accumulating `num1 = num1 + digit * placeValue`, incrementing `placeValue` by a factor of `10` each step.
2. Repeat the same for `l2` to get `num2`.
3. Compute `sum = num1 + num2`.
4. Build a new linked list from `sum`'s digits, least significant first (peeling off digits via `% 10` and `/ 10`, same as reversing an integer).
5. Return the new list's head.

## Complexity

* **Time:** `O(n + m)`

  * One pass over each list to reconstruct the numbers, plus a pass over the sum's digits to build the result.
* **Space:** `O(n + m)`

  * For the resulting linked list; the reconstructed integers themselves take `O(1)` extra space assuming fixed-width arithmetic (though see Notes below).

## Notes / Tips

* This approach silently breaks for very large numbers. Since the linked lists can represent arbitrarily long numbers, reconstructing them as native integers risks overflow well before reaching the list's actual length limits (a fixed-width `int` or even `long long` can't hold an arbitrarily long digit sequence).
* This is a real correctness risk, not just a performance concern — unlike most "brute force vs optimal" trade-offs in these notes, this approach can produce **wrong answers** on large inputs, which is exactly why the digit-by-digit simulation in Approach 2 is the actual intended solution, not just a faster alternative.

## Code

```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        // WARNING: overflows for very long lists — shown for comparison only.
        long long num1 = 0, place1 = 1;
        while (l1) {
            num1 += l1->val * place1;
            place1 *= 10;
            l1 = l1->next;
        }

        long long num2 = 0, place2 = 1;
        while (l2) {
            num2 += l2->val * place2;
            place2 *= 10;
            l2 = l2->next;
        }

        long long sum = num1 + num2;

        if (sum == 0) {
            return new ListNode(0);
        }

        ListNode* dummy = new ListNode(0);
        ListNode* tail = dummy;

        while (sum > 0) {
            tail->next = new ListNode(sum % 10);
            tail = tail->next;
            sum /= 10;
        }

        return dummy->next;
    }
};
```

---

# Approach 2 — Optimal / Simulated Digit-by-Digit Addition with Carry

## Idea

Since both lists already store digits least-significant-first, add them the same way addition is done by hand on paper: walk both lists simultaneously, summing corresponding digits plus any carry from the previous position, writing down `sum % 10` as the current result digit and carrying `sum / 10` into the next position. Continue until both lists (and any final carry) are exhausted.

## Dry Run

```text
l1 = 2 -> 4 -> 3   (342)
l2 = 5 -> 6 -> 4   (465)
```

Process:

```text
position 0: 2 + 5 + carry(0) = 7 → digit=7, carry=0
position 1: 4 + 6 + carry(0) = 10 → digit=0, carry=1
position 2: 3 + 4 + carry(1) = 8 → digit=8, carry=0
```

Both lists exhausted, no final carry → result: `7 -> 0 -> 8` (represents 807).

### Example with a final carry-out

```text
l1 = 9 -> 9   (99)
l2 = 1        (1)
```

```text
position 0: 9 + 1 + carry(0) = 10 → digit=0, carry=1
position 1: 9 + 0 (l2 exhausted) + carry(1) = 10 → digit=0, carry=1
position 2: 0 (both exhausted) + carry(1) = 1 → digit=1, carry=0
```

Result: `0 -> 0 -> 1` (represents 100).

## Algorithm

1. Create a `dummy` node and a `tail` pointer for building the result.
2. Initialize `carry = 0`.
3. While `l1` is not null, or `l2` is not null, or `carry != 0`:

   * `val1 = l1 ? l1->val : 0`, `val2 = l2 ? l2->val : 0`.
   * `sum = val1 + val2 + carry`.
   * `carry = sum / 10`.
   * Append a new node with value `sum % 10` to the result list.
   * Advance `l1` and `l2` if they're not null.
4. Return `dummy->next`.

## Complexity

* **Time:** `O(max(n, m))`

  * Both lists are traversed simultaneously, stopping once the longer one (plus any final carry) is exhausted.
* **Space:** `O(max(n, m))`

  * For the newly built result list — unavoidable since the output itself must be returned as a linked list of comparable length.

## Notes / Tips

* Continuing the loop while `carry != 0` (even after both lists are exhausted) is what correctly handles a final carry-out digit (e.g. `99 + 1 = 100`, which has one more digit than either input) — forgetting this condition is a common off-by-one bug in this problem.
* Using `val1 = l1 ? l1->val : 0` naturally handles lists of different lengths without needing separate logic once one list runs out — the shorter list simply contributes `0` for its missing positions.
* This digit-by-digit-with-carry simulation is the correct and standard approach for this problem specifically because the numbers can be arbitrarily long — it never risks overflow the way reconstructing full integers (Approach 1) does, since only single digits and a small carry value are ever handled at once.

## Code

```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode* dummy = new ListNode(0);
        ListNode* tail = dummy;
        int carry = 0;

        while (l1 || l2 || carry) {
            int val1 = l1 ? l1->val : 0;
            int val2 = l2 ? l2->val : 0;

            int sum = val1 + val2 + carry;
            carry = sum / 10;

            tail->next = new ListNode(sum % 10);
            tail = tail->next;

            if (l1) l1 = l1->next;
            if (l2) l2 = l2->next;
        }

        return dummy->next;
    }
};
```

---

## Key Template

```text
dummy = new ListNode(0)
tail = dummy
carry = 0

while l1 or l2 or carry:
    val1 = l1.val if l1 else 0
    val2 = l2.val if l2 else 0

    sum = val1 + val2 + carry
    carry = sum / 10

    tail.next = new ListNode(sum % 10)
    tail = tail.next

    if l1: l1 = l1.next
    if l2: l2 = l2.next

return dummy.next
```
# LeetCode 881 — Boats to Save People

## Metadata

* **LeetCode:** 881
* **Problem:** Boats to Save People
* **Difficulty:** Medium
* **Topics:** Array, Two Pointers, Greedy, Sorting
* **Pattern:** Sort + Two Pointers (Lightest with Heaviest)
* **Key Technique:** Sort by weight, then always try to pair the lightest remaining person with the heaviest remaining person in one boat — if they don't fit together, the heaviest goes alone
* **Optimal Complexity:** `O(n log n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given an array `people` where `people[i]` is a person's weight, and an integer `limit` (the maximum weight a single boat can carry, at most 2 people per boat), return the minimum number of boats needed to carry everyone across.

---

## Approaches

1. **Brute Force — Try Every Way to Pair People**
2. **Optimal — Sort + Two Pointers (Lightest with Heaviest)**

---

# Approach 1 — Brute Force / Try Every Way to Pair People

## Idea

Try every possible way to group people into boats of at most 2, where each boat's total weight doesn't exceed `limit`, and track the minimum number of boats across all valid groupings using backtracking.

## Dry Run

```text
people = [3, 2, 2, 1], limit = 3
```

Try pairing `1` with `2` (first `2`):

```text
boat: [1,2] (sum=3, fits)
remaining: [3, 2]
```

Try pairing `3` alone (can't pair with remaining `2`, since `3+2=5>3`):

```text
boat: [3]
remaining: [2]
```

Last person `2` alone:

```text
boat: [2]
```

Total boats: `3`. Try other pairings (e.g. `1` with the other `2`, or `1` alone) to see if fewer boats are possible — all valid groupings here end up needing `3` boats.

## Algorithm

1. Define a recursive `minBoats(remainingPeople)`:

   * If `remainingPeople` is empty, return `0`.
   * Try leaving the first remaining person alone in a boat: recurse on the rest, add `1`.
   * Try pairing the first remaining person with every other remaining person whose combined weight fits within `limit`: recurse on the rest (excluding both), add `1`.
   * Return the minimum across all these choices.
2. Return `minBoats(people)`.

## Complexity

* **Time:** `O(2^n)` (roughly, from exploring pairing/not-pairing choices for every person)

  * Backtracking through all possible groupings has exponential branching in the worst case.
* **Space:** `O(n)`

  * For the recursion stack.

## Notes / Tips

* Trying every possible pairing combination is unnecessary. Sorting first and always attempting to pair the lightest remaining person with the heaviest remaining person (Approach 2) is provably optimal, removing any need to explore alternative pairings.
* This exhaustive search is mainly useful for confirming the greedy strategy's correctness on small examples, not as a practical solution.

## Code

```cpp
class Solution {
public:
    int minBoats(vector<int> remaining, int limit) {
        if (remaining.empty()) {
            return 0;
        }

        int first = remaining[0];
        vector<int> rest(remaining.begin() + 1, remaining.end());

        // Option 1: first person goes alone
        int best = 1 + minBoats(rest, limit);

        // Option 2: pair first person with someone else who fits
        for (int i = 0; i < (int)rest.size(); i++) {
            if (first + rest[i] <= limit) {
                vector<int> pairedRest;
                for (int j = 0; j < (int)rest.size(); j++) {
                    if (j != i) pairedRest.push_back(rest[j]);
                }
                best = min(best, 1 + minBoats(pairedRest, limit));
            }
        }

        return best;
    }

    int numRescueBoats(vector<int>& people, int limit) {
        return minBoats(people, limit);
    }
};
```

---

# Approach 2 — Optimal / Sort + Two Pointers (Lightest with Heaviest)

## Idea

Sort everyone by weight. Use two pointers: `left` at the lightest remaining person, `right` at the heaviest remaining person. Always try to put the heaviest person on a boat together with the lightest remaining person — if their combined weight fits within `limit`, both go together (advance both pointers); if not, the heaviest person must go alone (advance only `right`). This greedy pairing is optimal because the heaviest remaining person can only ever be paired with someone, and if anyone can share a boat with them, it should be the *lightest* available person (since that's the "easiest" pairing to make work, freeing up all other lighter people to potentially pair with other heavy people later).

## Dry Run

```text
people = [3, 2, 2, 1], limit = 3
```

Sort: `[1, 2, 2, 3]`.

`left = 0` (value `1`), `right = 3` (value `3`):

```text
1 + 3 = 4 > 3 → doesn't fit → 3 goes alone
boats = 1, right-- → right=2
```

`left = 0` (value `1`), `right = 2` (value `2`):

```text
1 + 2 = 3 <= 3 → fits → both go together
boats = 2, left++ → left=1, right-- → right=1
```

`left = 1` (value `2`), `right = 1` (value `2`): `left == right`, only one person left:

```text
2 goes alone
boats = 3
```

`left` and `right` cross → stop.

Final: `3` boats, matching the brute-force result.

## Algorithm

1. Sort `people` in ascending order.
2. Initialize `left = 0`, `right = n - 1`, `boats = 0`.
3. While `left <= right`:

   * If `people[left] + people[right] <= limit` and `left != right`: both go together — increment `left`.
   * Decrement `right` (the heaviest remaining person always leaves, whether paired or alone).
   * Increment `boats`.
4. Return `boats`.

## Complexity

* **Time:** `O(n log n)`

  * Dominated by sorting; the two-pointer scan afterward is `O(n)`.
* **Space:** `O(1)` auxiliary (beyond the sort's own space)

  * Only two pointers and a boat counter — no extra structures.

## Notes / Tips

* The greedy correctness argument: the heaviest remaining person must go on *some* boat — either alone, or paired with exactly one other person. If they can be paired with anyone, pairing them with the **lightest** remaining person is always at least as good as pairing them with anyone heavier, since it "wastes" the least capacity and preserves every heavier person's chance to also find a partner later.
* The condition `left != right` guards against double-counting a single remaining person as both the lightest and heaviest simultaneously when only one person is left — without it, `boats` could be incorrectly incremented an extra time, or the same person could be considered "paired with themselves."
* This greedy two-pointer-after-sorting approach is a common shape for "pair items from two ends to satisfy a capacity constraint" problems, though the greedy *reasoning* here is closer to interval/resource-allocation greedy problems.

## Code

```cpp
class Solution {
public:
    int numRescueBoats(vector<int>& people, int limit) {
        sort(people.begin(), people.end());
        int left = 0, right = people.size() - 1;
        int boats = 0;

        while (left <= right) {
            if (left != right && people[left] + people[right] <= limit) {
                left++;
            }
            right--;
            boats++;
        }

        return boats;
    }
};
```

---

## Key Template

```text
sort(people)
left = 0, right = n - 1
boats = 0

while left <= right:
    if left != right and people[left] + people[right] <= limit:
        left += 1
    right -= 1
    boats += 1

return boats
```
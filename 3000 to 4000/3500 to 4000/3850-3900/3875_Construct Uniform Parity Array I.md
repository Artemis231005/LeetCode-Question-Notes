# LeetCode 3875 — Construct Uniform Parity Array I

## Metadata

* **LeetCode:** 3875
* **Problem:** Construct Uniform Parity Array I
* **Difficulty:** Easy
* **Topics:** Array, Math
* **Pattern:** Parity Case Analysis
* **Key Technique:** Odd minus even (or even minus odd) is always odd, so mixed-parity arrays can always be forced into all-odd
* **Optimal Complexity:** `O(1)` Time, `O(1)` Space

---

## Problem Statement

Given a distinct-integer array `nums1` of length `n`, build `nums2` of the same length where every `nums2[i]` is either `nums1[i]` or `nums1[i] - nums1[j]` for some `j != i`. Return whether it's possible to make every element of `nums2` share the same parity (all odd or all even).

---

## Approaches

1. **Brute Force — Try Both Target Parities Explicitly**
2. **Optimal — Direct Parity Construction**

---

# Approach 1 — Brute Force / Try Both Target Parities Explicitly

## Idea

Try to mechanically build `nums2` for each of the two possible targets ("all even" and "all odd"). For a given target, check every index `i`: is `nums1[i]` already the right parity? If yes, keep it. If not, search for any `j != i` such that `nums1[i] - nums1[j]` has the target parity. If every index can be satisfied for either target, return `true`.

## Dry Run

```text
nums1 = [2, 3]
```

### Try target = even

```text
i=0: nums1[0]=2 → already even → ok
i=1: nums1[1]=3 → odd, need j with (3 - nums1[j]) even
     j=0: 3 - 2 = 1 → odd → fails
     no valid j → target "even" fails
```

### Try target = odd

```text
i=0: nums1[0]=2 → even, need j with (2 - nums1[j]) odd
     j=1: 2 - 3 = -1 → odd → ok
i=1: nums1[1]=3 → already odd → ok
```

Both indices satisfied → target "odd" succeeds → return `true`.

## Algorithm

1. For each `target` in `{even, odd}`:

   * For each index `i`:

     * If `nums1[i]` already matches `target`, continue.
     * Else, scan all `j != i` for one where `nums1[i] - nums1[j]` matches `target`.
     * If none found, this target fails — break out.
   * If all indices succeeded, return `true`.
2. If neither target works, return `false`.

## Complexity

* **Time:** `O(n²)`

  * For each of 2 targets, each index may scan all other `n-1` indices looking for a valid `j`.
* **Space:** `O(1)`

## Notes / Tips

* This explicit search always ends up returning `true` — useful mainly to double check the reasoning used in Approach 2 before trusting the shortcut.
* Re-derives, index by index, something that already holds for the whole array at once.

## Code

```cpp
class Solution {
public:
    bool uniformArray(vector<int>& nums1) {
        int n = nums1.size();

        for (int target = 0; target <= 1; target++) {
            bool ok = true;

            for (int i = 0; i < n && ok; i++) {
                if (nums1[i] % 2 == target) {
                    continue;
                }

                bool found = false;
                for (int j = 0; j < n; j++) {
                    if (j == i) continue;
                    if (abs(nums1[i] - nums1[j]) % 2 == target) {
                        found = true;
                        break;
                    }
                }

                if (!found) {
                    ok = false;
                }
            }

            if (ok) {
                return true;
            }
        }

        return false;
    }
};
```

---

# Approach 2 — Optimal / Direct Parity Construction

## Idea

If every element of `nums1` already shares one parity, just copy `nums1` directly into `nums2` — done.

If `nums1` has a mix of odd and even values, then for every index `i`, pick any `j` from the opposite-parity group. `odd - even` and `even - odd` are both always odd. So setting `nums2[i] = nums1[i] - nums1[j]` for every index makes the entire array odd.

Either way — uniform parity already, or forcibly made uniform via subtraction — a valid `nums2` always exists.

## Dry Run

```text
nums1 = [4, 6]
```

All elements already even → copy directly:

```text
nums2 = [4, 6] → valid, all even
```

```text
nums1 = [2, 3]
```

Mixed parity → subtract opposite-parity element from every index:

```text
nums2[0] = 2 - 3 = -1 (odd)
nums2[1] = 3 - 2 = 1  (odd)
```

Both odd → valid.

## Algorithm

1. No casework actually needs to run at input-check time — a valid `nums2` exists for any input, as shown above.
2. Return `true`.

## Complexity

* **Time:** `O(1)`

  * No loop or scan needed — the answer doesn't depend on the input at all.
* **Space:** `O(1)`

## Notes / Tips

* Worth double-checking edge case `n = 1`: no valid `j` exists, but a single element is trivially "uniform parity" on its own, so `nums2[0] = nums1[0]` and it still works.
* The elaborate "choose `nums1[i]` or `nums1[i] - nums1[j]`" framing hides that the answer never depends on the actual values — only the case split (uniform vs. mixed) matters, and both cases are always constructible.

## Code

```cpp
class Solution {
public:
    bool uniformArray(vector<int>& nums1) {
        return true;
    }
};
```

---

## Key Template

```text
# Whenever a construction problem's condition reduces to
# a parity/invariant argument that holds for every case:

1. Check the "no change needed" case (already uniform).
2. Check the "forced" case (combine opposite groups to flip everything to one parity).
3. If both cases are always constructible, the answer is a constant — no computation needed.
```
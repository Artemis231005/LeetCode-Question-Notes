# LeetCode 2932 — Maximum Strong Pair XOR I

## Metadata

* **LeetCode:** 2932
* **Problem:** Maximum Strong Pair XOR I
* **Difficulty:** Easy
* **Topics:** Array, Bit Manipulation, Trie, Sliding Window, Sorting
* **Pattern:** Sort + Sliding Window + Binary Trie (Maximum XOR Query)
* **Key Technique:** After sorting, a pair `(x, y)` with `x <= y` is strong exactly when `y <= 2x`, so the valid partners of each number form a contiguous window — maintain that window in a binary trie and query the maximum XOR against it
* **Optimal Complexity:** `O(n log n + n * B)` Time, `O(n * B)` Space (`B` = number of bits)

---

## Problem Statement

Given an integer array `nums`, a pair `(x, y)` is a **strong pair** if `|x - y| <= min(x, y)`. Return the maximum `x XOR y` over all strong pairs. The same element may be used twice to form a pair.

---

## Approaches

1. **Brute Force — Check Every Pair**
2. **Optimal — Sort + Sliding Window + Binary Trie**

---

# Approach 1 — Brute Force / Check Every Pair

## Idea

Try every pair of elements, including pairing an element with itself. For each pair, test the strong pair condition and, if it holds, compare its XOR against the best seen so far.

## Dry Run

```text
nums = [5, 6, 25, 30]
```

```text
(5,5):   |0| <= 5 ✓   xor = 0
(5,6):   |1| <= 5 ✓   xor = 3  → best = 3
(5,25):  |20| <= 5 ✗
(5,30):  |25| <= 5 ✗
(6,25):  |19| <= 6 ✗
(6,30):  |24| <= 6 ✗
(25,25): ✓             xor = 0
(25,30): |5| <= 25 ✓   xor = 7  → best = 7
(30,30): ✓             xor = 0
```

Answer: `7`.

## Algorithm

1. Initialize `best = 0`.
2. For every pair of indices `(i, j)` with `i <= j`:

   * If `abs(nums[i] - nums[j]) <= min(nums[i], nums[j])`, update `best = max(best, nums[i] ^ nums[j])`.
3. Return `best`.

## Complexity

* **Time:** `O(n²)`

  * Every pair of elements is checked once.
* **Space:** `O(1)`

  * Only a running maximum.

## Notes / Tips

* With `n <= 50`, this is already fast enough and is the expected solution for this version of the problem.
* `best` can safely start at `0`, since pairing any element with itself always gives a valid strong pair with XOR `0`.
* For `x <= y` the condition simplifies to `y - x <= x`, i.e. `y <= 2x`, which is the property the optimal approach builds on.

## Code

```cpp
class Solution {
public:
    int maximumStrongPairXor(vector<int>& nums) {
        int n = nums.size();
        int best = 0;

        for (int i = 0; i < n; i++) {
            for (int j = i; j < n; j++) {
                if (abs(nums[i] - nums[j]) <= min(nums[i], nums[j])) {
                    best = max(best, nums[i] ^ nums[j]);
                }
            }
        }

        return best;
    }
};
```

---

# Approach 2 — Optimal / Sort + Sliding Window + Binary Trie

## Idea

Sort the array. For sorted values with `x <= y`, the pair is strong exactly when `y <= 2x`. So for each right-end value `nums[right]`, its valid partners are the elements from some `left` up to `right`, where `left` only ever moves forward as `right` increases — a sliding window.

Keep the numbers in the current window in a **binary trie** (with counts so elements can be removed when they leave the window). For each `right`, insert `nums[right]`, shrink the window from the left while `nums[right] > 2 * nums[left]`, then ask the trie for the maximum XOR of `nums[right]` against everything currently in the window. At each bit from high to low, the query prefers the child with the opposite bit, since that sets the bit in the XOR.

## Dry Run

```text
nums = [1, 2, 3, 4, 5]   (already sorted)
```

```text
right=0 (1): window {1}
   query 1 → 1^1 = 0 → best = 0

right=1 (2): 2 <= 2*1 ✓ → window {1,2}
   query 2 → max(2^1=3, 2^2=0) = 3 → best = 3

right=2 (3): 3 > 2*1 → remove 1 → window {2,3}   (3 <= 2*2 ✓)
   query 3 → max(3^2=1, 3^3=0) = 1 → best stays 3

right=3 (4): 4 <= 2*2 ✓ → window {2,3,4}
   query 4 → max(4^2=6, 4^3=7, 4^4=0) = 7 → best = 7

right=4 (5): 5 > 2*2 → remove 2 → window {3,4,5}   (5 <= 2*3 ✓)
   query 5 → max(5^3=6, 5^4=1, 5^5=0) = 6 → best stays 7
```

Answer: `7`, matching the brute force.

## Algorithm

1. Sort `nums`.
2. Initialize an empty binary trie (each node stores two children and a count), `left = 0`, `best = 0`.
3. For each `right` from `0` to `n - 1`:

   * Insert `nums[right]` into the trie (increment counts along its bit path).
   * While `nums[right] > 2 * nums[left]`: remove `nums[left]` from the trie (decrement counts) and increment `left`.
   * Query the trie with `nums[right]`: for each bit from high to low, go to the child with the opposite bit if that child exists with count `> 0` (and set that bit in the result); otherwise go to the same-bit child.
   * Update `best = max(best, queryResult)`.
4. Return `best`.

## Complexity

* **Time:** `O(n log n + n * B)`

  * Sorting is `O(n log n)`; each element is inserted, removed and queried once, each taking `O(B)` where `B` is the number of bits (7 here, since values are at most 100).
* **Space:** `O(n * B)`

  * For the trie nodes.

## Notes / Tips

* Counts on trie nodes are what make deletion possible: a branch is treated as present only while its count is positive, so removing an element never requires physically deleting nodes.
* The window is always non-empty (it contains `nums[right]` itself), so the query always has at least one valid path to follow. This also covers the self-pair case, which gives XOR `0`.
* For this problem's tiny constraints the trie is unnecessary, but it is the scalable solution — the harder version (LC 2935) has `n` up to `5 * 10^4` and values up to `2^20`, where `O(n²)` fails. For that version, set `BITS` to `20`.
* This is the standard maximum XOR pair technique (binary trie with greedy opposite-bit choice) combined with a sliding window to restrict which elements are eligible partners.

## Code

```cpp
class Solution {
    static const int BITS = 7; // nums[i] <= 100 < 128 (use 20 for LC 2935)
    vector<array<int, 2>> nxt;
    vector<int> cnt;

    int newNode() {
        nxt.push_back({-1, -1});
        cnt.push_back(0);
        return (int)nxt.size() - 1;
    }

    void update(int x, int delta) {
        int node = 0;
        for (int b = BITS - 1; b >= 0; b--) {
            int bit = (x >> b) & 1;
            if (nxt[node][bit] == -1) {
                nxt[node][bit] = newNode();
            }
            node = nxt[node][bit];
            cnt[node] += delta;
        }
    }

    int query(int x) {
        int node = 0, result = 0;
        for (int b = BITS - 1; b >= 0; b--) {
            int bit = (x >> b) & 1;
            int want = bit ^ 1;
            int child = nxt[node][want];

            if (child != -1 && cnt[child] > 0) {
                result |= (1 << b);
                node = child;
            } else {
                node = nxt[node][bit];
            }
        }
        return result;
    }

public:
    int maximumStrongPairXor(vector<int>& nums) {
        sort(nums.begin(), nums.end());

        nxt.clear();
        cnt.clear();
        newNode(); // root

        int left = 0, best = 0;

        for (int right = 0; right < (int)nums.size(); right++) {
            update(nums[right], +1);

            while (nums[right] > 2 * nums[left]) {
                update(nums[left], -1);
                left++;
            }

            best = max(best, query(nums[right]));
        }

        return best;
    }
};
```

---

## Key Template

```text
sort(nums)
trie = empty (counts on nodes)
left = 0
best = 0

for right in 0..n-1:
    trie.insert(nums[right])

    while nums[right] > 2 * nums[left]:
        trie.remove(nums[left])
        left += 1

    best = max(best, trie.maxXor(nums[right]))

return best
```
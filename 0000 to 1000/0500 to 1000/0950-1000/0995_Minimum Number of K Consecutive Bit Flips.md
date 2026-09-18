# LeetCode 995 — Minimum Number of K Consecutive Bit Flips

## Metadata

* **LeetCode:** 995
* **Problem:** Minimum Number of K Consecutive Bit Flips
* **Difficulty:** Hard
* **Topics:** Array, Bit Manipulation, Sliding Window, Queue, Prefix Sum
* **Pattern:** Greedy Left-to-Right + Difference Array (Parity Tracking)
* **Key Technique:** Process indices left to right; whenever the current bit (after accounting for all flips affecting it so far) is `0`, it must be flipped now, since nothing to its left can ever help it again — track the running "flip parity" with a difference array instead of literally re-flipping bits
* **Optimal Complexity:** `O(n)` Time, `O(n)` Auxiliary Space

---

## Problem Statement

Given a binary array `nums` and an integer `k`, in one operation you choose a subarray of size exactly `k` and flip every bit in it (`0` becomes `1`, `1` becomes `0`). Return the minimum number of such operations needed to make every element `1`, or `-1` if it's impossible.

---

## Approaches

1. **Brute Force — Literal Simulation, Flipping Windows Directly**
2. **Optimal — Greedy Left-to-Right with Difference Array Parity**

---

# Approach 1 — Brute Force / Literal Simulation, Flipping Windows Directly

## Idea

Simulate the process exactly as described: scan for the leftmost `0`, since it can only ever be fixed by a window starting exactly there (nothing to its left can flip it again once passed). Flip that entire size-`k` window directly in the array, count the operation, and repeat until either no zeros remain or a needed window would run off the end.

## Dry Run

```text
nums = [0, 1, 0], k = 1
```

Leftmost `0`: index `0`. Window `[0,0]` (size 1), flip:

```text
[1, 1, 0]
```

Leftmost `0`: index `2`. Window `[2,2]`, flip:

```text
[1, 1, 1]
```

No zeros remain → return flip count `2`.

## Algorithm

1. Initialize `flips = 0`.
2. While any element of `nums` is `0`:

   * Find the leftmost index `i` with `nums[i] == 0`.
   * If `i + k > n`, return `-1` (can't form a valid window here).
   * Flip every bit in `nums[i .. i+k-1]`.
   * Increment `flips`.
3. Return `flips`.

## Complexity

* **Time:** `O(n * k)`

  * In the worst case, up to `n` flip operations are performed, each directly touching up to `k` elements (plus the cost of rescanning for the next leftmost zero each time).
* **Space:** `O(1)`

  * Modifies the array in place, only a few index variables tracked.

## Notes / Tips

* Literally flipping every bit in the window on every operation is the main cost here — since only whether a bit's *cumulative* flip count is even or odd actually matters (not the literal bit value at every intermediate step), tracking flip parity instead of mutating the array directly removes this cost entirely.
* This brute force is useful to confirm the greedy window-selection rule (always start at the leftmost unfixed `0`) before optimizing how the flips themselves are tracked and applied.

## Code

```cpp
class Solution {
public:
    int minKBitFlips(vector<int>& nums, int k) {
        int n = nums.size();
        int flips = 0;

        while (true) {
            int i = -1;
            for (int idx = 0; idx < n; idx++) {
                if (nums[idx] == 0) {
                    i = idx;
                    break;
                }
            }

            if (i == -1) {
                return flips;
            }

            if (i + k > n) {
                return -1;
            }

            for (int j = i; j < i + k; j++) {
                nums[j] ^= 1;
            }

            flips++;
        }
    }
};
```

---

# Approach 2 — Optimal / Greedy Left-to-Right with Difference Array Parity

## Idea

Scan left to right, tracking the **parity** of how many flip operations currently affect each index using a difference array (same range-update technique as LC 1109, LC 2848, and LC 2772 — but here it's toggling a bit's state rather than summing a numeric amount). At each index `i`, compute the bit's effective current value: `nums[i]` XOR-ed with the running flip parity. If that's `0`, a flip must start exactly at `i` (greedy, and forced — nothing earlier can help anymore). If `i + k > n`, it's impossible. Otherwise, mark a flip starting at `i` in the difference array and continue.

## Dry Run

```text
nums = [0, 0, 0, 1, 0, 1, 1, 0], k = 3
```

Initialize `diff` array of size `n+1`, `flipParity = 0`, `flips = 0`.

```text
i=0: flipParity += diff[0]=0 → parity=0
     effective = nums[0] ^ 0 = 0 → must flip
     i+k=3 <= n=8 → apply: diff[0]+=1, diff[3]-=1
     flipParity += 1 (this flip) → parity=1
     flips=1

i=1: flipParity += diff[1]=0 → parity=1
     effective = nums[1] ^ 1 = 0^1 = 1 → already good, no flip

i=2: flipParity += diff[2]=0 → parity=1
     effective = nums[2] ^ 1 = 0^1 = 1 → good

i=3: flipParity += diff[3]=-1 → parity=0
     effective = nums[3] ^ 0 = 1^0 = 1 → good

i=4: flipParity += diff[4]=0 → parity=0
     effective = nums[4] ^ 0 = 0 → must flip
     i+k=7 <= 8 → apply: diff[4]+=1, diff[7]-=1
     flipParity += 1 → parity=1
     flips=2

i=5: flipParity += diff[5]=0 → parity=1
     effective = nums[5] ^ 1 = 1^1 = 0 → must flip
     i+k=8 <= 8 → apply: diff[5]+=1, diff[8]-=1
     flipParity += 1 → parity=2 (even, i.e. 0 effectively — track as count, check even/odd)
     flips=3

i=6: flipParity += diff[6]=0 → parity=2 (even)
     effective = nums[6] ^ (parity%2=0) = 1^0 = 1 → good

i=7: flipParity += diff[7]=-1 → parity=1
     effective = nums[7] ^ (parity%2=1) = 0^1 = 1 → good
```

All positions satisfied, no impossible case hit → return `flips = 3`.

## Algorithm

1. Create a difference array `diff` of size `n + 1`, all zeros.
2. Initialize `flipCount = 0` (running count of flips currently affecting the index, used via its parity) and `totalFlips = 0`.
3. For each index `i` from `0` to `n-1`:

   * `flipCount += diff[i]`.
   * Compute `effective = nums[i] ^ (flipCount % 2)` (parity of flips applied so far, XOR-ed with the original bit).
   * If `effective == 0`:

     * If `i + k > n`, return `-1`.
     * Apply the flip: `diff[i] += 1`, `diff[i + k] -= 1`.
     * `flipCount += 1`.
     * Increment `totalFlips`.
4. Return `totalFlips`.

## Complexity

* **Time:** `O(n)`

  * A single left-to-right pass, `O(1)` work per index.
* **Space:** `O(n)`

  * For the difference array (this can be reduced to `O(k)` by tracking flip parity using a sliding window of only the last `k` flip decisions, since flips more than `k` positions behind the current index can no longer affect it).

## Notes / Tips

* This is structurally the same greedy + difference array template as LC 2772 (Apply Operations to Make All Array Elements Equal to Zero) — the only real differences are that this problem tracks flip **parity** (even/odd via XOR) instead of a raw decrement amount, and it needs to *count* operations rather than just check feasibility.
* The greedy choice is forced, not just convenient: since scanning proceeds strictly left to right and a fixed bit is never revisited, any bit still `0` at index `i` can *only* be fixed by a flip window starting exactly at `i` — starting later would skip it, and starting earlier would already have been applied by a previous iteration.
* Using `flipCount % 2` (rather than tracking the literal flip count) is what avoids ever needing to know the exact number of overlapping flips at a position — only whether that count is even or odd determines the bit's current effective value.

## Code

```cpp
class Solution {
public:
    int minKBitFlips(vector<int>& nums, int k) {
        int n = nums.size();
        vector<int> diff(n + 1, 0);
        int flipCount = 0, totalFlips = 0;

        for (int i = 0; i < n; i++) {
            flipCount += diff[i];
            int effective = nums[i] ^ (flipCount % 2);

            if (effective == 0) {
                if (i + k > n) {
                    return -1;
                }

                diff[i] += 1;
                diff[i + k] -= 1;
                flipCount += 1;
                totalFlips++;
            }
        }

        return totalFlips;
    }
};
```

---

## Key Template

```text
diff = array of size (n + 1), all 0
flipCount = 0
totalFlips = 0

for i in 0..n-1:
    flipCount += diff[i]
    effective = nums[i] xor (flipCount % 2)

    if effective == 0:
        if i + k > n:
            return -1

        diff[i] += 1
        diff[i + k] -= 1
        flipCount += 1
        totalFlips += 1

return totalFlips
```
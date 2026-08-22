# 209. Minimum Size Subarray Sum

## Metadata

* **Topic:** Array, Sliding Window
* **Difficulty:** Medium
* **Pattern:** Variable-Size Sliding Window
* **Key Pattern:** Expand until `sum >= target`, then shrink to find the minimum window.

---

## Idea

We need the **minimum length of a contiguous subarray** whose sum is at least `target`.

Since all numbers are **positive**, we can use a sliding window.

Maintain:

```text
left = 0
sum = 0
```

Expand the window using `right`.

When:

```text
sum >= target
```

the current window is valid.

Now shrink from the left as much as possible while keeping the sum at least `target`.

---

## Dry Run

```text
target = 7
nums = [2,3,1,2,4,3]
```

Expand until sum becomes at least `7`:

```text
[2,3,1,2] → sum = 8 → length = 4
```

Shrink:

```text
[3,1,2] → sum = 6 → invalid
```

Continue expanding:

```text
[3,1,2,4] → sum = 10 → length = 4
```

Shrink:

```text
[1,2,4] → sum = 7 → length = 3
[2,4]   → sum = 6 → invalid
```

Continue:

```text
[2,4,3] → sum = 9
[4,3]   → sum = 7 → length = 2
```

Answer:

```text
2
```

---

## Algorithm

1. Initialize `left = 0`, `sum = 0`, and `ans = INT_MAX`.
2. Move `right` through the array.
3. Add `nums[right]` to `sum`.
4. While `sum >= target`:

   * Update the minimum length.
   * Remove `nums[left]`.
   * Move `left` forward.
5. Return `0` if no valid subarray exists.

---

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

---

## Notes / Tips

The important condition is:

```text
while (sum >= target)
```

Use `while`, not `if`, because we need to shrink the window as much as possible.

This works because all numbers are **positive**:

* Expand → sum increases.
* Shrink → sum decreases.

### Key Template

```text
left = 0
sum = 0

for right = 0 to n - 1:
    sum += nums[right]

    while sum >= target:
        update answer
        sum -= nums[left]
        left++
```

---

## Code

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int left = 0;
        int sum = 0;
        int ans = INT_MAX;

        for (int right = 0; right < nums.size(); right++) {
            sum += nums[right];

            while (sum >= target) {
                ans = min(ans, right - left + 1);
                sum -= nums[left];
                left++;
            }
        }

        return ans == INT_MAX ? 0 : ans;
    }
};
```

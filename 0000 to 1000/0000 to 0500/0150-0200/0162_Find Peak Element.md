# 162. Find Peak Element

## Metadata

* **Topic:** Binary Search
* **Difficulty:** Medium
* **Pattern:** Binary Search on Answer / Peak Finding
* **Key Pattern:** Compare `nums[mid]` with `nums[mid + 1]`
* **Key Template:** Binary search on a condition that tells which half contains the answer

---

## Idea

We need to find any **peak element**, where:

```text
nums[i] > nums[i - 1] && nums[i] > nums[i + 1]
```

The important observation is that we **do not need to find the maximum element**.

We can use Binary Search:

* If `nums[mid] < nums[mid + 1]`, we are on an **increasing slope**.

  * A peak must exist on the **right side**.
  * Move `low = mid + 1`.

* Otherwise, `nums[mid] > nums[mid + 1]`, we are on a **decreasing slope** or at a peak.

  * A peak exists at `mid` or somewhere on the **left side**.
  * Move `high = mid`.

Eventually, `low == high`, and that index is a peak.

The problem guarantees that `nums[i] != nums[i + 1]`, so there are no equal adjacent elements.

---

## Dry Run

### Example

```text
nums = [1, 2, 1, 3, 5, 6, 4]
```

Start:

```text
low = 0
high = 6
```

### Step 1

```text
mid = 3

nums[mid]     = 3
nums[mid + 1] = 5
```

Since:

```text
3 < 5
```

we are moving uphill, so a peak must exist on the right.

```text
low = mid + 1 = 4
```

---

### Step 2

```text
low = 4
high = 6

mid = 5

nums[mid]     = 6
nums[mid + 1] = 4
```

Since:

```text
6 > 4
```

we are moving downhill, so a peak exists at `mid` or to its left.

```text
high = mid = 5
```

---

### Step 3

```text
low = 4
high = 5

mid = 4

nums[mid]     = 5
nums[mid + 1] = 6
```

Since:

```text
5 < 6
```

move right:

```text
low = mid + 1 = 5
```

Now:

```text
low == high == 5
```

Therefore:

```text
index = 5
nums[5] = 6
```

`6` is a peak.

---

## Algorithm

1. Initialize `low = 0` and `high = n - 1`.
2. While `low < high`:

   * Calculate `mid`.
   * Compare `nums[mid]` with `nums[mid + 1]`.
   * If `nums[mid] < nums[mid + 1]`, move to the right:

     ```text
     low = mid + 1
     ```
   * Otherwise, move to the left including `mid`:

     ```text
     high = mid
     ```
3. When `low == high`, return `low`.

---

## Complexity

* **Time:** `O(log n)`
* **Space:** `O(1)`

---

## Notes / Tips

* We are finding **any peak**, not necessarily the global maximum.
* The key is to look at the **slope** between `mid` and `mid + 1`.
* Increasing slope → peak is on the **right**.
* Decreasing slope → peak is on the **left or at `mid`**.
* Use:

  ```text
  high = mid
  ```

  instead of `high = mid - 1`, because `mid` itself can be the peak.
* Use:

  ```text
  low = mid + 1
  ```

  when moving right because `mid` is definitely not the peak in that case.
* This is a classic **binary search on a property/condition**, rather than searching for a specific value.
* The condition `low < high` is useful because we access `mid + 1`.

---

## Key Template

```text
low = 0
high = n - 1

while (low < high) {
    mid = low + (high - low) / 2

    if (condition using mid and mid + 1) {
        low = mid + 1
    }
    else {
        high = mid
    }
}

return low
```

For this problem:

```text
if nums[mid] < nums[mid + 1]:
    low = mid + 1
else:
    high = mid
```

---

## Code

```
class Solution {
public:
    int findPeakElement(vector<int>& nums) {
        int low = 0;
        int high = nums.size() - 1;

        while (low < high) {
            int mid = low + (high - low) / 2;

            if (nums[mid] < nums[mid + 1]) {
                low = mid + 1;
            }
            else {
                high = mid;
            }
        }

        return low;
    }
};
```

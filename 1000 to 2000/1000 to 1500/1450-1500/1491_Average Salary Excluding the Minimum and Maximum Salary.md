# LeetCode 1491 — Average Salary Excluding the Minimum and Maximum Salary

## Metadata

- **LeetCode:** 1491
- **Problem:** Average Salary Excluding the Minimum and Maximum Salary
- **Difficulty:** Easy
- **Topics:** Array, Sorting
- **Pattern:** Single-Pass Min/Max/Sum Tracking
- **Key Technique:** Track min, max, and total sum in one pass, then compute `(sum - min - max) / (n - 2)`

---

# Approaches
1. **Optimal — Single-Pass Tracking**

---

# Approach - Optimal: Single-Pass Tracking

## Idea

No sorting needed — a single pass can track the running sum, minimum, and maximum simultaneously. Subtract min and max from the total sum at the end.

## Dry Run

```text
salary = [4000, 3000, 1000, 2000]
```

Process:
```text
4000 → sum=4000, min=4000, max=4000
3000 → sum=7000, min=3000, max=4000
1000 → sum=8000, min=1000, max=4000
2000 → sum=10000, min=1000, max=4000
```

Exclude min and max:
```text
(10000 - 1000 - 4000) / (4 - 2) = 5000 / 2 = 2500
```

## Algorithm

1. Initialize `sum = 0`, `minVal = +infinity`, `maxVal = -infinity`.
2. For each value in `salary`:
   - Add it to `sum`.
   - Update `minVal` and `maxVal` if needed.
3. Return `(sum - minVal - maxVal) / (n - 2)`.

## Complexity

- **Time:** `O(n)`
- **Space:** `O(1)`

## Notes / Tips

- Whenever a problem only needs min, max, and sum (not the full sorted order), a single pass beats sorting every time.
- Watch for integer division — cast to `double` before or during the division to avoid truncating the result.
- Constraints guarantee at least 3 salaries, so `n - 2` is always safely positive.

## Code

```cpp
class Solution {
public:
    double average(vector<int>& salary) {
        long sum = 0;
        int minVal = INT_MAX, maxVal = INT_MIN;

        for (int s : salary) {
            sum += s;
            minVal = min(minVal, s);
            maxVal = max(maxVal, s);
        }

        return (double)(sum - minVal - maxVal) / (salary.size() - 2);
    }
};
```

---

# Key Template

### Single-Pass Min/Max/Sum

```text
sum = 0
minVal = +infinity
maxVal = -infinity

for x in arr:
    sum += x
    minVal = min(minVal, x)
    maxVal = max(maxVal, x)

return (sum - minVal - maxVal) / (n - 2)
```

## Pattern Recognition

The key observation is:
> **If only min, max, and sum matter — not full order — a single pass replaces sorting entirely.**
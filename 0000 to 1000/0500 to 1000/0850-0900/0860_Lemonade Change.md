# Lemonade Change

## Problem

At a lemonade stand, each lemonade costs `$5`.

Customers pay using `$5`, `$10`, or `$20` bills.

You must give the correct change to each customer in order.

Return `true` if you can provide the correct change to every customer, otherwise return `false`.

Example:

```text
bills = [5,5,5,10,20]
Output = true
```

---

## Approach 1: Greedy

### Idea

Maintain the number of `$5` and `$10` bills.

For each customer:

* `$5` → no change needed, so keep it.
* `$10` → need one `$5` as change.
* `$20` → need `$15` as change.

  * Prefer one `$10` + one `$5`.
  * Otherwise, use three `$5` bills.

For `$20`, using `$10 + $5` is better because it preserves more `$5` bills for future `$10` customers.

### Dry Run

```text
bills = [5,5,5,10,20]

5  → five = 1
5  → five = 2
5  → five = 3

10 → give one 5
   → five = 2, ten = 1

20 → give 10 + 5
   → five = 1, ten = 0

Answer = true
```

### Algorithm

1. Initialize `five = 0` and `ten = 0`.
2. Traverse every bill.
3. If bill is `5`:

   * Increment `five`.
4. If bill is `10`:

   * If `five == 0`, return `false`.
   * Decrement `five`.
   * Increment `ten`.
5. If bill is `20`:

   * Prefer giving `10 + 5` if both are available.
   * Otherwise, give `3 × 5`.
   * If neither is possible, return `false`.
6. Return `true`.

### Complexity

* Time: `O(n)`
* Space: `O(1)`

### Code

```cpp
class Solution {
public:
    bool lemonadeChange(vector<int>& bills) {
        int five = 0;
        int ten = 0;

        for (int bill : bills) {
            if (bill == 5) {
                five++;
            }
            else if (bill == 10) {
                if (five == 0) {
                    return false;
                }

                five--;
                ten++;
            }
            else {
                if (ten > 0 && five > 0) {
                    ten--;
                    five--;
                }
                else if (five >= 3) {
                    five -= 3;
                }
                else {
                    return false;
                }
            }
        }

        return true;
    }
};
```

### Notes / Tips

* This is a classic **Greedy** problem.
* `$5` bills are the most valuable for giving change because both `$10` and `$20` customers may need them.
* For a `$20` bill, prefer:

  ```text
  $10 + $5
  ```

  over:

  ```text
  $5 + $5 + $5
  ```
* Never give three `$5`s when a `$10 + $5` combination is available.
* We only need to track the number of `$5` and `$10` bills.

### Key Template

```text
five = 0
ten = 0

for each bill:

    if bill == 5:
        five++

    else if bill == 10:
        if five == 0:
            return false

        five--
        ten++

    else:
        if ten > 0 and five > 0:
            ten--
            five--
        else if five >= 3:
            five -= 3
        else:
            return false

return true
```

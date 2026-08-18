# LeetCode 121 — Best Time to Buy and Sell Stock

## Metadata

* **LeetCode:** 121
* **Problem:** Best Time to Buy and Sell Stock
* **Difficulty:** Easy
* **Topics:** Array, Dynamic Programming
* **Pattern:** Greedy, Prefix Minimum
* **Key Technique:** Track minimum buying price and maximum profit
* **Optimal Complexity:** `O(n)` Time, `O(1)` Space

---

## Problem

You are given an array `prices` where `prices[i]` represents the price of a stock on the `i`-th day.

Choose **one day to buy** and a **later day to sell** to maximize profit.

Return the maximum profit.

If no profitable transaction is possible, return `0`.

### Example

```text id="w6q8fe"
prices = [7, 1, 5, 3, 6, 4]

Buy at 1
Sell at 6

Profit = 6 - 1 = 5
```

---

# Approach 1 — Brute Force

## Idea

Try every possible pair of days:

* Choose every possible buying day `i`.
* For every `i`, try every later selling day `j`.
* Calculate:

```text
profit = prices[j] - prices[i]
```

Keep track of the maximum profit.

The important condition is:

```text
j > i
```

because the stock must be bought **before** it is sold.

## Dry Run

```text id="x7kq4v"
prices = [7, 1, 5, 3, 6, 4]

Buy at 7:
    sell at 1 → -6
    sell at 5 → -2
    sell at 3 → -4
    sell at 6 → -1
    sell at 4 → -3

Buy at 1:
    sell at 5 → 4
    sell at 3 → 2
    sell at 6 → 5
    sell at 4 → 3

Buy at 5:
    sell at 3 → -2
    sell at 6 → 1
    sell at 4 → -1

Buy at 3:
    sell at 6 → 3
    sell at 4 → 1

Maximum = 5
```

## Notes / Tips

* This directly checks every valid transaction.
* It is useful for understanding the problem but wastes work because many pairs are examined independently.

## Complexity

* **Time:** `O(n²)`
* **Space:** `O(1)`

## Code

```cpp id="9nqf4x"
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int maxProfit = 0;

        for (int i = 0; i < prices.size(); i++) {
            for (int j = i + 1; j < prices.size(); j++) {
                maxProfit = max(maxProfit, prices[j] - prices[i]);
            }
        }

        return maxProfit;
    }
};
```

---

# Approach 2 — Prefix Minimum / Greedy

## Idea

Instead of trying every buying day for every selling day, process the array from left to right.

For each day:

1. Maintain the **minimum price seen so far**.
2. Treat the current price as the selling price.
3. Calculate the profit obtained by selling today:

```text
current profit = current price - minimum price
```

4. Update the maximum profit.

The key observation is:

> For any selling day, the best possible buying day is simply the cheapest day that occurred before it.

So we only need to remember the minimum price so far.

## Dry Run

```text id="3r1f0x"
prices = [7, 1, 5, 3, 6, 4]

Initial:
minPrice = ∞
maxProfit = 0
```

### Day 1 — Price = 7

```text
minPrice = 7
profit = 7 - 7 = 0
maxProfit = 0
```

### Day 2 — Price = 1

```text
minPrice = min(7, 1) = 1
profit = 1 - 1 = 0
maxProfit = 0
```

### Day 3 — Price = 5

```text
minPrice = 1
profit = 5 - 1 = 4
maxProfit = 4
```

### Day 4 — Price = 3

```text
minPrice = 1
profit = 3 - 1 = 2
maxProfit = 4
```

### Day 5 — Price = 6

```text
minPrice = 1
profit = 6 - 1 = 5
maxProfit = 5
```

### Day 6 — Price = 4

```text
minPrice = 1
profit = 4 - 1 = 3
maxProfit = 5
```

Final:

```text
answer = 5
```

## Why Does This Work?

Suppose we are currently at day `j`.

The selling price is fixed:

```text
prices[j]
```

To maximize:

```text
prices[j] - prices[i]
```

we need to minimize `prices[i]`, subject to:

```text
i < j
```

Therefore, the best buying price for day `j` is simply:

```text
minimum price among all previous days
```

This allows every possible selling day to be considered in `O(1)` time.

## Important Point — Update Order

For each price:

```text
minPrice = min(minPrice, price);
maxProfit = max(maxProfit, price - minPrice);
```

Updating `minPrice` first is safe because selling and buying on the **same day** produces:

```text
price - price = 0
```

which is never better than a positive profit.

It also naturally handles decreasing arrays.

### Example

```text id="x5b0w9"
prices = [7, 6, 5, 4, 3]

minPrice keeps decreasing.

No positive profit is possible.

answer = 0
```

## Notes / Tips

* Think **"cheapest price so far"**, not "best pair".
* The current day is always treated as the potential selling day.
* `minPrice` must only represent prices from the current day or earlier.
* If every price decreases, the answer remains `0`.
* This is a classic **one-pass greedy** problem.
* The same idea can be viewed as maintaining a **prefix minimum**.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

## Code

```cpp id="q7wq1b"
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int minPrice = INT_MAX;
        int maxProfit = 0;

        for (int price : prices) {
            minPrice = min(minPrice, price);
            maxProfit = max(maxProfit, price - minPrice);
        }

        return maxProfit;
    }
};
```

---

# Approach 3 — State-Based Greedy

## Idea

The same optimal solution can be viewed using two states:

### State 1 — Best Buying Price

Track the cheapest price at which the stock could have been bought so far.

```text
buy = minimum price seen so far
```

### State 2 — Best Profit

For every current price, calculate:

```text
profit = current price - buy
```

and keep the maximum.

This gives two continuously maintained pieces of information:

```text
minimum buying price
        ↓
current selling price
        ↓
maximum profit
```

## Dry Run

```text id="0z8s0v"
prices = [3, 2, 6, 5, 0, 3]

price   minPrice   profit   maxProfit
--------------------------------------
  3        3          0         0
  2        2          0         0
  6        2          4         4
  5        2          3         4
  0        0          0         4
  3        0          3         4
```

Final answer:

```text
4
```

The best transaction is:

```text
Buy at 2
Sell at 6
Profit = 4
```

## Notes / Tips

* This is mathematically the same `O(n)` greedy solution as Approach 2.
* The important state is only:

  * cheapest price seen so far
  * maximum profit seen so far
* No array, stack, or DP table is required.
* The problem becomes simple once the buying and selling decisions are separated.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

## Code

```cpp id="5k2kqv"
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int buy = INT_MAX;
        int profit = 0;

        for (int price : prices) {
            buy = min(buy, price);
            profit = max(profit, price - buy);
        }

        return profit;
    }
};
```

---

# Key Pattern

```text
For every selling day:
    find cheapest price before it
    calculate today's profit
    keep maximum profit
```

General template:

```cpp
int minPrice = INT_MAX;
int maxProfit = 0;

for (int price : prices) {
    minPrice = min(minPrice, price);
    maxProfit = max(maxProfit, price - minPrice);
}
```

### Core Formula

```text
profit = selling price - minimum previous buying price
```

The entire problem reduces to maintaining a **prefix minimum** while scanning the array once.

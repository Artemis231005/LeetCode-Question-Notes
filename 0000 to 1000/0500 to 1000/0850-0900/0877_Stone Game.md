# Stone Game

## Problem

Alice and Bob play a game with an even number of piles of stones.

* Alice goes first.
* On each turn, a player takes an entire pile from either the **beginning or the end**.
* Both players play optimally.
* Return whether Alice can win.

Example:

```text
piles = [5,3,4,5]
Output = true
```

---

## Approach 1: Dynamic Programming — Minimax

### Idea

At every turn, a player has two choices:

* Take the left pile.
* Take the right pile.

Instead of storing the maximum score directly, define:

```text
dp[l][r] = maximum score difference
           (current player - opponent)
           that can be obtained from piles[l...r]
```

For the current player:

* Take left:

  ```text
  piles[l] - dp[l+1][r]
  ```
* Take right:

  ```text
  piles[r] - dp[l][r-1]
  ```

We subtract the opponent's best score difference because the opponent becomes the current player after our move.

Therefore:

```text
dp[l][r] = max(
    piles[l] - dp[l+1][r],
    piles[r] - dp[l][r-1]
)
```

If the final score difference is positive, Alice wins.

### Dry Run

```text
piles = [5,3,4,5]

For one pile:
dp[i][i] = piles[i]

For [5,3]:
max(5 - 3, 3 - 5)
= max(2,-2)
= 2

For [3,4]:
max(3 - 4, 4 - 3)
= 1

For [4,5]:
max(4 - 5, 5 - 4)
= 1

Eventually:

dp[0][3] = 1

Score difference > 0
→ Alice wins
```

### Algorithm

1. Create a DP table where `dp[i][j]` represents the maximum score difference for the subarray `i...j`.
2. Initialize `dp[i][i] = piles[i]`.
3. Consider subarrays of increasing length.
4. For each range `[l, r]`:

   * Calculate the result if the current player takes the left pile.
   * Calculate the result if the current player takes the right pile.
   * Store the maximum.
5. Return whether `dp[0][n-1] > 0`.

### Complexity

* Time: `O(n²)`
* Space: `O(n²)`

### Code

```cpp
class Solution {
public:
    bool stoneGame(vector<int>& piles) {
        int n = piles.size();

        vector<vector<int>> dp(n, vector<int>(n));

        for (int i = 0; i < n; i++) {
            dp[i][i] = piles[i];
        }

        for (int len = 2; len <= n; len++) {
            for (int left = 0; left + len <= n; left++) {
                int right = left + len - 1;

                int takeLeft = piles[left] - dp[left + 1][right];
                int takeRight = piles[right] - dp[left][right - 1];

                dp[left][right] = max(takeLeft, takeRight);
            }
        }

        return dp[0][n - 1] > 0;
    }
};
```

### Notes / Tips

* This is a **Minimax + DP** problem.
* The important trick is storing **score difference**, not individual scores.
* `dp[l][r]` always represents the advantage of the player whose turn it currently is.
* After taking a pile, the opponent becomes the current player, so we **subtract** the opponent's optimal difference.
* Since the number of piles is even, the first player can always guarantee a win for the constraints of this problem. Therefore, a simpler solution can return `true`, but the DP approach is more useful for learning the general game-DP pattern.
* General pattern:

  ```text
  current choice - opponent's best result
  ```

### Key Template

```text
dp[l][r] = maximum score difference
           current player can achieve

dp[l][r] = max(
    value[l] - dp[l+1][r],
    value[r] - dp[l][r-1]
)
```

# LeetCode 292 — Nim Game

## Metadata

* **LeetCode:** 292
* **Problem:** Nim Game
* **Difficulty:** Easy
* **Topics:** Math, Dynamic Programming, Brainteaser, Game Theory
* **Pattern:** Game State DP → Closed-Form Pattern Recognition
* **Key Technique:** Compute whether each pile size is a winning or losing position bottom-up; the losing positions turn out to repeat every 4 stones, collapsing the whole DP into a single modulo check
* **Optimal Complexity:** `O(1)` Time, `O(1)` Space

---

## Problem Statement

You and a friend take turns removing `1`, `2`, or `3` stones from a pile of `n` stones. Whoever removes the last stone wins. Given `n`, and assuming both play optimally, and you go first, return `true` if you can win the game.

---

## Approaches

1. **Brute Force — Bottom-Up DP Over All Pile Sizes**
2. **Optimal — Closed-Form Pattern (n % 4 != 0)**

---

# Approach 1 — Brute Force / Bottom-Up DP Over All Pile Sizes

## Idea

Build up the answer for every pile size from `0` to `n`. A pile size `i` is a **winning** position for the player about to move if there exists *some* move (removing `1`, `2`, or `3` stones) that leaves the opponent in a **losing** position. A pile of `0` stones (no stones left to take) is a losing position for whoever is "about to move" (they've already lost, since the previous player took the last stone).

## Dry Run

```text
n = 4
```

```text
dp[0] = false (no stones left → the player "to move" has already lost)
dp[1] = true  (take 1, leave dp[0]=false for opponent → win)
dp[2] = true  (take 2, leave dp[0]=false → win)
dp[3] = true  (take 3, leave dp[0]=false → win)
dp[4] = ?
   take 1 → leaves dp[3]=true (opponent wins) → bad
   take 2 → leaves dp[2]=true (opponent wins) → bad
   take 3 → leaves dp[1]=true (opponent wins) → bad
   no move leaves opponent in a losing position → dp[4] = false
```

`dp[4] = false` → return `false` (matches the known result: with 4 stones, the first player always loses against optimal play).

## Algorithm

1. Initialize `dp` array of size `n + 1`, with `dp[0] = false`.
2. For each `i` from `1` to `n`:

   * `dp[i] = true` if **any** of `dp[i-1]`, `dp[i-2]`, `dp[i-3]` (when the index is valid, i.e. `>= 0`) is `false`; otherwise `dp[i] = false`.
3. Return `dp[n]`.

## Complexity

* **Time:** `O(n)`

  * One pass building up `dp` from `0` to `n`, each step doing constant work (checking up to 3 previous states).
* **Space:** `O(n)`

  * For the `dp` array.

## Notes / Tips

* This DP reveals a clear repeating pattern almost immediately: `dp[0]=false, dp[1]=true, dp[2]=true, dp[3]=true, dp[4]=false, dp[5]=true, ...` — losing positions occur exactly at multiples of `4`.
* Recognizing this pattern is what collapses the entire DP into the `O(1)` check in Approach 2 — computing and storing the whole table is unnecessary once the periodicity is spotted.
* Given `n`'s typical constraint size in this problem, even the `O(n)` DP runs instantly, but it doesn't scale to extremely large `n` the way the closed-form check does.

## Code

```cpp
class Solution {
public:
    bool canWinNim(int n) {
        vector<bool> dp(n + 1, false);

        for (int i = 1; i <= n; i++) {
            dp[i] = (i - 1 >= 0 && !dp[i - 1]) ||
                    (i - 2 >= 0 && !dp[i - 2]) ||
                    (i - 3 >= 0 && !dp[i - 3]);
        }

        return dp[n];
    }
};
```

---

# Approach 2 — Optimal / Closed-Form Pattern (n % 4 != 0)

## Idea

The DP from Approach 1 shows losing positions occur exactly when `n` is a multiple of `4`: whatever the first player removes (`1`, `2`, or `3`), the opponent can always remove enough (`3`, `2`, or `1` respectively) to bring the total removed in that round to exactly `4`, keeping the pile a multiple of `4` after their turn. This forces the first player to eventually face a pile of exactly `0` on their turn, at which point they've already lost (the opponent took the last stone). If `n` is **not** a multiple of `4`, the first player can always remove just enough stones (`n mod 4`) to leave a multiple of `4` for the opponent, putting the opponent in the losing role instead.

## Dry Run

```text
n = 4
```

```text
4 % 4 == 0 → first player loses → return false
```

```text
n = 5
```

```text
5 % 4 == 1 → first player takes 1 stone, leaving 4 (a multiple of 4) for the opponent
→ opponent is now stuck in the losing pattern → first player wins → return true
```

## Algorithm

1. Return `n % 4 != 0`.

## Complexity

* **Time:** `O(1)`

  * A single modulo operation.
* **Space:** `O(1)`

  * No extra structures used.

## Notes / Tips

* This is a classic combinatorial game theory result: whenever a game allows removing `1` to `k` items per turn and the goal is to take the last item, the losing positions for the player about to move are exactly the multiples of `k + 1` — here `k = 3`, so losing positions are multiples of `4`.
* The winning strategy in practice: whenever facing a non-multiple-of-4 pile, always remove `n mod 4` stones to hand the opponent a multiple of `4`. Repeating this each turn guarantees eventually leaving the opponent with `0` stones on their turn.
* This is a good example of how brute-force DP over small game states can reveal a periodic pattern that collapses into a trivial closed-form check, worth trying on any small game-state DP before assuming the DP itself is the final answer.

## Code

```cpp
class Solution {
public:
    bool canWinNim(int n) {
        return n % 4 != 0;
    }
};
```

---

## Key Template

```text
return n % 4 != 0
```
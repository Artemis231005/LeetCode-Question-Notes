# Smallest Divisible Digit Product II

## Problem

Given a positive integer `n` as a string and an integer `t`, find the **smallest number greater than or equal to `n`** whose product of digits is divisible by `t`.

If no such number exists, return `"-1"`.

Example:

```text
n = "123"
t = 6

123 → 1 × 2 × 3 = 6

Output = "123"
```

---

## Approach 1: Digit Factorization + Digit DP / Greedy Construction

### Idea

The product of decimal digits can only contain the prime factors:

```text
2, 3, 5, 7
```

So first factorize `t` into these primes.

If `t` contains any prime factor other than `2`, `3`, `5`, or `7`, no digit product can ever be divisible by `t`, so return `"-1"`.

Then construct the smallest valid number `>= n`.

For each digit, we need to track how many factors of:

```text
2, 3, 5, 7
```

are still required.

Because the required factor counts are small, we can use DP with states representing:

```text
(position, remaining factors, tight)
```

where `tight` tells us whether the constructed prefix is still exactly equal to the prefix of `n`.

### Important Digit Factorizations

```text
0 → product becomes 0
1 → no prime factors
2 → 2
3 → 3
4 → 2²
5 → 5
6 → 2 × 3
7 → 7
8 → 2³
9 → 3²
```

If a `0` is used, the entire digit product becomes `0`, which is divisible by every positive `t`.

### Dry Run

```text
n = "123"
t = 6

Factorize:
6 = 2 × 3

Check 123:
1 × 2 × 3 = 6

Product is divisible by 6.

Therefore:
answer = "123"
```

For a case where `n` itself does not work, the DP tries the smallest possible digit while maintaining the condition that the resulting number is still `>= n`.

### Algorithm

1. Factorize `t` using primes `2, 3, 5, 7`.
2. If a remaining factor exists after factorization, return `"-1"`.
3. Represent the required factors as counts of `2, 3, 5, 7`.
4. Precompute the factor contribution of every digit `0...9`.
5. Use digit DP to determine whether a valid number can be constructed from each state.
6. Try constructing a number of the same length as `n`.
7. If impossible, construct the smallest valid number with one more digit.
8. During reconstruction, always choose the smallest digit that allows a valid completion.
9. Return the constructed number.

### Complexity

Let `L` be the number of digits in `n`.

The number of possible factor-count states is bounded because `t <= 10^14`.

* Time: approximately `O(L × states × 10)`
* Space: `O(L × states)`

### Code

```cpp
class Solution {
public:
    string n;
    int target[4];
    int memo[20][16][16][8][8][2];
    bool vis[20][16][16][8][8][2];

    int factorCount[10][4] = {
        {0, 0, 0, 0},
        {0, 0, 0, 0},
        {1, 0, 0, 0},
        {0, 1, 0, 0},
        {2, 0, 0, 0},
        {0, 0, 1, 0},
        {1, 1, 0, 0},
        {0, 0, 0, 1},
        {3, 0, 0, 0},
        {0, 2, 0, 0}
    };

    bool canBuild(int pos, int a, int b, int c, int d, int tight) {
        if (pos == n.size()) {
            return a == 0 && b == 0 && c == 0 && d == 0;
        }

        if (vis[pos][a][b][c][d][tight]) {
            return memo[pos][a][b][c][d][tight];
        }

        vis[pos][a][b][c][d][tight] = true;

        int low = tight ? n[pos] - '0' : 0;

        for (int digit = low; digit <= 9; digit++) {
            if (pos == 0 && n.size() > 1 && digit == 0) {
                continue;
            }

            int na = max(0, a - factorCount[digit][0]);
            int nb = max(0, b - factorCount[digit][1]);
            int nc = max(0, c - factorCount[digit][2]);
            int nd = max(0, d - factorCount[digit][3]);

            int ntight = tight && (digit == low);

            if (canBuild(pos + 1, na, nb, nc, nd, ntight)) {
                return memo[pos][a][b][c][d][tight] = 1;
            }
        }

        return memo[pos][a][b][c][d][tight] = 0;
    }

    string buildSameLength() {
        if (!canBuild(0, target[0], target[1], target[2], target[3], 1)) {
            return "";
        }

        string ans;
        int a = target[0];
        int b = target[1];
        int c = target[2];
        int d = target[3];
        int tight = 1;

        for (int pos = 0; pos < n.size(); pos++) {
            int low = tight ? n[pos] - '0' : 0;

            for (int digit = low; digit <= 9; digit++) {
                if (pos == 0 && n.size() > 1 && digit == 0) {
                    continue;
                }

                int na = max(0, a - factorCount[digit][0]);
                int nb = max(0, b - factorCount[digit][1]);
                int nc = max(0, c - factorCount[digit][2]);
                int nd = max(0, d - factorCount[digit][3]);

                int ntight = tight && (digit == low);

                if (canBuild(pos + 1, na, nb, nc, nd, ntight)) {
                    ans += char('0' + digit);
                    a = na;
                    b = nb;
                    c = nc;
                    d = nd;
                    tight = ntight;
                    break;
                }
            }
        }

        return ans;
    }

    string smallestNumber(int length) {
        string ans;

        int a = target[0];
        int b = target[1];
        int c = target[2];
        int d = target[3];

        for (int pos = 0; pos < length; pos++) {
            for (int digit = 1; digit <= 9; digit++) {
                int na = max(0, a - factorCount[digit][0]);
                int nb = max(0, b - factorCount[digit][1]);
                int nc = max(0, c - factorCount[digit][2]);
                int nd = max(0, d - factorCount[digit][3]);

                bool possible = false;

                if (pos == length - 1) {
                    possible = (na == 0 && nb == 0 && nc == 0 && nd == 0);
                }

                if (possible) {
                    ans += char('0' + digit);
                    a = na;
                    b = nb;
                    c = nc;
                    d = nd;
                    break;
                }
            }
        }

        return ans;
    }

    string smallestNumber(string N, long long t) {
        n = N;

        for (int i = 0; i < 4; i++) {
            target[i] = 0;
        }

        long long primes[] = {2, 3, 5, 7};

        for (int i = 0; i < 4; i++) {
            while (t % primes[i] == 0) {
                target[i]++;
                t /= primes[i];
            }
        }

        if (t != 1) {
            return "-1";
        }

        memset(vis, false, sizeof(vis));

        string ans = buildSameLength();

        if (!ans.empty()) {
            return ans;
        }

        int length = n.size() + 1;

        while (true) {
            string result = smallestNumber(length);

            if (!result.empty()) {
                return result;
            }

            length++;
        }
    }
};
```

### Notes / Tips

* The key observation is that **only prime factors `2, 3, 5, 7` can appear in a non-zero decimal digit product**.
* Factorizing `t` first tells us whether a solution is possible.
* Digit `0` is special because once it appears, the entire product becomes `0`, which is divisible by any positive `t`.
* This is much harder than ordinary greedy digit construction because we must satisfy both:

  * `answer >= n`
  * digit product divisible by `t`
* The general pattern is **digit DP + state compression using prime-factor counts**.
* For smaller constraints, a simple brute-force search may work, but for the actual constraints of this problem, state-based construction is the important idea.

### Key Template

```text
factorize t into 2, 3, 5, 7

if another prime factor remains:
    return "-1"

required = factor counts of 2, 3, 5, 7

digit DP state:
    position
    remaining factor counts
    tight

try digits from smallest to largest

if same-length answer exists:
    construct smallest one
else:
    construct smallest valid number with more digits
```

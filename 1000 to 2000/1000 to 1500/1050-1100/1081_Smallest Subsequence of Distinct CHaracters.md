# LeetCode 1081 — Smallest Subsequence of Distinct Characters

## Metadata

- **LeetCode:** 1081
- **Problem:** Smallest Subsequence of Distinct Characters
- **Difficulty:** Medium
- **Topics:** String, Stack, Greedy, Monotonic Stack
- **Pattern:** Monotonic Stack with Last-Occurrence Lookahead
- **Key Technique:** Pop a larger character off the stack only if it reappears later and hasn't already been used (identical problem to LC 316 — Remove Duplicate Letters)

---

# Approaches

1. **Brute Force — Greedy Recursion**
2. **Optimal — Monotonic Stack + Last Occurrence Map**

---

# Approach 1 — Brute Force / Greedy Recursion

## Idea

At each step, pick the smallest character that can safely start the result — meaning every other distinct character remaining still appears somewhere after it. Fix that character, drop everything before it and all its other occurrences, then recurse on the rest.

## Dry Run

```text
s = "cdadabcc"
```

Distinct chars: `a, b, c, d`

Smallest safe starting char: `a` (its earliest position still leaves `b, c, d` reachable after).

Take `a`, drop prefix + other `a`'s:
```text
remaining = "bcc"
```

Recurse → smallest safe char is `b`:
```text
remaining = "cc"
```

Recurse → smallest safe char is `c`:
```text
remaining = ""
```

Final:
```text
"abc"
```

## Algorithm

1. If `s` is empty, return `""`.
2. Count occurrences of each character in `s`.
3. Scan `s`, tracking the smallest character seen so far as candidate cut position `pos`, decrementing counts as you go.
4. Stop scanning once the count of the current character hits `0` (no more occurrences left, so it's safe to cut).
5. Take `s[pos]`, recurse on `s[pos+1:]` with all occurrences of `s[pos]` removed.
6. Concatenate and return.

## Complexity

- **Time:** `O(26 * n)` — up to 26 recursive levels, each scanning up to `n` characters
- **Space:** `O(n)` — recursion stack and intermediate strings

## Notes / Tips

- Correct but wasteful — rescanning and rebuilding substrings at every level adds overhead.

## Code

```cpp
class Solution {
public:
    string smallestSubsequence(string s) {
        if (s.empty()) return "";

        vector<int> count(26, 0);
        for (char c : s) {
            count[c - 'a']++;
        }

        int pos = 0;
        for (int i = 0; i < s.size(); i++) {
            if (s[i] < s[pos]) {
                pos = i;
            }
            count[s[i] - 'a']--;
            if (count[s[i] - 'a'] == 0) {
                break;
            }
        }

        string rest = s.substr(pos + 1);
        rest.erase(remove(rest.begin(), rest.end(), s[pos]), rest.end());

        return s[pos] + smallestSubsequence(rest);
    }
};
```

---

# Approach 2 — Optimal / Monotonic Stack + Last Occurrence Map

## Idea

Build the result with a stack. For each character:
- Skip it if already in the stack (only one copy of each distinct character allowed).
- Otherwise, while the top of the stack is **greater** than the current character **and** that top character still appears later in the string, pop it.
- Push the current character.

## Dry Run

```text
s = "cdadabcc"
```

Last occurrence index of each char:
```text
c → 7
d → 3
a → 4
b → 5
```

Process:
```text
i=0, 'c': stack empty → push → stack = [c]
i=1, 'd': 'c' < 'd' → no pop → push → stack = [c, d]
i=2, 'a': 'd' > 'a' and d reappears later (last=3 > 2) → pop d
          'c' > 'a' and c reappears later (last=7 > 2) → pop c
          stack empty → push a → stack = [a]
i=3, 'd': not in stack → push → stack = [a, d]
i=4, 'a': already in stack → skip
i=5, 'b': 'd' > 'b' and d does NOT reappear later (last=3 < 5) → can't pop d
          push b → stack = [a, d, b]
i=6, 'c': not in stack → push → stack = [a, d, b, c]
i=7, 'c': already in stack → skip
```

Final stack:
```text
[a, d, b, c] → "adbc"
```

## Algorithm

1. Track the **last occurrence index** of every character in `s`.
2. Initialize an empty stack and a `seen` set for characters currently in the stack.
3. For each index `i` and character `c = s[i]`:
   - If `c` is already in `seen`, skip it.
   - While the stack is non-empty, `stack.top() > c`, and `lastOccurrence[stack.top()] > i`, pop the top and mark it unseen.
   - Push `c` and mark it seen.
4. Join the stack into the result string.

## Complexity

- **Time:** `O(n)` — each character is pushed and popped at most once
- **Space:** `O(26)` for the stack/seen set, `O(n)` for the last-occurrence map

## Notes / Tips

- Both pop conditions are required together: `stack.top() > c` (greedy improvement) and `lastOccurrence[stack.top()] > i` (safe to defer).
- The `seen` set both prevents duplicates in the output and makes membership checks `O(1)`.

## Code

```cpp
class Solution {
public:
    string smallestSubsequence(string s) {
        vector<int> lastOccurrence(26, 0);
        for (int i = 0; i < s.size(); i++) {
            lastOccurrence[s[i] - 'a'] = i;
        }

        vector<bool> seen(26, false);
        string stack;

        for (int i = 0; i < s.size(); i++) {
            char c = s[i];

            if (seen[c - 'a']) continue;

            while (!stack.empty() &&
                   stack.back() > c &&
                   lastOccurrence[stack.back() - 'a'] > i) {
                seen[stack.back() - 'a'] = false;
                stack.pop_back();
            }

            stack.push_back(c);
            seen[c - 'a'] = true;
        }

        return stack;
    }
};
```

---

# Key Template

### Monotonic Stack with Last-Occurrence + Seen Set

```text
lastOccurrence[c] = last index of c in s
seen = {}
stack = []

for i, c in enumerate(s):
    if c in seen: continue

    while stack not empty and stack.top() > c and lastOccurrence[stack.top()] > i:
        seen.remove(stack.pop())

    stack.push(c)
    seen.add(c)

return join(stack)
```

## Pattern Recognition

When you see:

```text
Distinct characters
+
Lexicographically smallest subsequence
+
Keep relative order
```

Think:

```text
Greedy: smaller char can replace a larger one on top of the stack
        ↓
Only safe to pop if the popped char reappears later
        ↓
Monotonic stack + last-occurrence map + seen set
```

The key observation is:

> **Same problem as LC 316 in disguise — "smallest subsequence with all distinct characters" and "remove duplicate letters" are the same constraint stated two different ways.**
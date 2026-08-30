# LeetCode 316 — Remove Duplicate Letters

## Metadata

- **LeetCode:** 316
- **Problem:** Remove Duplicate Letters
- **Difficulty:** Medium
- **Topics:** String, Stack, Greedy, Monotonic Stack
- **Pattern:** Monotonic Stack with Last-Occurrence Lookahead
- **Key Technique:** Pop a larger character off the stack only if it reappears later and hasn't already been used

---

# Approaches

1. **Brute Force — Greedy Recursion**
2. **Optimal — Monotonic Stack + Last Occurrence Map**

---

# Approach 1 — Brute Force / Greedy Recursion

## Idea

At each step, pick the **smallest character** that can safely start the result — meaning every other distinct character still remaining in the string appears somewhere after it. Fix that character, remove everything before it and all its other occurrences, then recurse on the rest.

## Dry Run

```text
s = "cbacdcbc"
```

Find last occurrence of each char:
```text
c → last at index 7
b → last at index 6
a → last at index 2
d → last at index 4
```

Scan from left, find smallest prefix `[0..i]` such that every distinct char in `s` appears in `s[0..i]` (all chars covered) — pick the smallest char in that window whose own last occurrence is `>= i` isn't removed too early.

Concretely: smallest char reachable early is `a` at index 2 (since before index 2, we'd be missing `a` if we stopped earlier). Take `a`.

Remove everything up to and including index 2, and remove all other `a`'s from the rest:
```text
remaining = "dcbc"
```

Repeat on `"dcbc"` → smallest valid starting char is `b` (since `d` and `c` both reappear after position of `b`)... continue recursively.

Final result:
```text
"acdb"
```

## Algorithm

1. If `s` is empty, return `""`.
2. Count occurrences of each character in `s`.
3. Find `pos` = index of the smallest character such that all characters before `pos` still occur again later (i.e. `pos` is the earliest index where it's "safe" to cut).
   - Practically: scan `s`, tracking the smallest character seen so far that you could stop at; a character at index `i` is a valid cut point once its count would drop to `0` if skipped further, or simpler: iterate and pick smallest char whose remaining count allows deferring all larger-but-earlier chars.
4. Take `s[pos]` as the next output character.
5. Recurse on the substring of `s[pos+1:]` with all occurrences of `s[pos]` removed.
6. Concatenate and return.

## Complexity

- **Time:** `O(26 * n)` — up to 26 recursive levels (one per unique lowercase letter), each scanning up to `n` characters
- **Space:** `O(n)` — recursion stack and intermediate strings

## Notes / Tips

- Correct but wasteful — rebuilding substrings and rescanning counts at every recursive level adds unnecessary overhead.
- Good for building intuition: the answer is built by greedily choosing the smallest safe character at each position.

## Code

```cpp
class Solution {
public:
    string removeDuplicateLetters(string s) {
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

        return s[pos] + removeDuplicateLetters(rest);
    }
};
```

---

# Approach 2 — Optimal / Monotonic Stack + Last Occurrence Map

## Idea

Build the result with a stack. For each character:
- Skip it if it's already in the stack (each letter appears once in the final answer).
- Otherwise, while the top of the stack is **greater** than the current character **and** that top character still appears later in the string, pop it (it can be placed later instead).
- Push the current character.

This greedily keeps the stack as small/lexicographically-first as possible while guaranteeing every character can still be placed.

## Dry Run

```text
s = "cbacdcbc"
```

Last occurrence index of each char:
```text
c → 7
b → 6
a → 2
d → 4
```

Process:
```text
i=0, 'c': stack empty → push → stack = [c]
i=1, 'b': 'c' > 'b' and c reappears later (last=7 > 1) → pop c
          push b → stack = [b]
i=2, 'a': 'b' > 'a' and b reappears later (last=6 > 2) → pop b
          stack empty → push a → stack = [a]
i=3, 'c': not in stack → push → stack = [a, c]
i=4, 'd': not in stack → push → stack = [a, c, d]
i=5, 'c': already in stack → skip
i=6, 'b': 'd' > 'b' but d does NOT reappear later (last=4 < 6) → can't pop d
          push b → stack = [a, c, d, b]
i=7, 'c': already in stack → skip
```

Final stack:
```text
[a, c, d, b] → "acdb"
```

## Algorithm

1. Count/track the **last occurrence index** of every character in `s`.
2. Initialize an empty stack and a `seen` set (or boolean array) for characters currently in the stack.
3. For each index `i` and character `c = s[i]`:
   - If `c` is already in `seen`, skip it.
   - While the stack is non-empty, `stack.top() > c`, and `lastOccurrence[stack.top()] > i`, pop the top and mark it unseen.
   - Push `c` onto the stack and mark it seen.
4. Join the stack into the result string.

## Complexity

- **Time:** `O(n)` — each character is pushed and popped at most once
- **Space:** `O(26)` for the stack/seen set (bounded by alphabet size), `O(n)` for the last-occurrence map

## Notes / Tips

- The two conditions for popping are both required: `stack.top() > c` (greedy improvement) **and** `lastOccurrence[stack.top()] > i` (safe to defer, it reappears later) — dropping either breaks correctness.
- The `seen` set is what prevents duplicate letters in the output and also what makes a character's stack membership check `O(1)`.
- This exact pattern (monotonic stack + last-occurrence + seen-set) also solves "Smallest Subsequence of Distinct Characters" (LC 1081) — same problem, different name.

## Code

```cpp
class Solution {
public:
    string removeDuplicateLetters(string s) {
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
Remove duplicate letters
+
Lexicographically smallest result
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

> **A character on the stack can be safely removed and replaced by a smaller one only if it's guaranteed to appear again later — last-occurrence tracking is what makes that safe.**
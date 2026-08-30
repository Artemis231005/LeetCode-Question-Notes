# LeetCode 71 — Simplify Path

## Metadata

- **LeetCode:** 71
- **Problem:** Simplify Path
- **Difficulty:** Medium
- **Topics:** String, Stack
- **Pattern:** Stack-Based Path Normalization
- **Key Technique:** Split path by `/`, use a stack to handle `.`, `..`, and empty segments, then rebuild

---

# Approaches

1. **Brute Force — Split + Repeated String Concatenation**
2. **Optimal — Stack + Single Join**

---

# Approach 1 — Brute Force / Split + Repeated String Concatenation

## Idea

Split the path on `/` to get all segments. Walk through them, maintaining a running result string. For `..`, chop off the last segment from the result by finding the last `/`. For `.` or empty segments, skip. 
Rebuild the string by repeated concatenation as you go.

## Dry Run

```text
path = "/a/./b/../../c/"
```

Split on `/`:
```text
["", "a", ".", "b", "..", "..", "c", ""]
```

Process:
```text
"a"  → result = "/a"
"."  → skip
"b"  → result = "/a/b"
".." → chop last segment → result = "/a"
".." → chop last segment → result = ""
"c"  → result = "/c"
""   → skip
```

Final:
```text
"/c"
```

## Algorithm

1. Split `path` by `/` into tokens.
2. Initialize `result = ""`.
3. For each token:
   - If empty or `.`, skip.
   - If `..`, remove the last `/segment` from `result` (find last `/` and truncate).
   - Otherwise, append `/token` to `result`.
4. If `result` is empty, return `"/"`.
5. Return `result`.

## Complexity

- **Time:** `O(n^2)` — truncating/appending to a string repeatedly can shift memory each time
- **Space:** `O(n)` — for tokens and result

## Notes / Tips

- Works correctly but the repeated string chopping/concatenation is wasteful.
- Easy to get the `..` truncation logic wrong (e.g. forgetting the case where result is already empty).

## Code

```cpp
class Solution {
public:
    string simplifyPath(string path) {
        vector<string> tokens;
        stringstream ss(path);
        string token;

        while (getline(ss, token, '/')) {
            tokens.push_back(token);
        }

        string result = "";

        for (string& t : tokens) {
            if (t == "" || t == ".") {
                continue;
            } else if (t == "..") {
                size_t pos = result.find_last_of('/');
                if (pos != string::npos) {
                    result = result.substr(0, pos);
                }
            } else {
                result += "/" + t;
            }
        }

        return result.empty() ? "/" : result;
    }
};
```

---

# Approach 2 — Optimal / Stack + Single Join

## Idea

Same splitting logic, but instead of mutating a string in place, push valid directory names onto a stack (vector). For `..`, pop from the stack. At the end, join the stack contents with `/` in a single pass.

## Dry Run

```text
path = "/a/./b/../../c/"
```

Split on `/`:
```text
["", "a", ".", "b", "..", "..", "c", ""]
```

Process with stack:
```text
"a"  → push → stack = [a]
"."  → skip
"b"  → push → stack = [a, b]
".." → pop → stack = [a]
".." → pop → stack = []
"c"  → push → stack = [c]
""   → skip
```

Join:
```text
"/" + "c" = "/c"
```

## Algorithm

1. Split `path` by `/` into tokens.
2. Initialize an empty stack (vector of strings).
3. For each token:
   - If empty or `.`, skip.
   - If `..`, pop from the stack if it's non-empty.
   - Otherwise, push the token onto the stack.
4. Join stack contents with `/`, prefixed by `/`.
5. If the stack is empty, return `"/"`.

## Complexity

- **Time:** `O(n)` — each token processed once, final join is a single linear pass
- **Space:** `O(n)` — for tokens and stack

## Notes / Tips

- This is the standard approach — building the final string once at the end avoids the repeated shifting cost of Approach 1.
- Common mistake: popping from an empty stack on `..` (e.g. `"/../"` should just stay `"/"`) — always guard with an emptiness check.
- The splitting logic (`.`, `..`, empty segment handling) is reusable for any "path normalization" style problem (Unix paths, URL paths, etc.).

## Code

```cpp
class Solution {
public:
    string simplifyPath(string path) {
        vector<string> stack;
        stringstream ss(path);
        string token;

        while (getline(ss, token, '/')) {
            if (token == "" || token == ".") {
                continue;
            } else if (token == "..") {
                if (!stack.empty()) {
                    stack.pop_back();
                }
            } else {
                stack.push_back(token);
            }
        }

        string result = "";
        for (string& dir : stack) {
            result += "/" + dir;
        }

        return result.empty() ? "/" : result;
    }
};
```

---

# Key Template

### Path Normalization via Stack

```text
tokens = split(path, "/")
stack = []

for token in tokens:
    if token == "" or token == ".": continue
    if token == "..":
        if stack not empty: stack.pop()
    else:
        stack.push(token)

result = "/" + join(stack, "/")
return result if stack not empty else "/"
```

## Pattern Recognition

When you see:

```text
Path string
+
"." / ".." segments
+
Normalize / simplify
```

Think:

```text
Split by delimiter
        ↓
Stack: push real names, pop on invalid, skip "." and empty
        ↓
Join stack at the end
```

The key observation is:

> **`..` is just a pop and a real directory name is just a push — path simplification is stack simulation in disguise.**
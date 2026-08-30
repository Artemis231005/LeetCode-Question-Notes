# LeetCode 503 — Next Greater Element II

## Metadata

- **LeetCode:** 503
- **Problem:** Next Greater Element II
- **Difficulty:** Medium
- **Topics:** Array, Stack, Monotonic Stack
- **Pattern:** Monotonic Stack (Circular Array)
- **Key Technique:** Traverse the array twice (using modulo indexing) while maintaining a decreasing stack of indices to simulate circularity

---

# Approaches

1. **Brute Force — Check Every Pair (Circular)**
2. **Optimal — Monotonic Stack (Traverse Twice)**

---

# Approach 1 — Brute Force

## Idea

For every element, look ahead (wrapping around the array using modulo) until you find a greater element, or until you've checked all `n` elements.

Since the array is circular, "next" can wrap from the end back to the beginning.

## Dry Run

```text
nums = [1, 2, 1]
```

For `i = 0` (value `1`):
```text
check nums[1] = 2 → greater! answer[0] = 2
```

For `i = 1` (value `2`):
```text
check nums[2] = 1 → not greater
check nums[0] = 1 → not greater (wrapped, checked all n-1 others)
answer[1] = -1
```

For `i = 2` (value `1`):
```text
check nums[0] = 1 → not greater
check nums[1] = 2 → greater! answer[2] = 2
```

Result:
```text
[2, -1, 2]
```

## Algorithm

1. For each index `i` from `0` to `n-1`:
2. Loop `j` from `i+1` to `i+n-1`, using `nums[j % n]` to wrap around.
3. If `nums[j % n] > nums[i]`, record it as the answer and break.
4. If no greater element is found after `n-1` checks, answer is `-1`.

## Complexity

- **Time:** `O(n^2)`
- **Space:** `O(n)` (for the answer array)

## Notes / Tips

- Simple to reason about but too slow for large inputs.
- The modulo trick `j % n` is the standard way to simulate circular traversal — worth remembering on its own.

## Code

```cpp
class Solution {
public:
    vector<int> nextGreaterElements(vector<int>& nums) {
        int n = nums.size();
        vector<int> ans(n, -1);

        for (int i = 0; i < n; i++) {
            for (int j = 1; j < n; j++) {
                int next = (i + j) % n;
                if (nums[next] > nums[i]) {
                    ans[i] = nums[next];
                    break;
                }
            }
        }

        return ans;
    }
};
```

---

# Approach 2 — Optimal / Monotonic Stack (Traverse Twice)

## Idea

Circularity can be simulated without doubling the array in memory — just iterate index range `[0, 2n)` and map with `i % n`, combined with a monotonic stack to get next-greater.

- Pop from the stack while the stack's top is `<= nums[i % n]` (not useful as a next-greater for anything smaller).
- If the stack is empty, there is no next greater → `-1`.
- Otherwise, the top of the stack is the next greater element.
- Push `nums[i % n]` onto the stack (only meaningful during the second pass for real answers, but works because we push every visited index/value regardless).

## Dry Run

```text
nums = [1, 2, 1]
n = 3
```

Iterate `i` from `5` down to `0` (i.e., `2n-1` to `0`), stack holds candidate values, `idx = i % n`:

### i = 5, idx = 2, value = 1
```text
stack = []
stack empty → ans[2] = -1 (tentative)
push 1 → stack = [1]
```

### i = 4, idx = 1, value = 2
```text
stack top = 1 ≤ 2 → pop
stack = []
stack empty → ans[1] = -1 (tentative)
push 2 → stack = [2]
```

### i = 3, idx = 0, value = 1
```text
stack top = 2 > 1 → keep
ans[0] = 2
push 1 → stack = [2, 1]
```

### i = 2, idx = 2, value = 1 (second pass, real answers locked in from here)
```text
stack top = 1 ≤ 1 → pop
stack top = 2 > 1 → keep
ans[2] = 2
push 1 → stack = [2, 1]
```

### i = 1, idx = 1, value = 2
```text
stack top = 1 ≤ 2 → pop
stack top = 2 ≤ 2 → pop
stack empty → ans[1] = -1
push 2 → stack = [2]
```

### i = 0, idx = 0, value = 1
```text
stack top = 2 > 1 → keep
ans[0] = 2
push 1 → stack = [2, 1]
```

Final:
```text
ans = [2, -1, 2]
```

## Algorithm

1. Initialize `ans` array of size `n` filled with `-1`.
2. Initialize an empty stack (holds values, or indices if you need original positions).
3. Loop `i` from `2n - 1` down to `0`:
   - Let `idx = i % n`.
   - While the stack is not empty and `stack.top() <= nums[idx]`, pop.
   - If `i < n` (we're in the "real" pass, i.e., first half after conceptually looping twice):
     - If stack is empty → `ans[idx] = -1`.
     - Else → `ans[idx] = stack.top()`.
   - Push `nums[idx]` onto the stack.
4. Return `ans`.

## Complexity

- **Time:** `O(n)` — each index is pushed and popped from the stack at most a constant number of times across the `2n` iterations.
- **Space:** `O(n)` — for the stack and the answer array.

## Notes / Tips

- This is similar to **Next Greater Element I** , the only new idea is looping over `2n` indices with `i % n` instead of physically duplicating the array.
- Maintain a **decreasing stack**: pop everything `<=` current value before deciding the answer, because those elements can never be the "next greater" for anything after them.
- Common mistake: forgetting to only record the answer during the "real" pass (`i < n`) — pushing happens every iteration, but writing to `ans` should only happen once real indices are being resolved (though pushing on both passes is what makes circularity work).

## Code

```cpp
class Solution {
public:
    vector<int> nextGreaterElements(vector<int>& nums) {
        int n = nums.size();
        vector<int> ans(n, -1);
        stack<int> st; // stores values

        for (int i = 2 * n - 1; i >= 0; i--) {
            int idx = i % n;

            while (!st.empty() && st.top() <= nums[idx]) {
                st.pop();
            }

            if (i < n) {
                if (!st.empty()) {
                    ans[idx] = st.top();
                }
            }

            st.push(nums[idx]);
        }

        return ans;
    }
};
```

---

# Key Template

### Next Greater Element (Circular Array, Monotonic Stack)

```cpp
vector ans(n, -1);
stack st;

for i = 2 * n - 1 to 0, i--
    idx = i % n;

    while !st.empty() && st.top() <= nums[idx]
        st.pop()

    if i < n && !st.empty()
        ans[idx] = st.top();

    st.push(nums[idx]);
```




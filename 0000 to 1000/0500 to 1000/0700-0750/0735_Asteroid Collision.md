# LeetCode 735 — Asteroid Collision

## Metadata

- **LeetCode:** 735
- **Problem:** Asteroid Collision
- **Difficulty:** Medium
- **Topics:** Array, Stack, Simulation
- **Pattern:** Monotonic Stack Collision Simulation
- **Key Technique:** Push right-moving asteroids, and when a left-moving one arrives, resolve collisions against the stack top in a loop before deciding whether to push it

---

# Approaches

1. **Brute Force — Repeated Array Scan Until Stable**
2. **Optimal — Single-Pass Stack Simulation**

---

# Approach 1 — Brute Force / Repeated Array Scan Until Stable

## Idea

Repeatedly scan the array left to right looking for the **first** adjacent pair that collides (a positive asteroid immediately followed by a negative one). Resolve that one collision, shrink the array, and rescan from the start. Keep going until no more collisions are found.

## Dry Run

```text
asteroids = [5, 10, -5]
```

Scan 1 — collision found at index (1,2): `10` (right) and `-5` (left):
```text
|10| > |5| → -5 destroyed
asteroids = [5, 10]
```

Scan 2 — no adjacent positive-then-negative pair found:
```text
stable
```

Result:
```text
[5, 10]
```

## Algorithm

1. Repeat until no collision is found in a full pass:
   - Scan for the first index `i` where `asteroids[i] > 0` and `asteroids[i+1] < 0`.
   - Compare magnitudes:
     - If equal, remove both.
     - If left one bigger, remove the right one.
     - If right one bigger, remove the left one.
   - Restart the scan from the beginning.
2. Return the remaining array once stable.

## Complexity

- **Time:** `O(n^2)` — worst case, one collision resolved per full rescan
- **Space:** `O(n)` — for the mutable array

## Notes / Tips

- Correct but wasteful — rescanning from the start after every single collision is unnecessary.
- Useful for building intuition on the collision rules before jumping to the stack version.

## Code

```cpp
class Solution {
public:
    vector<int> asteroidCollision(vector<int>& asteroids) {
        vector<int> arr = asteroids;

        bool changed = true;
        while (changed) {
            changed = false;

            for (int i = 0; i + 1 < arr.size(); i++) {
                if (arr[i] > 0 && arr[i + 1] < 0) {
                    int left = arr[i], right = arr[i + 1];

                    if (abs(left) == abs(right)) {
                        arr.erase(arr.begin() + i, arr.begin() + i + 2);
                    } else if (abs(left) > abs(right)) {
                        arr.erase(arr.begin() + i + 1);
                    } else {
                        arr.erase(arr.begin() + i);
                    }

                    changed = true;
                    break;
                }
            }
        }

        return arr;
    }
};
```

---

# Approach 2 — Optimal / Single-Pass Stack Simulation

## Idea

Only a right-moving asteroid (positive) followed later by a left-moving one (negative) can ever collide, and collisions cascade backward into the stack. Push right-moving asteroids. When a left-moving asteroid arrives, keep popping the stack while the top is positive and smaller in magnitude (it gets destroyed). If the top is equal in magnitude, both are destroyed. If the top is positive and bigger, the incoming asteroid is destroyed. If the stack top is negative or empty, the incoming asteroid just gets pushed (no collision possible).

## Dry Run

```text
asteroids = [10, 2, -5]
```

Process:
```text
10 → positive → push → stack = [10]
2  → positive → push → stack = [10, 2]
-5 → negative, check top=2:
       |2| < |5| → 2 destroyed, pop → stack = [10]
     check top=10:
       |10| > |5| → -5 destroyed, stop
     stack = [10]
```

Final:
```text
[10]
```

### Equal-magnitude example

```text
asteroids = [8, -8]
```

Process:
```text
8  → push → stack = [8]
-8 → top=8, |8| == |8| → both destroyed, pop, don't push -8
stack = []
```

Final:
```text
[]
```

## Algorithm

1. Initialize an empty stack.
2. For each asteroid `a`:
   - If `a > 0`, push it (can never collide going in, only later).
   - Else (`a < 0`), set `alive = true` and loop:
     - While the stack is non-empty, top is positive, and `stack.top() < -a` (top smaller), pop the top (destroyed) and continue looping.
     - If the stack is non-empty and `stack.top() == -a`, pop the top and set `alive = false` (both destroyed).
     - Else if the stack is non-empty and `stack.top() > -a`, set `alive = false` (incoming destroyed).
     - If `alive`, push `a`.
3. Return the stack as the final array.

## Complexity

- **Time:** `O(n)` — each asteroid is pushed and popped at most once overall
- **Space:** `O(n)` — for the stack/result

## Notes / Tips

- Right-moving asteroids never need to look backward — only left-moving ones trigger collision checks, and only against right-movers sitting on top of the stack.
- A negative asteroid encountering a negative (or empty) stack top never collides — same direction or nothing to hit.
- Common mistake: forgetting the `alive` flag and pushing the incoming asteroid even after it was already destroyed in the popping loop.
- The `while` loop is what makes this a true monotonic stack — a single asteroid can wipe out multiple stack entries in one go.

## Code

```cpp
class Solution {
public:
    vector<int> asteroidCollision(vector<int>& asteroids) {
        vector<int> st;

        for (int a : asteroids) {
            bool alive = true;

            while (alive && a < 0 && !st.empty() && st.back() > 0) {
                if (st.back() < -a) {
                    st.pop_back();
                } else if (st.back() == -a) {
                    st.pop_back();
                    alive = false;
                } else {
                    alive = false;
                }
            }

            if (alive) {
                st.push_back(a);
            }
        }

        return st;
    }
};
```

---

# Key Template

When you see:

```text
Moving objects, opposite directions
+
Collisions destroy one or both
+
Cascading effect on prior elements
```

Think:

```text
Only opposing directions (right then left) can ever collide
        ↓
Push right-movers, resolve left-movers against the stack top in a loop
```


```text
stack = []

for a in asteroids:
    alive = true

    while alive and a < 0 and stack not empty and stack.top() > 0:
        if stack.top() < -a: stack.pop()
        elif stack.top() == -a: stack.pop(); alive = false
        else: alive = false

    if alive: stack.push(a)

return stack
```

## Pattern Recognition

The key observation is:

> **A left-moving asteroid can only collide with right-movers still sitting on the stack — same-direction or already-resolved elements never interact again.**
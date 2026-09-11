# LeetCode 4045 — Count Robot Groups

## Metadata

* **LeetCode:** 4045
* **Problem:** Count Robot Groups
* **Difficulty:** Medium
* **Topics:** Array, Stack, Simulation, Greedy
* **Pattern:** Right-to-Left Monotonic Merge Scan
* **Key Technique:** After a merge, a group's future merges only ever depend on the position and speed of the original *rightmost* robot in that group — so a single running "current group" reference (not a full stack) is enough
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given strictly increasing robot positions, their speeds, and a merge `distance`, robots (or groups) merge whenever the gap between adjacent ones becomes `<= distance`. A merged group always takes the position and speed of its **rightmost** member. Return the number of groups remaining after all possible merges (over unbounded continuous time).

---

## Approaches

1. **Brute Force — Event-Driven Merge Simulation**
2. **Optimal — Right-to-Left Single-Pass Scan**

---

# Approach 1 — Brute Force / Event-Driven Merge Simulation

## Idea

Directly simulate the physical process: repeatedly find the pair of currently-adjacent groups that will merge **soonest** (either immediately, if already within `distance`, or at a computable future time if the gap is shrinking), merge that pair (the result takes the rightmost's position and speed), and repeat until no adjacent pair can ever merge (every remaining gap is either constant or growing, and already `> distance`).

## Dry Run

```text
position = [1, 5, 6, 20], speed = [4, 3, 2, 3], distance = 1
```

Compute merge times for each adjacent pair:

```text
(0,1): gap=4, relSpeed=3-4=-1 (shrinking) → merges at t = (4-1)/1 = 3
(1,2): gap=1 <= distance=1 → merges at t = 0
(2,3): gap=14, relSpeed=3-2=1 (growing) → never merges
```

Earliest merge: pair `(1,2)` at `t=0`. Merge into rightmost's values `(6, 2)`:

```text
position = [1, 6, 20], speed = [4, 2, 3]
```

Recompute merge times:

```text
(0,1): gap=5, relSpeed=2-4=-2 (shrinking) → merges at t = (5-1)/2 = 2
(1,2): gap=14, relSpeed=3-2=1 (growing) → never merges
```

Earliest merge: pair `(0,1)` at `t=2`. Merge into rightmost's values `(6, 2)`:

```text
position = [6, 20], speed = [2, 3]
```

Recompute: `(0,1)`: gap=14, relSpeed=1 (growing) → never merges. No merges remain.

Final group count: `2`.

## Algorithm

1. Copy `position` and `speed` into mutable working arrays.
2. Repeat:

   * For every adjacent pair, compute whether and when they'd merge: immediately if `gap <= distance`, or at a future time if the gap is shrinking (`speed[right] < speed[left]`), otherwise never.
   * Find the pair with the smallest merge time.
   * If one exists, merge it (remove both entries, replace with the rightmost's position/speed).
   * If no pair can ever merge, stop.
3. Return the number of remaining entries.

## Complexity

* **Time:** `O(n²)`

  * Each merge requires rescanning all remaining adjacent pairs (`O(n)`), and up to `n - 1` merges can occur.
* **Space:** `O(n)`

  * For the mutable copies of `position` and `speed` that shrink as merges happen.

## Notes / Tips

* This mirrors the problem's physical description literally — useful for confirming correctness, but re-scanning all pairs after every single merge is wasteful, especially since most of the merge-time computations don't change between iterations.
* The `gap <= distance` check for immediate merging and the `speed[right] < speed[left]` check for eventual merging are the two building blocks reused directly in the optimal approach — this version just doesn't yet exploit the underlying monotonic structure to avoid rescanning.

## Code

```cpp
class Solution {
public:
    int countRobotGroups(vector<int>& position, vector<int>& speed, int distance) {
        vector<long long> pos(position.begin(), position.end());
        vector<long long> spd(speed.begin(), speed.end());

        while (true) {
            int bestIdx = -1;
            double bestTime = -1;

            for (int i = 0; i + 1 < (int)pos.size(); i++) {
                long long gap = pos[i + 1] - pos[i];
                long long relSpeed = spd[i + 1] - spd[i];

                double mergeTime;
                if (gap <= distance) {
                    mergeTime = 0.0;
                } else if (relSpeed < 0) {
                    mergeTime = (double)(gap - distance) / (-relSpeed);
                } else {
                    continue;
                }

                if (bestIdx == -1 || mergeTime < bestTime) {
                    bestTime = mergeTime;
                    bestIdx = i;
                }
            }

            if (bestIdx == -1) {
                break;
            }

            pos.erase(pos.begin() + bestIdx);
            spd.erase(spd.begin() + bestIdx);
        }

        return pos.size();
    }
};
```

---

# Approach 2 — Optimal / Right-to-Left Single-Pass Scan

## Idea

Process robots from right to left. Track the position and speed of the **current group boundary** (initially just the last robot). For each robot to its left, check whether it will ever merge with that group: either the gap is already `<= distance`, or the robot is faster than the group's speed (so it will eventually catch up, since the gap shrinks linearly forever). If it merges, it's simply absorbed — critically, since a merged group always takes the **rightmost** robot's position and speed, absorbing a robot from the left never changes the group's boundary values. If it doesn't merge, this robot becomes a brand-new group boundary, and the count increases.

## Dry Run

```text
position = [1, 5, 6, 20], speed = [4, 3, 2, 3], distance = 1
```

Start from the rightmost robot as the initial group:

```text
groupPos = 20, groupSpeed = 3, count = 1
```

`i=2` (pos=6, speed=2):

```text
gap = 20 - 6 = 14 > 1
speed[i]=2 > groupSpeed=3? no
→ new group: count = 2, groupPos = 6, groupSpeed = 2
```

`i=1` (pos=5, speed=3):

```text
gap = 6 - 5 = 1 <= 1 → merges immediately
→ absorbed, group stays (6, 2), count stays 2
```

`i=0` (pos=1, speed=4):

```text
gap = 6 - 1 = 5 > 1
speed[i]=4 > groupSpeed=2? yes → eventually merges
→ absorbed, group stays (6, 2), count stays 2
```

Final count: `2`, matching the brute-force result.

## Algorithm

1. Initialize `count = 1`, `groupPos = position[n-1]`, `groupSpeed = speed[n-1]`.
2. For `i` from `n - 2` down to `0`:

   * Check if robot `i` merges with the current group: `(groupPos - position[i] <= distance)` OR `(speed[i] > groupSpeed)`.
   * If it merges, do nothing (the group's boundary values are unaffected).
   * Otherwise, increment `count` and update `groupPos = position[i]`, `groupSpeed = speed[i]` (robot `i` becomes the new boundary).
3. Return `count`.

## Complexity

* **Time:** `O(n)`

  * A single right-to-left pass, constant work per robot.
* **Space:** `O(1)`

  * Only `count`, `groupPos`, and `groupSpeed` are tracked — no stack or extra arrays needed, since a merge never needs to "look further down" past the immediate group boundary.

## Notes / Tips

* This looks like it should need a full stack (as in Car Fleet, LC 853), but it doesn't: once a robot merges into a group, the group's representative position/speed never changes (it's always the original rightmost robot's values), so there's nothing to re-compare against further left — a single running variable suffices instead of a stack.
* The two merge conditions capture the entire physical model: `groupPos - position[i] <= distance` covers "already close enough to merge instantly," and `speed[i] > groupSpeed` covers "gap is strictly shrinking forever, so it merges eventually" — no other scenario leads to a merge, since equal or slower speed with a gap already `> distance` means the gap never closes.
* Common mistake: updating `groupSpeed`/`groupPos` to robot `i`'s own values when it merges — a merge must leave the group's boundary as the original rightmost robot's values, not the newly absorbed robot's.

## Code

```cpp
class Solution {
public:
    int countRobotGroups(vector<int>& position, vector<int>& speed, int distance) {
        int n = position.size();
        int count = 1;

        long long groupPos = position[n - 1];
        long long groupSpeed = speed[n - 1];

        for (int i = n - 2; i >= 0; i--) {
            bool merges = (groupPos - position[i] <= distance) || (speed[i] > groupSpeed);

            if (!merges) {
                count++;
                groupPos = position[i];
                groupSpeed = speed[i];
            }
        }

        return count;
    }
};
```

---

## Key Template

```text
count = 1
groupPos = position[n-1]
groupSpeed = speed[n-1]

for i from n-2 down to 0:
    merges = (groupPos - position[i] <= distance) or (speed[i] > groupSpeed)

    if not merges:
        count += 1
        groupPos = position[i]
        groupSpeed = speed[i]

return count
```
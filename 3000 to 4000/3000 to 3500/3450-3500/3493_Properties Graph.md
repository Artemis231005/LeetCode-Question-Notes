# LeetCode 3493 — Properties Graph

## Metadata

* **LeetCode:** 3493
* **Problem:** Properties Graph
* **Difficulty:** Medium
* **Topics:** Array, Hash Table, Bit Manipulation, Graph, Union Find
* **Pattern:** Pairwise Set Intersection + Connected Components (Union-Find)
* **Key Technique:** Since every value is bounded between `1` and `100`, represent each row as a fixed-size bitset instead of a general set — computing an intersection size then becomes a single word-level AND plus a population count, instead of comparing elements one by one
* **Optimal Complexity:** `O(n² * m / 64 + n * α(n))` Time, `O(n * 100)` Space

---

## Problem Statement

Given a 2D array `properties` of size `n x m` and an integer `k`, build an undirected graph where nodes `i` and `j` are connected exactly when the number of **distinct** values shared between `properties[i]` and `properties[j]` is at least `k`. Return the number of connected components in this graph.

---

## Approaches

1. **Brute Force — Nested-Loop Intersection + Union-Find**
2. **Optimal — Bitset Intersection + Union-Find**

---

# Approach 1 — Brute Force / Nested-Loop Intersection + Union-Find

## Idea

For every pair of rows, compute their intersection size directly by comparing elements pairwise with nested loops (tracking which values have already been counted, to correctly handle duplicates within a row). Whenever a pair's intersection meets `k`, union their indices using a Union-Find structure. After checking all pairs, count the number of distinct groups.

## Dry Run

```text
properties = [[1,2],[1,1],[3,4],[4,5],[5,6],[7,7]], k = 1
```

Compare row `0` (`[1,2]`) and row `1` (`[1,1]`):

```text
for each value in row 0, check if it appears in row 1, counting each distinct match once:
   1 appears in row 1 → distinct match: {1}
   2 does not appear in row 1
intersection size = 1 >= k=1 → union(0, 1)
```

Compare row `0` and row `2` (`[3,4]`):

```text
1 not in row 2, 2 not in row 2 → intersection size = 0 < 1 → no edge
```

Continue checking all `C(6,2)=15` pairs, union-ing whenever the intersection meets `k=1`. After all unions, count distinct roots among the 6 nodes.

Final: `3` connected components, matching the expected output.

## Algorithm

1. Initialize a Union-Find structure with each row as its own group.
2. For each pair `(i, j)` with `i < j`:

   * Compute their distinct intersection size using nested loops (comparing each element of row `i` against every element of row `j`, tracking already-counted values to avoid double-counting duplicates).
   * If the intersection size `>= k`, union `i` and `j`.
3. Count the number of distinct roots among all `n` rows.
4. Return that count.

## Complexity

* **Time:** `O(n² * m²)`

  * `O(n²)` pairs, each requiring an `O(m²)` nested-loop comparison to compute the distinct intersection size.
* **Space:** `O(n)`

  * For the Union-Find `parent` array (plus small temporary sets per comparison).

## Notes / Tips

* Comparing every element of one row against every element of another with nested loops is the main bottleneck — converting each row into a proper set-like structure first (a bitset, given the small bounded value range) turns each pairwise intersection into a much cheaper operation.
* Since `properties[i][j]` values are bounded to `1..100` regardless of `m`, this fixed range is exactly what makes the bitset optimization in Approach 2 possible.

## Code

```cpp
class Solution {
public:
    vector<int> parent;

    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

    void unite(int x, int y) {
        int rootX = find(x), rootY = find(y);
        if (rootX != rootY) parent[rootX] = rootY;
    }

    int intersectCount(vector<int>& a, vector<int>& b) {
        set<int> counted;
        for (int val : a) {
            if (counted.count(val)) continue;

            for (int other : b) {
                if (other == val) {
                    counted.insert(val);
                    break;
                }
            }
        }
        return counted.size();
    }

    int numberOfComponents(vector<vector<int>>& properties, int k) {
        int n = properties.size();
        parent.resize(n);
        for (int i = 0; i < n; i++) parent[i] = i;

        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                if (intersectCount(properties[i], properties[j]) >= k) {
                    unite(i, j);
                }
            }
        }

        int components = 0;
        for (int i = 0; i < n; i++) {
            if (find(i) == i) components++;
        }

        return components;
    }
};
```

---

# Approach 2 — Optimal / Bitset Intersection + Union-Find

## Idea

Since every value in `properties` is guaranteed to fall within `1..100`, represent each row as a fixed-size `bitset<101>`, with bit `v` set if value `v` appears anywhere in that row. The distinct intersection size between two rows is then just `(bitsetA & bitsetB).count()` — a single word-level AND operation followed by a population count, computed in roughly `O(m/64)` time instead of comparing elements one by one. Use this fast intersection check inside the same Union-Find structure as Approach 1.

## Dry Run

```text
properties = [[1,2],[1,1],[3,4],[4,5],[5,6],[7,7]], k = 1
```

Build bitsets:

```text
row0 bits: {1,2}
row1 bits: {1}       (duplicate 1s collapse into a single bit)
row2 bits: {3,4}
row3 bits: {4,5}
row4 bits: {5,6}
row5 bits: {7}
```

Intersect row0 & row1: `{1,2} & {1} = {1}` → count `1 >= k=1` → union(0,1).
Intersect row0 & row2: `{1,2} & {3,4} = {}` → count `0` → no edge.
Intersect row2 & row3: `{3,4} & {4,5} = {4}` → count `1 >= 1` → union(2,3).
Intersect row3 & row4: `{4,5} & {5,6} = {5}` → count `1 >= 1` → union(3,4).
Intersect row5 with everything else: no shared value `7` anywhere else → stays isolated.

Groups after all unions: `{0,1}`, `{2,3,4}`, `{5}` → `3` connected components, matching the brute-force result.

## Algorithm

1. Build an array of `bitset<101>` (or similar fixed-size bit structure), one per row, setting a bit for each value present in that row.
2. Initialize a Union-Find structure with each row as its own group.
3. For each pair `(i, j)` with `i < j`:

   * Compute `(bitset[i] & bitset[j]).count()`.
   * If that count is `>= k`, union `i` and `j`.
4. Count the number of distinct roots among all `n` rows.
5. Return that count.

## Complexity

* **Time:** `O(n * m + n² * m / 64)`

  * `O(n * m)` to build all the bitsets; `O(n²)` pairs, each intersection costing roughly `O(m / 64)` thanks to word-level bitset operations (up to `O(101/64) ≈ O(2)` words per intersection, effectively constant given the fixed value range).
* **Space:** `O(n * 100)`

  * For storing one fixed-size bitset per row.

## Notes / Tips

* This optimization is only possible because the value range is small and fixed (`1..100`) — a `bitset<101>` (or equivalently, a 2-word `unsigned long long` pair) captures a row's distinct values in constant space, and hardware-level bitwise AND plus population count make intersection nearly free compared to explicit element-by-element comparison.
* Deduplication (handling repeated values within a single row, like the two `1`s in `[1,1]`) comes for free with a bitset — setting the same bit twice has no effect, unlike the nested-loop approach which needed an explicit "already counted" tracker.
* This is the same Union-Find-for-connected-components template as LC 547 and LC 1971 — the only new idea here is using a bitset to make the pairwise "should these be connected?" check as cheap as possible given the problem's specific value constraints.

## Code

```cpp
class Solution {
public:
    vector<int> parent;

    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

    void unite(int x, int y) {
        int rootX = find(x), rootY = find(y);
        if (rootX != rootY) parent[rootX] = rootY;
    }

    int numberOfComponents(vector<vector<int>>& properties, int k) {
        int n = properties.size();
        vector<bitset<101>> bits(n);

        for (int i = 0; i < n; i++) {
            for (int val : properties[i]) {
                bits[i].set(val);
            }
        }

        parent.resize(n);
        for (int i = 0; i < n; i++) parent[i] = i;

        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                if ((bits[i] & bits[j]).count() >= (size_t)k) {
                    unite(i, j);
                }
            }
        }

        int components = 0;
        for (int i = 0; i < n; i++) {
            if (find(i) == i) components++;
        }

        return components;
    }
};
```

---

## Key Template

```text
bits[i] = bitset<101>, set bit v for every value v in properties[i]

parent = [0, 1, ..., n-1]

function find(x):
    if parent[x] != x: parent[x] = find(parent[x])
    return parent[x]

function unite(x, y):
    rootX = find(x); rootY = find(y)
    if rootX != rootY: parent[rootX] = rootY

for i in 0..n-1:
    for j in i+1..n-1:
        if (bits[i] & bits[j]).count() >= k:
            unite(i, j)

count distinct find(i) for i in 0..n-1
return count
```
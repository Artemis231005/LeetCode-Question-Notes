# LeetCode 133 — Clone Graph

## Metadata

* **LeetCode:** 133
* **Problem:** Clone Graph
* **Difficulty:** Medium
* **Topics:** Hash Table, Depth-First Search, Breadth-First Search, Graph
* **Pattern:** Graph Traversal with a Visited/Clone Map
* **Key Technique:** Map each original node to its clone the first time it's encountered, so revisiting a node (via a cycle or a different path) reuses the existing clone instead of creating a duplicate or recursing infinitely
* **Optimal Complexity:** `O(V + E)` Time, `O(V)` Auxiliary Space

---

## Problem Statement

Given a reference to a node in a connected undirected graph, return a **deep copy** (clone) of the graph. Each node contains a value and a list of references to its neighbors.

---

## Approaches

1. **Brute Force — Naive Recursive Clone Without a Visited Map**
2. **Optimal — DFS (or BFS) with a Hash Map from Original to Clone**

---

# Approach 1 — Brute Force / Naive Recursive Clone Without a Visited Map

## Idea

Try to clone the graph by recursively creating a new node for the current one, then recursively cloning each of its neighbors and attaching them. This seems reasonable at first glance, but without tracking which original nodes have already been cloned, it breaks down as soon as the graph contains a cycle (which it generally does, since edges are undirected — even a single edge `A-B` means `A` is a neighbor of `B` and vice versa).

## Dry Run

```text
graph: 1 - 2 (a single edge, undirected)
```

Clone node `1`:

```text
create clone1
clone1's neighbors: clone the neighbor 2
    create clone2
    clone2's neighbors: clone the neighbor 1 (since edge is undirected, 2's neighbor list includes 1)
        create clone1_again (a NEW node, not reusing the original clone1!)
        clone1_again's neighbors: clone the neighbor 2
            create clone2_again ...
            ... this recurses forever
```

This never terminates — every "clone" of a node triggers cloning its neighbor, which triggers cloning it right back, endlessly creating new node objects.

## Algorithm

1. Define a recursive `cloneNode(node)`:

   * Create a new node with `node->val`.
   * For each neighbor in `node->neighbors`, recursively call `cloneNode(neighbor)` and add the result to the new node's neighbor list.
   * Return the new node.
2. Call `cloneNode(root)`.

## Complexity

* **Time:** Does not terminate (infinite recursion) on any graph containing a cycle — which includes any graph with more than one node, since edges are undirected.
* **Space:** Unbounded (grows without limit until a stack overflow occurs).

## Notes / Tips

* This isn't just "slower" but it's **broken** for any graph with a cycle, which is essentially guaranteed here since edges are bidirectional. It's included to highlight exactly why a visited/clone-tracking structure isn't optional, but a hard requirement for correctness.
* Even setting aside infinite recursion, this approach would also create multiple distinct clone objects for the same original node if there were multiple paths to it (e.g. a diamond-shaped graph `A-B`, `A-C`, `B-D`, `C-D`), corrupting the graph structure by duplicating nodes that should be shared.
* The fix is simple and standard: a hash map from original node to its already-created clone, checked *before* creating a new clone for any node.

## Code

```cpp
class Solution {
public:
    Node* cloneNode(Node* node) {
        // BROKEN: no visited/clone tracking — infinite recursion on any cycle.
        Node* newNode = new Node(node->val);

        for (Node* neighbor : node->neighbors) {
            newNode->neighbors.push_back(cloneNode(neighbor)); // recurses forever
        }

        return newNode;
    }

    Node* cloneGraph(Node* node) {
        if (!node) return nullptr;
        return cloneNode(node); // never actually returns for graphs with cycles
    }
};
```

---

# Approach 2 — Optimal / DFS (or BFS) with a Hash Map from Original to Clone

## Idea

Maintain a hash map from each original node to its corresponding clone. Traverse the graph (via DFS or BFS) starting from the given node. The first time a node is encountered, create its clone and store it in the map immediately — **before** recursing into its neighbors — so that if a neighbor's traversal leads back to this same node (via a cycle), the existing clone is reused instead of triggering another clone or infinite recursion.

## Dry Run

```text
graph: 1 - 2 (a single edge, undirected)
```

DFS from node `1`:

```text
1 not in map → create clone1, map = {1: clone1}
process 1's neighbors: [2]
    2 not in map → create clone2, map = {1: clone1, 2: clone2}
    process 2's neighbors: [1]
        1 IS in map → return existing clone1 (no new node created, no further recursion)
    clone2's neighbors = [clone1]
clone1's neighbors = [clone2]
```

Traversal terminates cleanly. Final cloned graph: `clone1 <-> clone2`, correctly mirroring the original.

## Algorithm

1. If `node` is null, return null.
2. Initialize an empty hash map `cloneMap` (original node → clone).
3. Define a recursive `dfs(node)`:

   * If `node` is already in `cloneMap`, return `cloneMap[node]` immediately (already cloned, or currently being cloned).
   * Otherwise, create a new clone, store it in `cloneMap[node]` **before** processing neighbors.
   * For each neighbor of `node`, recursively call `dfs(neighbor)` and append the result to the clone's neighbor list.
   * Return the clone.
4. Return `dfs(node)`.

## Complexity

* **Time:** `O(V + E)`

  * Each node is cloned exactly once (subsequent visits are `O(1)` map lookups), and each edge is processed once from each direction.
* **Space:** `O(V)`

  * For the `cloneMap` (storing one entry per original node) and the recursion stack (up to `O(V)` deep in the worst case).

## Notes / Tips

* Storing the new clone in `cloneMap` **before** recursing into its neighbors is the critical ordering detail — this is what breaks the infinite recursion from Approach 1: when a neighbor's traversal leads back to the current node, it finds the (possibly still being filled in) clone already in the map and stops, rather than trying to clone it again.
* BFS works equally well here with the same core idea (map before enqueueing/expanding neighbors). The choice between DFS and BFS is purely stylistic for this problem, since both achieve the same `O(V + E)` complexity.
* This "map an object to its transformed counterpart the moment it's first seen, before processing its dependencies" pattern generalizes to any deep-copy problem involving cyclic or shared references — not just graphs (e.g. cloning a linked list with random pointers, LC 138, uses the exact same idea).

## Code

```cpp
class Solution {
public:
    unordered_map<Node*, Node*> cloneMap;

    Node* dfs(Node* node) {
        if (cloneMap.find(node) != cloneMap.end()) {
            return cloneMap[node];
        }

        Node* clone = new Node(node->val);
        cloneMap[node] = clone;

        for (Node* neighbor : node->neighbors) {
            clone->neighbors.push_back(dfs(neighbor));
        }

        return clone;
    }

    Node* cloneGraph(Node* node) {
        if (!node) {
            return nullptr;
        }

        return dfs(node);
    }
};
```

---

## Key Template

```text
cloneMap = {}

function dfs(node):
    if node in cloneMap:
        return cloneMap[node]

    clone = new Node(node.val)
    cloneMap[node] = clone     # map BEFORE recursing — breaks cycles

    for neighbor in node.neighbors:
        clone.neighbors.append(dfs(neighbor))

    return clone

if node is null: return null
return dfs(node)
```
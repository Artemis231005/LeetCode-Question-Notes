# 108. Convert Sorted Array to Binary Search Tree

## Metadata

* **Topic:** Binary Tree, Binary Search Tree, Divide & Conquer
* **Difficulty:** Easy
* **Pattern:** Divide & Conquer
* **Key Pattern:** Middle element as root
* **Key Template:** Recursive divide-and-conquer on a sorted range

---

## Idea

We are given a **sorted array in ascending order** and need to construct a **height-balanced Binary Search Tree (BST)**.

For a BST:

* All elements smaller than the root go to the **left subtree**.
* All elements greater than the root go to the **right subtree**.
* To keep the tree balanced, choose the **middle element** as the root.

For every subarray:

1. Pick its middle element as the root.
2. Recursively build the left subtree from the left half.
3. Recursively build the right subtree from the right half.

This is naturally solved using **recursion + divide and conquer**.

---

## Dry Run

### Example

```text
nums = [-10, -3, 0, 5, 9]
```

Middle element:

```text
          0
        /   \
      -10    5
        \     \
        -3     9
```

Steps:

```text
[-10, -3, 0, 5, 9]
          ↑
        root = 0

Left half:  [-10, -3]
Right half: [5, 9]
```

For `[-10, -3]`:

```text
mid = -3
      -3
     /
   -10
```

For `[5, 9]`:

```text
mid = 9
     9
    /
   5
```

Final tree:

```text
        0
       / \
     -3   9
    /    /
  -10   5
```

The exact balanced BST can differ depending on which middle element is chosen when the subarray has an even number of elements.

---

## Algorithm

1. If the current range is empty (`left > right`), return `nullptr`.
2. Find the middle index:

   ```cpp
   mid = left + (right - left) / 2;
   ```
3. Create a TreeNode using `nums[mid]`.
4. Recursively construct the left subtree using:

   ```text
   left ... mid - 1
   ```
5. Recursively construct the right subtree using:

   ```text
   mid + 1 ... right
   ```
6. Return the root.

---

## Complexity

* **Time:** `O(n)` — every array element is used exactly once.
* **Space:** `O(log n)` recursion stack for a balanced tree.
* **Auxiliary Space:** `O(log n)` excluding the output tree.

---

## Notes / Tips

* The array is already **sorted**, so the middle element is the natural choice for maintaining balance.
* Do **not** create separate left/right arrays; use indices instead.
* Use:

  ```cpp
  left + (right - left) / 2
  ```

  to calculate the middle safely.
* The recursion is performed on **index ranges**, not on copied arrays.
* For an even-sized range, either middle can produce a valid balanced BST.
* This is a classic **divide-and-conquer** pattern:

  ```text
  solve(left, right)
      ↓
  choose middle
      ↓
  solve(left, mid - 1)
      ↓
  solve(mid + 1, right)
  ```

---

## Code

```cpp
class Solution {
public:
    TreeNode* build(vector<int>& nums, int left, int right) {
        if (left > right) {
            return nullptr;
        }

        int mid = left + (right - left) / 2;

        TreeNode* root = new TreeNode(nums[mid]);

        root->left = build(nums, left, mid - 1);
        root->right = build(nums, mid + 1, right);

        return root;
    }

    TreeNode* sortedArrayToBST(vector<int>& nums) {
        return build(nums, 0, nums.size() - 1);
    }
};
```

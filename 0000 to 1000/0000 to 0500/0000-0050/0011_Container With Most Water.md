# LeetCode 11 — Container With Most Water

## Metadata

* **LeetCode:** 11
* **Problem:** Container With Most Water
* **Difficulty:** Medium
* **Topics:** Array, Two Pointers, Greedy
* **Pattern:** Two Pointers — Opposite Ends
* **Key Pattern:** Start with the maximum width and move the pointer at the shorter line
* **Key Template:** Two Pointers — Opposite Ends
* **Key Technique:** Maximize `min(height[left], height[right]) × (right - left)`
* **Optimal Complexity:** `O(n)` time, `O(1)` space

The problem asks for the maximum area formed by two vertical lines. The area is `min(height[i], height[j]) × (j - i)`.

---

# Approaches

1. **Brute Force — Check Every Pair**
2. **Optimal — Two Pointers**

---

# Approach 1 — Brute Force / Check Every Pair

## Idea

Try every possible pair of lines.

For every pair `(i, j)`, calculate:

```text
area = min(height[i], height[j]) × (j - i)
```

Keep track of the maximum area.

## Dry Run

```text
height = [1, 8, 6, 2, 5, 4, 8, 3, 7]
```

Consider:

```text
i = 1 → height = 8
j = 8 → height = 7
```

Width:

```text
8 - 1 = 7
```

Height:

```text
min(8, 7) = 7
```

Area:

```text
7 × 7 = 49
```

So the current maximum is:

```text
49
```

## Algorithm

1. Initialize `maxArea = 0`.
2. For every index `i`:

   * Check every index `j > i`.
3. Calculate:

   ```text
   area = min(height[i], height[j]) × (j - i)
   ```
4. Update `maxArea`.
5. Return `maxArea`.

## Complexity

* **Time:** `O(n²)`
* **Space:** `O(1)`

## Notes / Tips

* This checks every possible pair.
* It is straightforward but too slow for `n = 10^5`.

## Code

```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int maxArea = 0;

        for (int i = 0; i < height.size(); i++) {
            for (int j = i + 1; j < height.size(); j++) {
                int width = j - i;
                int h = min(height[i], height[j]);

                maxArea = max(maxArea, width * h);
            }
        }

        return maxArea;
    }
};
```

---

# Approach 2 — Optimal / Two Pointers

## Idea

Start with the two lines that give the **maximum possible width**:

```text
left = 0
right = n - 1
```

Calculate the current area.

The important observation is:

> The area is limited by the **shorter line**.

Suppose:

```text
height[left] < height[right]
```

The current height is `height[left]`.

If we move `right` inward:

* Width decreases.
* The limiting height is still at most `height[left]`.

Therefore, moving the taller line cannot produce a better container while the shorter line remains fixed.

So we **must move the shorter pointer** and look for a taller line.

This is the key greedy observation behind the `O(n)` solution.

## Dry Run

```text
height = [1,8,6,2,5,4,8,3,7]
```

Start:

```text
left = 0
right = 8
```

Values:

```text
1 and 7
```

Area:

```text
min(1, 7) × (8 - 0)
= 1 × 8
= 8
```

`height[left]` is smaller, so move `left`.

---

Now:

```text
left = 1
right = 8
```

Values:

```text
8 and 7
```

Area:

```text
7 × 7 = 49
```

Maximum:

```text
49
```

Since the right line is shorter:

```text
right--
```

Continue until:

```text
left >= right
```

The maximum remains:

```text
49
```

## Algorithm

1. Initialize:

   ```text
   left = 0
   right = n - 1
   maxArea = 0
   ```
2. While `left < right`:

   * Calculate:

     ```text
     width = right - left
     h = min(height[left], height[right])
     area = width × h
     ```
   * Update `maxArea`.
3. Move the pointer pointing to the shorter line:

   * If:

     ```text
     height[left] < height[right]
     ```

     move `left++`.
   * Otherwise, move `right--`.
4. Return `maxArea`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

Each pointer only moves inward, so the array is traversed at most once.

## Notes / Tips

* The formula to remember:

  ```text
  area = min(height[left], height[right]) × (right - left)
  ```
* Start from **opposite ends**.
* Always move the **shorter** line.
* Do not move the taller line first: reducing the width while keeping the same limiting height cannot improve the area.
* If both heights are equal, moving either pointer is valid.
* This is a classic **Two Pointers + Greedy** problem.

## Code

```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int left = 0;
        int right = height.size() - 1;
        int maxArea = 0;

        while (left < right) {
            int width = right - left;
            int h = min(height[left], height[right]);

            int area = width * h;
            maxArea = max(maxArea, area);

            if (height[left] < height[right]) {
                left++;
            }
            else {
                right--;
            }
        }

        return maxArea;
    }
};
```

---

# Approach Comparison

| Approach     |    Time |  Space | Status      |
| ------------ | ------: | -----: | ----------- |
| Brute Force  | `O(n²)` | `O(1)` | Brute       |
| Two Pointers |  `O(n)` | `O(1)` | **Optimal** |

---

# Key Template

### Two Pointers — Opposite Ends

```cpp
int left = 0;
int right = n - 1;

while (left < right) {
    // Calculate current answer

    if (condition_to_move_left) {
        left++;
    }
    else {
        right--;
    }
}
```

### For This Problem

```cpp
while (left < right) {
    int area = min(height[left], height[right]) * (right - left);

    maxArea = max(maxArea, area);

    if (height[left] < height[right]) {
        left++;
    }
    else {
        right--;
    }
}
```

## Final Takeaway

The progression is:

```text
Brute:
Try every pair
        ↓
Optimal:
Start with maximum width
        ↓
Area is limited by the shorter line
        ↓
Move the shorter pointer
        ↓
Find a potentially taller boundary
```

The main pattern to recognize is:

> **When maximizing an area between two boundaries, start from opposite ends and eliminate the shorter boundary.**

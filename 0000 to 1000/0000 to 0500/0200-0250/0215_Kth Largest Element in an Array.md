# 215. Kth Largest Element in an Array

## Metadata

* **Topic:** Array, Heap, Sorting
* **Difficulty:** Medium
* **Pattern:** Min Heap
* **Key Pattern:** Maintain a min heap of size `k`.

---

## Idea

We need to find the **kth largest** element.

Instead of sorting the entire array, maintain a **min heap containing the `k` largest elements** seen so far.

Why a min heap?

The smallest element among these `k` elements is exactly the **kth largest element**.

For every number:

* Push it into the heap.
* If heap size becomes greater than `k`, remove the smallest element.

At the end:

```text
heap.top() = kth largest element
```

---

## Dry Run

```text
nums = [3,2,1,5,6,4]
k = 2
```

Maintain a min heap of size `2`.

```text
3 → [3]
2 → [2,3]
1 → [2,3,1] → remove 1 → [2,3]
5 → [2,3,5] → remove 2 → [3,5]
6 → [3,5,6] → remove 3 → [5,6]
4 → [4,5,6] → remove 4 → [5,6]
```

Finally:

```text
heap.top() = 5
```

So the **2nd largest** element is `5`.

---

## Algorithm

1. Create an empty min heap.
2. Traverse every element in `nums`.
3. Push the current element into the heap.
4. If heap size exceeds `k`, remove the smallest element.
5. Return `heap.top()`.

---

## Complexity

* **Time:** `O(n log k)`
* **Space:** `O(k)`

---

## Notes / Tips

### Why Min Heap?

For the `k` largest elements:

```text
[largest ... kth largest]
             ↑
          smallest
          in heap
```

Therefore, the root of the min heap is the answer.

### Alternative

You can sort:

```text
sort(nums.begin(), nums.end());
return nums[n - k];
```

Complexity:

```text
O(n log n)
```

Heap is better when `k` is much smaller than `n`.

### Key Template

```text
minHeap

for each element:
    push(element)

    if heap.size() > k:
        pop()

return heap.top()
```

---

## Code

```cpp
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        priority_queue<int, vector<int>, greater<int>> minHeap;

        for (int num : nums) {
            minHeap.push(num);

            if (minHeap.size() > k) {
                minHeap.pop();
            }
        }

        return minHeap.top();
    }
};
```

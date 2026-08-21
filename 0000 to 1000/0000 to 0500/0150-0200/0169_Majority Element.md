# 169. Majority Element

## Metadata

* **Topic:** Array, Hashing, Greedy
* **Difficulty:** Easy
* **Pattern:** Boyer-Moore Voting Algorithm
* **Key Pattern:** Cancel out different elements
* **Key Template:** Candidate + Count

---

## Idea

We need to find the element that appears **more than `n / 2` times**.

The key observation is:

> Since the majority element appears more than half the time, it cannot be completely cancelled out by all the other elements.

Use the **Boyer-Moore Voting Algorithm**:

* Maintain a `candidate`.
* Maintain a `count`.
* If `count == 0`, choose the current element as the new candidate.
* If the current element equals the candidate, increase `count`.
* Otherwise, decrease `count`.

Think of it as **cancelling pairs of different elements**.

Because the majority element occurs more than all other elements combined, it will survive the cancellation.

---

## Dry Run

### Example

```text
nums = [2, 2, 1, 1, 1, 2, 2]
```

Start:

```text
candidate = -
count = 0
```

| Element | Candidate | Count | Action                   |
| ------- | --------: | ----: | ------------------------ |
| `2`     |       `2` |   `1` | `count == 0`, choose `2` |
| `2`     |       `2` |   `2` | Same → `count++`         |
| `1`     |       `2` |   `1` | Different → `count--`    |
| `1`     |       `2` |   `0` | Different → `count--`    |
| `1`     |       `1` |   `1` | `count == 0`, choose `1` |
| `2`     |       `1` |   `0` | Different → `count--`    |
| `2`     |       `2` |   `1` | `count == 0`, choose `2` |

Final:

```text
candidate = 2
```

Therefore, the majority element is:

```text
2
```

---

## Algorithm

1. Initialize:

   ```text
   candidate = 0
   count = 0
   ```
2. Traverse the array.
3. If `count == 0`, set the current element as `candidate`.
4. If the current element equals `candidate`, increment `count`.
5. Otherwise, decrement `count`.
6. Return `candidate`.

---

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

---

## Notes / Tips

* The problem guarantees that a majority element **exists**.
* Therefore, we do **not** need a final verification step.
* The algorithm works because:

  ```text
  majority count > n / 2
  ```
* Every different element can cancel one occurrence of the majority element.
* Even after all possible cancellations, the majority element remains.
* This is much better than using a frequency map when `O(1)` extra space is required.

### Important Pattern

Whenever you see:

```text
Find element appearing > n/2 times
```

immediately think:

```text
Boyer-Moore Voting Algorithm
```

---

## Key Template

```text
candidate = 0
count = 0

for each element:
    if count == 0:
        candidate = element

    if element == candidate:
        count++
    else:
        count--

return candidate
```

---

## Code

```cpp
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int candidate = 0;
        int count = 0;

        for (int num : nums) {
            if (count == 0) {
                candidate = num;
            }

            if (num == candidate) {
                count++;
            }
            else {
                count--;
            }
        }

        return candidate;
    }
};
```

# 278. First Bad Version

## Metadata

* **Topic:** Binary Search
* **Difficulty:** Easy
* **Pattern:** Binary Search on Monotonic Property
* **Key Pattern:** Find the first position where `isBadVersion()` becomes `true`.

---

## Idea

Versions are arranged like:

```text
Good → Good → Good → Bad → Bad → Bad
```

Once a version is bad, **every version after it is also bad**.

So this is a classic **"first occurrence" binary search**.

For `mid`:

* If `isBadVersion(mid) == true`:

  * `mid` could be the first bad version.
  * Search left:

    ```text
    high = mid
    ```

* If `isBadVersion(mid) == false`:

  * `mid` is definitely not the answer.
  * Search right:

    ```text
    low = mid + 1
    ```

---

## Dry Run

Suppose:

```text
n = 5
first bad version = 4
```

```text
Version:  1  2  3  4  5
Status:   G  G  G  B  B
```

Start:

```text
low = 1
high = 5
```

### Step 1

```text
mid = 3
```

`3` is good.

Therefore:

```text
low = 4
```

### Step 2

```text
mid = 4
```

`4` is bad.

It could be the first bad version:

```text
high = 4
```

Now:

```text
low == high == 4
```

Answer:

```text
4
```

---

## Algorithm

1. Set `low = 1`, `high = n`.
2. While `low < high`:

   * Calculate `mid`.
   * If `mid` is bad:

     ```text
     high = mid
     ```
   * Otherwise:

     ```text
     low = mid + 1
     ```
3. Return `low`.

---

## Complexity

* **Time:** `O(log n)` calls to `isBadVersion()`.
* **Space:** `O(1)`.

---

## Code

```cpp
class Solution {
public:
    int firstBadVersion(int n) {
        int low = 1;
        int high = n;

        while (low < high) {
            int mid = low + (high - low) / 2;

            if (isBadVersion(mid)) {
                high = mid;
            }
            else {
                low = mid + 1;
            }
        }

        return low;
    }
};
```

---

## Notes / Tips

* This is **not** ordinary binary search for an exact value.
* We are finding the **first `true`** in a monotonic sequence.
* If `mid` is bad, keep `mid` because it might be the answer:

  ```text
  high = mid
  ```
* If `mid` is good, discard it:

  ```text
  low = mid + 1
  ```
* Use:

  ```text
  low + (high - low) / 2
  ```

  to avoid integer overflow.

### Key Pattern

```text
false false false true true true
                  ↑
             first true
```

---

## Key Template

```text
low = first possible index
high = last possible index

while (low < high):
    mid = low + (high - low) / 2

    if condition(mid) is true:
        high = mid
    else:
        low = mid + 1

return low
```

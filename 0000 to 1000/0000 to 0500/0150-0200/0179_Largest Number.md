# 179. Largest Number

## Metadata

* **Topic:** Array, Sorting
* **Difficulty:** Medium
* **Pattern:** Custom Comparator
* **Key Pattern:** For two numbers `a` and `b`, compare `a + b` with `b + a`.

---

## Idea

We need to arrange the numbers such that their concatenation forms the **largest possible number**.

Normal sorting does not work.

For two numbers `a` and `b`:

```text
a before b  →  "ab"
b before a  →  "ba"
```

Place `a` before `b` if:

```text
"ab" > "ba"
```

Example:

```text
a = 3, b = 30

"330" > "303"
```

So `3` should come before `30`.

After sorting, concatenate all numbers.

### Edge Case

If the result starts with `0`, all numbers were zero.

Return:

```text
"0"
```

---

## Dry Run

```text
nums = [3, 30, 34, 5, 9]
```

Custom sorting gives:

```text
[9, 5, 34, 3, 30]
```

Concatenate:

```text
9 + 5 + 34 + 3 + 30
= "9534330"
```

So the answer is:

```text
"9534330"
```

---

## Algorithm

1. Convert numbers to strings.
2. Sort using the comparator:

   ```text
   a + b > b + a
   ```
3. Concatenate the sorted strings.
4. If the first character is `'0'`, return `"0"`.
5. Otherwise return the concatenated result.

---

## Complexity

* **Time:** `O(n log n · k)` where `k` is the average number of digits.
* **Space:** `O(nk)` for storing strings.

---

## Notes / Tips

* This is **not normal descending numerical sorting**.
* The comparator is the main trick:

  ```text
  a + b > b + a
  ```
* Example:

  ```text
  [10, 2] → "210"
  ```

  because `"210" > "102"`.
* Return `"0"` instead of `"000..."` when all values are zero.
* `sort()` must use a comparator that defines the desired ordering.

---

## Key Template

```text
Convert numbers → strings

sort(a, b):
    return a + b > b + a

concatenate all strings
```

---

## Code

```cpp
class Solution {
public:
    string largestNumber(vector<int>& nums) {
        vector<string> arr;

        for (int num : nums) {
            arr.push_back(to_string(num));
        }

        sort(arr.begin(), arr.end(), [](string& a, string& b) {
            return a + b > b + a;
        });

        if (arr[0] == "0") {
            return "0";
        }

        string ans;

        for (string& s : arr) {
            ans += s;
        }

        return ans;
    }
};
```

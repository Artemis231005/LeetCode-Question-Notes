# 217. Contains Duplicate

## Metadata

* **Topic:** Array, Hash Set
* **Difficulty:** Easy
* **Pattern:** Hashing
* **Key Pattern:** If an element already exists in a set, it is a duplicate.

---

## Idea

We need to determine whether the array contains **any duplicate value**.

Use an `unordered_set` to store elements we have already seen.

For every element:

* If it is already in the set → duplicate exists → return `true`.
* Otherwise, insert it into the set.

If we finish the array without finding a duplicate, return `false`.

---

## Dry Run

```text
nums = [1,2,3,1]
```

Process:

```text
1 → set = {1}
2 → set = {1,2}
3 → set = {1,2,3}
1 → already exists → duplicate found
```

Answer:

```text
true
```

---

## Algorithm

1. Create an empty hash set.
2. Traverse the array.
3. If the current element already exists in the set, return `true`.
4. Otherwise, insert it.
5. If traversal finishes, return `false`.

---

## Complexity

* **Time:** `O(n)` average
* **Space:** `O(n)`

---

## Notes / Tips

### Key Template

```text
set = {}

for each element:
    if element exists in set:
        return true

    insert element

return false
```

You can also solve it by sorting:

```text
sort(nums)
```

and checking adjacent elements, but that takes `O(n log n)` time.

---

## Code

```cpp
class Solution {
public:
    bool containsDuplicate(vector<int>& nums) {
        unordered_set<int> seen;

        for (int num : nums) {
            if (seen.count(num)) {
                return true;
            }

            seen.insert(num);
        }

        return false;
    }
};
```

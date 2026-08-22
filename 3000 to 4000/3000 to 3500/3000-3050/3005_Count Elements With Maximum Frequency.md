# Count Elements With Maximum Frequency

## Problem

Given an integer array `nums`, return the **total number of elements** that appear with the **maximum frequency**.

Example:

```text
nums = [1,2,2,3,1,4]

Frequency:
1 → 2
2 → 2
3 → 1
4 → 1

Maximum frequency = 2

Elements with maximum frequency:
1 → 2 occurrences
2 → 2 occurrences

Answer = 2 + 2 = 4
```

---

## Approach 1: Frequency Map

### Idea

First count the frequency of every element.

Then find the maximum frequency.

Finally, add the frequencies of all elements whose frequency equals the maximum frequency.

### Dry Run

```text
nums = [1,2,2,3,1,4]

Frequency map:
1 → 2
2 → 2
3 → 1
4 → 1

maxFrequency = 2

1 has frequency 2 → add 2
2 has frequency 2 → add 2

Answer = 4
```

### Algorithm

1. Create a frequency map.
2. Traverse `nums` and count each element.
3. Find the maximum frequency.
4. Traverse the frequency map.
5. Add every frequency that equals the maximum frequency.
6. Return the total.

### Complexity

* Time: `O(n)` average.
* Space: `O(n)`.

### Code

```cpp
class Solution {
public:
    int maxFrequencyElements(vector<int>& nums) {
        unordered_map<int, int> freq;

        for (int num : nums) {
            freq[num]++;
        }

        int maxFreq = 0;

        for (auto [num, count] : freq) {
            maxFreq = max(maxFreq, count);
        }

        int answer = 0;

        for (auto [num, count] : freq) {
            if (count == maxFreq) {
                answer += count;
            }
        }

        return answer;
    }
};
```

### Notes / Tips

* The question asks for the **total number of occurrences**, not the number of distinct elements.
* If three different elements each occur `3` times, the answer is `9`, not `3`.
* Pattern:

  ```text
  frequency → maximum frequency → sum matching frequencies
  ```
* A frequency map is the most direct approach.

### Key Template

```text
freq = count frequencies

maxFreq = maximum frequency

answer = 0

for each (element, frequency):
    if frequency == maxFreq:
        answer += frequency

return answer
```

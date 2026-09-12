# LeetCode 49 — Group Anagrams

## Metadata

* **LeetCode:** 49
* **Problem:** Group Anagrams
* **Difficulty:** Medium
* **Topics:** Array, Hash Table, String, Sorting
* **Pattern:** Canonical Key Grouping
* **Key Technique:** Map every string to a canonical representation (its sorted form, or a character frequency signature) so all anagrams collapse to the same hash map key
* **Optimal Complexity:** `O(n * k)` Time, `O(n * k)` Space (where `k` is the max string length)

---

## Problem Statement

Given an array of strings `strs`, group the anagrams together. Two strings are anagrams if one can be rearranged into the other using all its characters exactly once.

---

## Approaches

1. **Brute Force — Compare Every Pair of Strings Directly**
2. **Better — Sorted String as Hash Map Key**
3. **Optimal — Character Frequency Signature as Hash Map Key**

---

# Approach 1 — Brute Force / Compare Every Pair of Strings Directly

## Idea

For each string, check it against every group formed so far by comparing it directly (via a full anagram check, e.g. sorting both and comparing, or frequency counting) with one representative from each existing group. If it matches a group, add it there; otherwise, start a new group.

## Dry Run

```text
strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
```

Process `"eat"`: no groups yet → new group `["eat"]`.

Process `"tea"`: compare against `"eat"` (anagram check) → match → add to that group: `["eat", "tea"]`.

Process `"tan"`: compare against `"eat"` → not anagram → new group `["tan"]`.

Process `"ate"`: compare against `"eat"` → match → add: `["eat", "tea", "ate"]`.

Process `"nat"`: compare against `"eat"` → no; compare against `"tan"` → match → add: `["tan", "nat"]`.

Process `"bat"`: compare against `"eat"` → no; compare against `"tan"` → no → new group `["bat"]`.

Final groups: `["eat","tea","ate"], ["tan","nat"], ["bat"]`.

## Algorithm

1. Initialize an empty list of groups.
2. For each string `s` in `strs`:

   * For each existing group, check if `s` is an anagram of that group's first element (via sorting or frequency comparison).
   * If a match is found, add `s` to that group.
   * If no match is found, create a new group containing just `s`.
3. Return all groups.

## Complexity

* **Time:** `O(n² * k log k)`

  * For each of the `n` strings, comparing against up to `n` existing groups, each comparison costing `O(k log k)` to sort and compare strings of length up to `k`.
* **Space:** `O(n * k)`

  * For storing all the grouped strings.

## Notes / Tips

* Repeatedly comparing each new string against a representative of every existing group is redundant — a canonical key (like the sorted string itself) lets a hash map do this grouping in a single pass instead.
* This pairwise comparison approach scales quadratically in the number of strings, which is the main bottleneck removed by the hash map approaches.

## Code

```cpp
class Solution {
public:
    bool isAnagram(string& a, string& b) {
        if (a.size() != b.size()) return false;

        string sortedA = a, sortedB = b;
        sort(sortedA.begin(), sortedA.end());
        sort(sortedB.begin(), sortedB.end());

        return sortedA == sortedB;
    }

    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        vector<vector<string>> groups;

        for (string& s : strs) {
            bool placed = false;

            for (auto& group : groups) {
                if (isAnagram(s, group[0])) {
                    group.push_back(s);
                    placed = true;
                    break;
                }
            }

            if (!placed) {
                groups.push_back({s});
            }
        }

        return groups;
    }
};
```

---

# Approach 2 — Better / Sorted String as Hash Map Key

## Idea

Anagrams always produce the exact same string when sorted. Use each string's sorted version as a hash map key, and append the original string to the list stored under that key. Strings that are anagrams of each other will always land under the same key.

## Dry Run

```text
strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
```

Process:

```text
"eat" → sorted "aet" → map["aet"] = ["eat"]
"tea" → sorted "aet" → map["aet"] = ["eat", "tea"]
"tan" → sorted "ant" → map["ant"] = ["tan"]
"ate" → sorted "aet" → map["aet"] = ["eat", "tea", "ate"]
"nat" → sorted "ant" → map["ant"] = ["tan", "nat"]
"bat" → sorted "abt" → map["abt"] = ["bat"]
```

Final groups (values of the map): `["eat","tea","ate"], ["tan","nat"], ["bat"]`.

## Algorithm

1. Initialize an empty hash map from string to list of strings.
2. For each string `s` in `strs`:

   * Compute `key = sorted(s)`.
   * Append `s` to `map[key]`.
3. Return all values in the map as the grouped result.

## Complexity

* **Time:** `O(n * k log k)`

  * Each of the `n` strings is sorted, costing `O(k log k)` where `k` is the string's length.
* **Space:** `O(n * k)`

  * For the hash map storing all strings, plus the sorted keys.

## Notes / Tips

* This is a huge improvement over Approach 1 — sorting each string once and using it as a key turns pairwise comparisons into a single hash map insertion per string.
* Sorting is the main remaining cost per string — Approach 3 removes even this by using a frequency count instead, which is faster for strings restricted to a small fixed alphabet.

## Code

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> groups;

        for (string& s : strs) {
            string key = s;
            sort(key.begin(), key.end());
            groups[key].push_back(s);
        }

        vector<vector<string>> result;
        for (auto& [key, group] : groups) {
            result.push_back(group);
        }

        return result;
    }
};
```

---

# Approach 3 — Optimal / Character Frequency Signature as Hash Map Key

## Idea

Since the problem is restricted to lowercase English letters, build a fixed-size 26-element frequency count for each string and use that count (encoded as a string or tuple) as the hash map key instead of the sorted string. Anagrams always produce identical frequency counts, and computing a frequency count is faster than sorting.

## Dry Run

```text
strs = ["eat", "tea", "bat"]
```

Process `"eat"`:

```text
count = [1,0,0,0,1,0,...,1,...] (a:1, e:1, t:1, rest 0)
key = "1#0#0#0#1#0#...#1#..." (encoded as counts joined by a delimiter)
map[key] = ["eat"]
```

Process `"tea"`:

```text
count = same as "eat" (a:1, e:1, t:1)
key = same string → map[key] = ["eat", "tea"]
```

Process `"bat"`:

```text
count = a:1, b:1, t:1 → different key → map[newKey] = ["bat"]
```

## Algorithm

1. Initialize an empty hash map from key (encoded frequency count) to list of strings.
2. For each string `s` in `strs`:

   * Build a `count` array of size `26`, all zeros.
   * For each character in `s`, increment `count[c - 'a']`.
   * Encode `count` into a single string key (e.g. joining counts with a delimiter to avoid ambiguity, like `"1#0#2#..."`).
   * Append `s` to `map[key]`.
3. Return all values in the map as the grouped result.

## Complexity

* **Time:** `O(n * k)`

  * Each of the `n` strings requires a single `O(k)` pass to build its frequency count and encode the key, avoiding the `O(k log k)` sort.
* **Space:** `O(n * k)`

  * For the hash map storing all strings, plus the frequency-count keys (each of fixed size `26`, contributing `O(n)` total for the keys themselves, and `O(n*k)` for the grouped strings).

## Notes / Tips

* Using a delimiter (like `'#'`) between counts in the encoded key is important — without it, counts like `1, 23` and `12, 3` could produce the same concatenated string (`"123"`), causing incorrect collisions.
* This approach trades sorting's `O(k log k)` for a `O(k)` frequency count, which matters more as string lengths grow, though for typical LeetCode-scale inputs both approaches run comfortably fast.
* For inputs with a larger or unknown character set (not just lowercase English letters), this fixed-26-slot approach doesn't directly apply — a hash map keyed by character would replace the fixed-size array, similar to the adaptation needed in LC 242 (Valid Anagram) for non-lowercase inputs.

## Code

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> groups;

        for (string& s : strs) {
            vector<int> count(26, 0);
            for (char c : s) {
                count[c - 'a']++;
            }

            string key = "";
            for (int i = 0; i < 26; i++) {
                key += to_string(count[i]) + "#";
            }

            groups[key].push_back(s);
        }

        vector<vector<string>> result;
        for (auto& [key, group] : groups) {
            result.push_back(group);
        }

        return result;
    }
};
```

---

## Key Template

```text
groups = {}

for s in strs:
    count = array of size 26, all 0
    for c in s:
        count[c - 'a'] += 1

    key = encode(count)  # e.g. join with a delimiter
    groups[key].append(s)

return list(groups.values())
```
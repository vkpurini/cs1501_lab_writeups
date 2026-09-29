# Lab 3 Writeup

**Author:** Vivek Purini

---

## Remove Sub-Folders from the Filesystem

**LeetCode Problem:** [1233. Remove Sub-Folders from the Filesystem](https://leetcode.com/problems/remove-sub-folders-from-the-filesystem/)

## Solution

```java
class Solution {
    public List<String> removeSubfolders(String[] folder) {
        Arrays.sort(folder);
        
        List<String> result = new ArrayList<>();
        String lastAdded = "";
        
        for (String f : folder) {
            if (result.isEmpty() || !f.startsWith(lastAdded + "/")) {
                result.add(f);
                lastAdded = f;
            }
        }
        
        return result;
    }
}
```

## Description

The solution first sorts the array of folder paths alphabetically. Sorting guarantees that a folder's parent, if it exists in the list, always appears immediately before it — since a shorter path that is a prefix of a longer one always sorts first (e.g., `"/a"` before `"/a/b"`). This lets us avoid comparing every folder against every other folder; we only ever need to compare a folder against the single most recently kept folder.

We iterate through the sorted list while tracking `lastAdded`, the last folder added to the result. For each folder `f`, we check whether it starts with `lastAdded + "/"`. The appended `"/"` is essential, since it distinguishes true nesting from a coincidental string prefix match — for example, `"/a/b/ca"` starts with the raw string `"/a/b/c"`, but is not actually a sub-folder of it, since the character right after the prefix is `'a'`, not `'/'`.

If `f` is not a sub-folder of `lastAdded` (or if `result` is still empty, meaning this is the first folder processed), we add `f` to the result and update `lastAdded` to `f`, since it becomes the new reference point for subsequent comparisons. Otherwise, `f` is a sub-folder of something already kept, so we skip it.

## Runtime and Memory Analysis

**Time complexity: O(n log n)**

Where `n` is the number of folders. Sorting the array dominates the runtime at O(n log n) comparisons. The subsequent single pass through the sorted list is O(n), with each iteration performing a `startsWith` check bounded by the folder path length. Since the sort dominates, the overall time complexity is O(n log n).

**Space complexity: O(n)**

The `result` list holds up to `n` folder paths in the worst case, when no folder is a sub-folder of another. Sorting may also use auxiliary space depending on the underlying sort implementation (Java's `Arrays.sort` uses a comparison-based sort with O(log n) to O(n) auxiliary space for object arrays), but overall the space used is O(n).

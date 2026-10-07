# Lab 4 Writeup

**Author:** Vivek Purini

---

## Maximum Product of Two Elements in an Array

**LeetCode Problem:** [1464. Maximum Product of Two Elements in an Array](https://leetcode.com/problems/maximum-product-of-two-elements-in-an-array/)

## Solution

```java
class Solution {
    public int maxProduct(int[] nums) {
        int first = 0, second = 0;  // the two largest values seen so far

        for (int i = 0; i < nums.length; i++) {
            if (nums[i] > first) {
                second = first;
                first = nums[i];
            } else if (nums[i] > second) {
                second = nums[i];
            }
        }

        return (first - 1) * (second - 1);
    }
}
```

## Description

Every `nums[i]` is at least 1, so each factor `(nums[i] - 1)` is non-negative. The product of two non-negative numbers is largest when both are as large as possible, so the problem reduces to finding the two largest values in the array and returning `(first - 1) * (second - 1)`.

Instead of sorting, the solution finds them in one pass using two variables, `first` (largest so far) and `second` (second largest), both starting at `0`. Since every value is at least 1, the first real element always replaces the initial `0`. For each element:
- If it is larger than `first`, it becomes the new `first` and the old `first` is demoted to `second`.
- Otherwise, if it is larger than `second`, it replaces `second`.
- Otherwise, it is ignored.

Duplicates work correctly: in `[1,5,4,5]`, the second `5` is not larger than `first` but is larger than `second`, so it becomes `second`, giving `(5-1) * (5-1) = 16`.

## Runtime and Memory Analysis

**Time complexity: O(n)** — a single pass over the array with constant work per element. (Sorting would cost O(n log n).)

**Space complexity: O(1)** — only a few extra variables are used, regardless of input size.

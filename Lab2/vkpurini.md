# Lab 2 Writeup

**Author:** Vivek Purini

---

## Merge Two Binary Trees

**LeetCode Problem:** [617. Merge Two Binary Trees](https://leetcode.com/problems/merge-two-binary-trees/)

## Solution

```java
class Solution {
    public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
        if (root1 == null) return root2;
        if (root2 == null) return root1;

        TreeNode merged = new TreeNode(root1.val + root2.val);

        merged.left = mergeTrees(root1.left, root2.left);
        merged.right = mergeTrees(root1.right, root2.right);

        return merged;
    }
}
```

## Description

The solution uses recursion to traverse both trees simultaneously, node by node.

The base cases handle the situation where one tree has no node at a given position: if `root1` is `null`, we return `root2` as-is (and vice versa), since there is nothing to merge with.

When both trees have a node at the same position, we create a new `TreeNode` whose value is the sum of the two nodes' values. We then recurse down both left subtrees together and both right subtrees together, building the merged tree bottom-up as the call stack unwinds.

## Runtime and Memory Analysis

**Time complexity: O(min(n, m))**

Where `n` and `m` are the number of nodes in `root1` and `root2` respectively. We only visit positions where both trees have a node — once one tree runs out of nodes at a given subtree, we return immediately without recursing further. In the worst case (identical tree shapes), this visits every node in the smaller tree.

**Space complexity: O(min(h1, h2))**

Where `h1` and `h2` are the heights of the two trees. The recursion stack depth is bounded by how deep we can go before one tree bottoms out, which is at most the height of the shallower tree. In the worst case (a completely skewed tree), this is O(min(n, m)).

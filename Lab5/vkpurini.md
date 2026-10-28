# Lab 5 Writeup

**Author:** Vivek Purini

---

## Find the Town Judge

**LeetCode Problem:** [997. Find the Town Judge](https://leetcode.com/problems/find-the-town-judge/)

## Solution

```python
class Solution(object):
    def findJudge(self, n, trust):
        score = [0] * (n + 1)        # people are labeled 1..n
        for a, b in trust:
            score[a] -= 1            # a trusts someone, so a can't be the judge
            score[b] += 1            # b is trusted by one more person
        for i in range(1, n + 1):
            if score[i] == n - 1:
                return i
        return -1
```

## Description

The town judge trusts nobody and is trusted by everyone else. That means their net trust score should be exactly n - 1: they gain +1 for every person who trusts them (n - 1 people) and never lose points for trusting anyone.

The solution uses a single `score` array of size n + 1 (1-indexed to match the labels). For each trust relationship `[a, b]`, `a` gets -1 (they trust someone, so they can't be the judge) and `b` gets +1 (they're trusted by one more person). After processing all relationships, anyone with a score of exactly n - 1 is the judge. If nobody has that score, return -1.

## Runtime and Memory Analysis

**Time complexity: O(n)**

**Space complexity: O(n)**

The `score` array holds one entry per person, so it grows linearly with n.

---
title: 算法之动态规划
date: 2018-01-04 01:35:41
description: 以代码实例理解动态规划
tags:
categories:
- Arithmetic
copyright: false
---

# 基本思想
动态规划算法通常用于求解具有某种最优性质的问题。在这类问题中，可能会有许多可行解。每一个解都对应于一个值，我们希望找到具有最优值的解。动态规划算法与分治法类似，其基本思想也是将待求解问题分解成若干个子问题，先求解子问题，然后从这些子问题的解得到原问题的解。与分治法不同的是，适合于用动态规划求解的问题，经分解得到子问题往往不是互相独立的。若用分治法来解这类问题，则分解得到的子问题数目太多，有些子问题被重复计算了很多次。如果我们能够保存已解决的子问题的答案，而在需要时再找出已求得的答案，这样就可以避免大量的重复计算，节省时间。我们可以用一个表来记录所有已解的子问题的答案。不管该子问题以后是否被用到，只要它被计算过，就将其结果填入表中。这就是动态规划法的基本思路。具体的动态规划算法多种多样，但它们具有相同的填表格式。

# 实例
以 `Leetcode 198` 题为例，

题目，作为一名职业强盗，要抢劫一条街上的户主；每户都有一定数量的金钱，唯一的约束在于如果抢劫连续的两家，则会触发警报；

如果采用自顶向下的解法：

```
# -*- coding: UTF-8 -*-

class Solution(object):
    def rob(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        self.result = []
        for index in range(len(nums)):
            self.result.append(-1)

        return self.solve(len(nums) - 1, nums)

    def solve(self, range, nums):
        """
        :range: 范围
        :nums:  数据列表
        """
        if range < 0:
            return 0

        if self.result[range] >= 0:
            return self.result[range]
        
        self.result[range] = max(nums[range] + self.solve(range - 2, nums), self.solve(range - 1, nums))

        return self.result[range]
```

如果采用自下向上的解法：

```
# -*- coding: UTF-8 -*-

class Solution(object):
    def rob(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        if len(nums) == 0:
            return 0;

        if len(nums) == 1:
            return nums[0];

        self.result = [0] * len(nums)
        self.result[0] = nums[0]
        self.result[1] = max(nums[0], nums[1])

        for index in range(2, len(nums)):
            self.result[index] = max(nums[index] + self.result[index - 2], self.result[index -1])
        return self.result[len(nums) - 1]
```

运行测试

```
data = [
    155,44,52,58,250,225,109,118,211,73,137,96,137,89,174,66,134,26,
    25,205,239,85,146,73,55,6,122,196,128,50,61,230,94,208,46,243,
    105,81,157,89,205,78,249,203,238,239,217,212,241,242,157,79,133,66,36,165
]

s = Solution()
print s.rob(data)
```

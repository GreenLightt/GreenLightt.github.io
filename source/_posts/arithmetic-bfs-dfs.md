---
title: 算法之图搜索
date: 2018-01-11 00:53:41
description: 以代码实例理解深度优先算法和广度优先算法
tags:
categories:
- Arithmetic
copyright: false
---

图是一种灵活的数据结构，一般作为一种模型用来定义对象之间的关系或联系。对象由顶点（`V`）表示，而对象之间的关系或者关联则通过图的边（`E`）来表示。
图可以分为有向图和无向图，一般用 `G=(V,E)` 来表示图。经常用邻接矩阵或者邻接表来描述一副图。
在图的基本算法中，最初需要接触的就是图的遍历算法，根据访问节点的顺序，可分为广度优先搜索（`BFS`）和深度优先搜索（`DFS`）。

# 深度优先算法
深度优先搜索在搜索过程中访问某个顶点后，需要递归地访问此顶点的所有未访问过的相邻顶点。

## N 皇后问题
**实例，以 [`Leetcode 51`](https://leetcode.com/problems/n-queens/description/)题**

题目， 在 `N * N`的棋盘上摆放 `N` 个皇后，使得任意两个皇后都不能处于同一行、同一列或同一斜线上；

```
# -*- coding: UTF-8 -*-

class Solution(object):
    def solveNQueens(self, n):
        """
        算法的入口
        :type n: int
        :rtype: List[List[str]]
        """
        # 存放最后的可能性结果，每一个元素都是一个结果，每一个结果都是一个列表
        self.answer = []
        # 保存每一列皇后所在的行号, 此处进行初始化
        self.path = []
        for index in range(n):
            self.path.append(-1)

        self.dns(0, n)

        return self.answer

    def dns(self, col, n):
        """
        深度优先遍历算法
        : col: 当前列
        ：  n: 总列
        """
        # 超过边界，记录这个结果
        if col >= n:
            # 遍历整个棋盘并输出
            result_record = []
            for i in range(n):
                row_record = []
                for j in range(n):
                    # 如果该行该列是皇后，标记为 Q
                    if self.path[i] == j:
                        row_record.append('Q')
                    else:
                        row_record.append('.')
                result_record.append("".join(row_record))
            self.answer.append(result_record)
            return

        for row in range(n):
            # 对于当前列，每一行都是可能性
            if self.__check(row, col, n):
                self.path[col] = row
                self.dns(col + 1, n)
                # 复原
                self.path[col] = -1


    def __check(self, row, col, n):
        """
        私有方法：检查当前行与列是否有 行冲突及斜线冲突
        : row: 行
        : col: 列
        """
        # 和前 col - 1 列存在的数据作比较
        for search_col in range(col):
            if  self.path[search_col] == row or abs(self.path[search_col] - row) == abs(search_col - col):
                return False
        return True
```

# 广度优先算法

广度优先遍历是连通图的一种遍历策略。因为它的思想是从一个顶点 `V0` 开始，辐射状地优先遍历其周围较广的区域，故得名。

## 岛的数量
**实例，以 [`Leetcode 200`](https://leetcode.com/problems/number-of-islands/description/)题**

题目， 在一个 2 维数组中， 1 表示岛，0 表示水，求被水环绕的岛的数量；


```
11110
11010
11000
00000
```

上面的例子中，被水（0） 环绕的岛（1）的整体只有 1 个，所以结果就是 1 啦；

```
# -*- coding: UTF-8 -*-

class Solution(object):
    def numIslands(self, grid):
        """
        算法的入口
        :type grid: List[List[str]]
        :rtype: int
        """
        # 如果二维数组中有一维不存在，则返回 0
        if len(grid) == 0 or len(grid[0]) == 0:
            return 0

        lands_num = 0
        # 遍历是岛的点，将其周围的岛全部挖掉，则一个整体确定消失,也叫 洪水填充法
        for row in range(len(grid)):
            for col in range(len(grid[row])):
                if grid[row][col] == '1':
                    self.__flood_fill(row, col, grid)
                    lands_num += 1
        
        return lands_num

    def __flood_fill(self, row, col, grid):
        """
        处理 grid 二维数组中处于 row 行 col 列的岛周围连结的岛
        """
        # 队列，存放与当前 row 行 col 列的岛周围连结的岛
        connect_land = [[row, col]]
        
        while len(connect_land) != 0:
            # 对队列的头结点上下左右的岛 处理
            current_land = connect_land[0]
            grid[current_land[0]][current_land[1]] = '0'
            self.__deal(current_land[0] + 1, current_land[1], connect_land, grid)
            self.__deal(current_land[0] - 1, current_land[1], connect_land, grid)
            self.__deal(current_land[0], current_land[1] - 1, connect_land, grid)
            self.__deal(current_land[0], current_land[1] + 1, connect_land, grid)
            # 移出队列头结点
            del connect_land[0]
     
    def __deal(self, row, col, connect_land, grid):
        """
        对于 grid 二维数组中处于 row 行 col 列的岛进行判断，如果也是岛，则加入到待处理的队列中
        """
        total_rows = len(grid)
        total_cols = len(grid[0])

        if row >= 0 and row < total_rows and col >= 0 and col < total_cols and grid[row][col] == '1':
            grid[row][col] = '0'
            connect_land.append([row, col])
```




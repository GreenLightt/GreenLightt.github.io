---
title: 算法之树搜索
date: 2018-03-05 21:24:41
description: 以代码实例理解树算法
tags:
categories:
- Arithmetic
copyright: false
---

# 树的介绍

树是一种数据结构，可以用来表示层次关系，因表示的样子很像一颗倒立的树而得名。

![此处输入图片的描述][1]

实例, [`Leetcode 100`](https://leetcode.com/problems/same-tree/description/) 比较两棵树是否是一样的

```
class Solution(object):
    def isSameTree(self, p, q):
        """
        :type p: TreeNode
        :type q: TreeNode
        :rtype: bool
        """
        # 如果都是空结点，返回 true
        if (not p) and (not q):
            return True
        # 判断根结点是否相同
        if  p and q and p.val == q.val:
            # 判断左树和右树是否相同
            return self.isSameTree(p.left, q.left) and self.isSameTree(p.right, q.right)

        return False
```

实例, [`Leetcode 101`](https://leetcode.com/problems/symmetric-tree/description/) 比较一棵树是不是镜像对称的

```
class Solution(object):
    def isSymmetric(self, root):
        """
        :type root: TreeNode
        :rtype: bool
        """
        # 如果树为空
        if not root:
            return True
        # 判断左子树与右子树是否是镜像
        return self.isMirror(root.left, root.right)

    def isMirror(self, tree_a, tree_b):
        # 如果两棵树都为空
        if (not tree_a) and (not tree_b):
            return True
        # 判断两棵树都不为空，且根结点一致
        if tree_a and tree_b and tree_a.val == tree_b.val:
            # 判断A树的左子树和B的右子树，A树的右子树和B的左子树
            return self.isMirror(tree_a.left, tree_b.right) and self.isMirror(tree_a.right, tree_b.left)
        return False
```

# 二叉树

在计算机科学中，二叉树是每个节点最多有两个子树的树结构。


遍历有三种方式：

- 前序：根左右。[Leetcode  144](https://leetcode.com/problems/binary-tree-preorder-traversal/description/)
- 中序：左根右。[Leetcode  94](https://leetcode.com/problems/binary-tree-inorder-traversal/description/)
- 后序：左右根。[Leetcode 145](https://leetcode.com/problems/binary-tree-postorder-traversal/description/)

根据遍历的方式延伸出构造题：

- 根据前序和中序构造二叉树。 [Leetcode 105](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/description/)
- 根据中序和后序构造二叉树。[Leetcode 106](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/description/)

## 非递归遍历
如果不采用递归，分别进行三种遍历的思路如下：

**前序遍历**
使用栈辅助，右子树先进栈，左子树再进栈；pop; 右子树先进栈，左子树再进栈 ...

**中序遍历**
1. 有左子树，则根节点进栈； R = 左子树
2. 栈顶被弹出，访问当前节点， R = 右子树

**后序遍历**
1. 有左子树，则根节点进栈；R = 左子树
2. 第一次出现，R = 右子树
3. 第二次出现，根节点访问自己

# 二叉查找树

二叉排序树或者是一棵空树，或者是具有下列性质的二叉树：

（1）若左子树不空，则左子树上所有结点的值均小于或等于它的根结点的值；
（2）若右子树不空，则右子树上所有结点的值均大于或等于它的根结点的值；
（3）左、右子树也分别为二叉排序树；

在树中查询是否存在指定值的结点的代价是 `O(h)` （`h` 是树的深度）

**实战题：**

- 判断是否是二叉查找树 [Leetcode 98](https://leetcode.com/problems/validate-binary-search-tree/description/)
- 在二叉查找树中找到第 k 个小的元素 [Leetcode 230](https://leetcode.com/problems/kth-smallest-element-in-a-bst/description/)
- 有序数组转换成平衡二叉树 [Leetcode 108 ](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/description/)

# 字典树

又称单词查找树，`Trie` 树，是一种树形结构，是一种哈希树的变种。典型应用是用于统计，排序和保存大量的字符串（但不仅限于字符串），所以经常被搜索引擎系统用于文本词频统计。它的优点是：利用字符串的公共前缀来减少查询时间，最大限度地减少无谓的字符串比较，查询效率比哈希树高。

**性质**

它有 3 个基本性质：

- 根节点不包含字符，除根节点外每一个节点都只包含一个字符；
- 从根节点到某一节点，路径上经过的字符连接起来，为该节点对应的字符串；
- 每个节点的所有子节点包含的字符都不相同。

**实战**

1. 构造一棵字典树（前缀树） [Leetcode 208](https://leetcode.com/problems/implement-trie-prefix-tree/description/)


# 总结

- 树的问题是典型的递归问题，可以转换成解决子树的问题
- 注意空树，单结点树，子节点为 `Null` 的边界情况 
- 用递归理清思路
- 数据量较大时，使用非递归写法

  [1]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20180112004428.png
  



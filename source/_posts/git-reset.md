---
title: Git 回退提交的错误
date: 2017-09-27 21:42:41
description: 总结Git 回退时的命令
tags:
- Git
categories:
- Linux
copyright: false
---

# 本地分支
在本地分支上进行回退操作，回退的内容分三部分，`commit`信息、`index file`（暂存区域）和源代码；

1. 回退`commit`, `index file`, 本地源码

 ```
    # 回滚到哪次commit，根据commitid
    git reset --hard <commitid>
    # 回滚到最近的几次（n）
    git reset --hard HEAD~<n>
    # 回滚到和远程dev分支一样
    git reset --hard origin/dev
 ```
2. 回退`commit`, `index file`， 保留本地源码；此为`git`默认方式

 ```
   git reset --mixed <commitid>
 ```
3. 回退`commit`信息，保留`index file` 和 本地源码

 ```
   git reset --soft <commitid>
 ```

# 远程分支
1. 删除远程分支上的最后一次提交

 ```
   git revert HEAD
   git push origin master
 ```

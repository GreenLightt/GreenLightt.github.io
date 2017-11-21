---
title: Git 日常操作
date: 2017-09-27 17:57:41
description: 总结经常用的 Git 操作
tags:
- Git
categories:
- Version Management
copyright: false
---

# 分支 
## 本地分支
1. 查看本地分支

 ```
  git branch
 ```
2. 删除本地分支

 ```
git branch -d ~分支名
 ```
3. 重命名本地分支

 ```
 git branch -m <old_branch_name> <new_branch_name> 
 ```
4. 查看分支最新的修改

 ```
   git branch -v
 ```
5. 合并分支
将B分支合并到`A`分支上

 ```
 git checkout <想合并入的分支A>  
 git merge <合并分支B>
 ```
6. 创建指定远程跟踪分支(`origin`/`serverfix`)的本地跟踪分支(`serverfix`)

 ```
  git checkout --track origin/serverfix
 ```
7. 本地分支 关联 远程分支
本地`dev`分支关联到远程`dev`分支

 ```
 git branch --set-upstream dev origin/dev
 ```


## 远程分支
1. 查看远程分支

 ```shell
  git branch -r
 ```
2. 删除远程分支

 ```shell
 git push origin :<branch_name>
或 
 git push origin --delete <branch_name>
 ```
3. 显式获取远程引用的完整列表

 ```
    git ls-remote (remote)
 ```
4. 更新本地远程分支

 ```
    git pull origin 
 ```
5. 获取远程分支的最新信息

 ```
  git remote show origin
 ```
6. 删除本地远程跟踪分支

 ```
    git remote prune origin
 ```
7. 获取本地没有的远程分支

 ```
   git fetch
 ```
## 本地分支和远程分支
1.  查看远程分支和本地分支 

 ```
  git branch -a
 ```
2. 查看本地分支对应的远程分支

 ```
  git branch -vv
 ```

## 提交分支
1. 提交本地分支A到远程分支B

 ```
    git push origin A:B
 ```

# 文件
## 跟踪与取消跟踪
取消跟踪
```
git update-index --assume-unchanged your_file_path
```

继续跟踪
```
git update-index --no-assume-unchanged your_file_path
```

# 子模块
## 添加子模块
项目当前未引入子模块，引入子模块：

```
git submodule add [url] [path]

# 例如：
# git submodule add git@git.aimoge.com:ncshop/ncshop-admin-foreign.git foreign
```

## 删除子模块
项目当前存在一个要删除的子模块，目录是`foreign`，有 4 个步骤：
1. 移出`git`管理：`git rm --cached foreign`
2. 手动删除子模块所在目录：`rm -rf foreign`
3. 编辑 `.gitmodules` 文件，将子模块的相关配置节点删除掉
4. 编辑 `.git/config` 文件，将子模块的相关配置节点删除掉

## 初始化子模块
当使用 `git clone` 下来的工程中带有 `submodule` 时，初始的时候，`submodule` 的内容并不会自动下载下来的，此时，要执行以下命令

```
git submodule update --init --recursive
```

## 更新子模块

```
git submodule update
```


# 标签
## 列出所有标签
```
git tag
```
## 推送标签
```
git push origin 标签名
```
## 删除本地标签
```
git tag -d 标签名
```
## 删除远程标签
```
git push origin :refs/tags/标签名  
```
## 检索标签
取指定`tag`标签代码，到新创建的分支上
```
git checkout -b branch_name tag_name
```
如果存在`tag`名和`branch`名相同，则
```
git checkout -b branch_name tags/tag_name
```


# 配置
## 修改配置信息
1. 修改git commit时的编辑器
 ```
  git config core.editor vim
 ```

2. 修改不带任何参数的push为simple
 ```
  git config  push.default simple
 ```

3. 颜色
 ```
 git config color.ui auto
 ```

## 中文文件名乱码
`git` 默认中文文件名是 `\xxx\xxx` 等八进制形式，是因为对 `0x80` 以上的字符进行 `quote`，执行以下命令：

```
git config --global core.quotepath false
```

# 历史
## 提交历史
```
# git log 会按提交时间列出所有的更新， -p 选项展开显示每次提交的内容差异，用 -2 则仅显示最近的两次更新
git log -p -2       
# 仅显示简要的增改行数统计
git log --stat
```

## 命令历史
1. 查看命令历史

 ```
  git reflog 
 ```

# 提交
## 强制提交覆盖远程
```
  git push -f origin 远程分支名
```
## 修改上一次提交的author及邮箱
```
  git commit --amend --author='Redgo Hong <redgo_hong@qq.com>'
```
## 修改最近一次提交的commit注释
```
  git commit --amend
```
## 提交时，指定时间
```
git commit --date="时间"
```
这里的时间格式与`date  -R`相同；假如是第二天的时间,可以先通过`date -d "$(date -R) 1 day" +'%a, %d %b %Y %T  %z'`  得出具体时间; 

## 撤销提交
撤销某次提交，不影响之前或之后的
```
git revert 提交ID
```

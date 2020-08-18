---
description: 如何使用git打patch
---

# git patch

## 1、概览

生成patch的方法有两种`git format-patch` 和 `git diff`，它们生成的文件对应着不同的打patch命令：

`git format-patch`：`git am`

`git diff`：`git am`、`git apply`

git diff生成的是UNIX标准补丁.diff文件，

git format-patch生成的Git专用.patch 文件。 

.diff文件只是记录文件改变的内容，不带有commit记录信息,多个commit可以合并成一个diff文件。

.patch文件带有记录文件改变的内容，也带有commit记录信息,每个commit对应一个patch文件


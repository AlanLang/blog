---
title: Git进阶操作
tags: git
categories: 教程
abbrlink: 27737
date: 2020-07-15 13:53:23
---

## 回退本地修改
```
git reset --hard <需要回退到的版本号（只需输入前几位）>
```
`hard`和`soft`还有`mixed` 的区别可以看[这里](https://stackoverflow.com/questions/3528245/whats-the-difference-between-git-reset-mixed-soft-and-hard)

## 强制将本地版本覆盖要远端
```
git push origin <分支名> --force
```

<!-- more -->

## 撤销远程版本的某一次提交
比如
```
A1–A2–B1
```
如果想撤销A2的提交
```
git revert HEAD                     //撤销最近一次提交
git revert HEAD~1                   //撤销上上次的提交，注意：数字从0开始
git revert 0ffaacc                  //撤销0ffaacc这次提交
```
git revert 命令意思是撤销某次提交。它会产生一个新的提交，虽然代码回退了，但是版本依然是向前的，所以，当你用revert回退之后，所有人pull之后，他们的代码也自动的回退了。
但是，要注意以下几点：

## 从一个分支合并特定的commits到另一个分支
### 单个commit
比如
```
dd2e86 - 946992 -9143a9 - a6fd86 - 5a6057 [master]
           \
            76cada - 62ecb3 - b886a0 [feature]
```
比如，feature 分支上的commit 62ecb3 非常重要，它含有一个bug的修改，或其他人想访问的内容。无论什么原因，你现在只需要将62ecb3 合并到master，而不合并feature上的其他commits，所以我们用git cherry-pick命令来做：
```
git checkout master
git cherry-pick 62ecb3
```
这样就好啦。现在62ecb3 就被合并到master分支，并在master中添加了commit（作为一个新的commit）。
cherry-pick 和merge比较类似，如果git不能合并代码改动（比如遇到合并冲突），git需要你自己来解决冲突并手动添加commit。

### 多个commit
还以上例为例，假设你需要合并feature分支的commit 76cada ~62ecb3 到master分支。
首先需要基于feature创建一个新的分支，并指明新分支的最后一个commit：
```
git checkout -b newbranch 62ecb3
```
然后，rebase这个新分支的commit到master（--ontomaster）。76cada^ 指明你想从哪个特定的commit开始。
```
git rebase --onto master 76cada^
```
得到的结果就是feature分支的commit 76cada ~62ecb3 都被合并到了master分支。
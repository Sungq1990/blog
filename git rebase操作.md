---
title: git rebase操作
date: 2020-05-26 19:06:54
tags: git
---

之前一直都是用git merge命令来合并其它分支的最新改动到本分支的，这样操作的话项目的提交树就会有很多无用的信息，同时用CI工具发布代码的时候迁出就会很慢，回滚则会更慢。
<!--more-->

merge主要发生在这样几个地方

1 两个人同时开发一个分支，在拉取对方代码的时候

git pull origin b1

这个时候就会在本分支产生一个merge，如果你用git pull origin b1 --rebase就可以避免

2 要将代码合并到master的时候

git checkout master
git pull origin master (错）
应该用 git pull origin master --rebase
然后大家往往会选择 git merge master到本分支，这里如果用
git rebase -i master
就可以避免merge
那么这里如果commit较多，产生冲突怎么办？
先放弃rebase
git rebase --abort
然后压缩本分支
git rebase -i HEAD~10
表示压缩本次提交在内的10次提交
将10次提交中无效的提交（修改bug、typo等等）合并到一起
然后再去git rebase -i master

git pull origin xxxx --rebase
git rebase -i master
git rebase -i HEAD~10

三个rebase命令减少merge和commit
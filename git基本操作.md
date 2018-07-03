---
title: git基本操作
date: 2018-06-25 10:02:00
tags: git
---

git常用命令
<!--more-->
#### 初始化仓库
```txt
git init
git add .
git commit -m “init”
git remote add origin 你的github仓库地址
git push
```
```txt
git clone url 从远端拉取仓库
git checkout -b dev 新建本地dev分支
git pull origin dev 把远端dev拉取到本地dev
```
#### 提交文件步骤
```txt
git status -- 查看文件状态
git diff filename -- 查看文件变化
git add filename -- 加入新文件至git管理
git checkout filename -- 撤销文件更改
git commit -m "info" -- commit
git push
```
#### git文件大小写敏感
```txt
git config core.ignorecase false 
```
#### 从mater拉自己的开发分支
~~~txt
git branch dev 新建本地分支
git checkout dev 切到新分支
git pull origin master 同步远端master代码到新分支
git push --set-upstream origin dev 远端新建dev分支，并将本地分支相关联
git checkout dev 检出dev分支
git merge dev 合并dev分支
git push origin dev 推到远端dev分支
~~~
#### 其他
> git branch -a 查看本地和远程所有分支
> git branch -r 查看远端分支
> git checkout -b 本地分支名 origin/远程分支名 从远程仓库里拉取一条本地不存在的分支
> git push --set-upstream origin name 新建远端分支
> git报错:
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
处理:
git branch --set-upstream-to=origin/远端仓库名 本地仓库名
git branch -vv 查看本地和远端仓库关联情况


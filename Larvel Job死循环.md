---
title: Larvel Job死循环
date: 2019-11-25 20:11:54
tags: php Larvel Job
---

公司的一个项目队列用的是laravel自带的job，框架的版本为5.7，队列用的是数据库驱动，失败的队列会进入musql的failed_jobs表，正在执行的则会进入jobs表。今天越到的一个问题就是由于系统设计问题，经常会出现队列执行失败的情况，需要重新执行队列。这一次偷懒没有用命令行重试而是直接选择了数据库操作，出现一个问题就是sql插入的时候将\\变成了\，这样就会导致框架找不到这个job，报

```
Class does not exist
```

这里还有个问题就是job的重试次数是写在代码里的，最终会映射到数据库里的maxTries字段，这里需要说明的是如果是在命令行


```
php artisan queue:work --sleep=3 --tries=3
```

指定重试次数的话应该不会有这个问题。现在出现了一个严重的死循环问题，laravel的attempts失效了直接到最大值255，然后数据库里数据不断的在被删除和插入，由于增删操作太快通过Navicat已经手工删除不了这条记录，解决方法为用Navicat输入命令行锁表

```
lock tables jobs write;

删除数据，然后

unlock tables;
```

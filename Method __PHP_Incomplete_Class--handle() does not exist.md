---
title: laravel job报错
date: 2022-03-17 18:10:00
tags: laravel php job
---

```
执行job任务时报Method __PHP_Incomplete_Class::handle() does not exist，
原因就是一台机器同时跑多个laravel项目，并且有多个项目有job任务，就会冲突
解决方法就是修改queue.php下面的配置
如果是走数据库的就改database.queue 改个名字，同理redis就改redis下面的queue
比如把queue=>default 改成 queue=>default_1
然后修改php artisan queue:work  --queue=default_1 即可
```
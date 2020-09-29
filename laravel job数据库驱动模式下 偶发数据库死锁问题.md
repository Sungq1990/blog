---
title: laravel job数据库驱动模式下 偶发数据库死锁问题
date: 2020-09-29 16:23:54
tags: laravel
---

两个用到queue的项目线上突然经常偶发性的发送数据库死锁的报警，先说一下背景，
laravel版本是5.6，queue用的是数据库驱动，报错信息如下

```
SQLSTATE[40001]: Serialization failure: 1213 Deadlock found when trying to get lock; try restarting transaction (SQL: delete from `jobs` where `id` = 110894)

Trace:#0 /var/www/bifrost/vendor/laravel/framework/src/Illuminate/Database/Connection.php(624): Illuminate\Database\Connection->runQueryCallback('delete from `jo...', Array, Object(Closure))
#1 /var/www/bifrost/vendor/laravel/framework/src/Illuminate/Database/Connection.php(490): Illuminate\Database\Connection->run('delete from `jo...', Array, Object(Closure))
#2 /var/www/bifrost/vendor/laravel/framework/src/Illuminate/Database/Connection.php(435): Illuminate\Database\Connection->affectingStatement('delete from `jo...', Array)
#3 /var/www/bifrost/vendor/laravel/framework/src/Illuminate/Database/Query/Builder.php(2587): Illuminate\Database\Connection->delete('delete from `jo...', Array)
#4 /var/www/bifrost/vendor/laravel/framework/src/Illuminate/Queue/DatabaseQueue.php(294): Illuminate\Database\Query\Builder->delete()
#5 /var/www/bifrost/vendor/laravel/framework/src/Illuminate/Database/Concerns/ManagesTransactions.php(29): Illuminate\Queue\DatabaseQueue->Illuminate\Queue\{closure}(Object(Illuminate\Database\MySqlConnection))
#6 /var/www/bifrost/vendor/laravel/framework/src/Illuminate/Queue/DatabaseQueue.php(296): Illuminate\Database\Connection->transaction(Object(Closure))
#7 /var/www/bifrost/vendor/laravel/framework/src/Illuminate/Queue/Jobs/DatabaseJob.php(68): Illuminate\Queue\DatabaseQueue->deleteReserved('default', 110894)
```

初步怀疑是多线程同时消费造成的(事实上也是)，这里有一点需要注意，laravel用数据库驱动的queue最好只开一个worker，
不然有概率出现死锁的问题，如果驱动替换成redis就不会有相关问题，用redis很流畅，建议优先用redis。
我这个问题是因为公司的k8s配置有问题，有几个pod就有几个worker，刚好释放pod的脚本出问题了，就出现了多个worker。
说这么多重点就是一个，如果queue用的database驱动，最好就开一个worker，这个是文档上没有重点说明的。
相关参考资料
https://github.com/laravel/framework/issues/7046
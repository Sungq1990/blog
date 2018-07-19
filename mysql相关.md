---
title: mysql相关
date: 2018-07-19 15:02:00
tags: mysql
---

记录一些mysql的知识点
<!--more-->

#### 强制走索引
~~~txt
1:force index()
~~~
#### 查看mysql状态
~~~txt
1:show processlist where host like "%%"
~~~
#### 查看sql效率
~~~txt
explain sql
~~~
#### select加锁(for update手工加锁语句)
~~~txt
乐观锁和悲观锁策略
悲观锁：在读取数据时锁住那几行，其他对这几行的更新需要等到悲观锁结束时才能继续 。
乐观锁：读取数据时不锁，更新时检查是否数据已经被更新过，如果是则取消当前更新，一般在悲观锁的等待时间过长而不能接受时我们才会选择乐观锁
~~~
###### 例1: (明确指定主键，并且有此数据，row lock)
~~~txt
SELECT * FROM products WHERE id='3' FOR UPDATE;
~~~
###### 例2: (明确指定主键，若查无此数据，无lock)
~~~txt
SELECT * FROM products WHERE id='-1' FOR UPDATE;
~~~
###### 例3: (无主键，table lock)
~~~txt
SELECT * FROM products WHERE name='Mouse' FOR UPDATE;
~~~
###### 例4: (主键不明确，table lock)
~~~txt
SELECT * FROM products WHERE id<>'3' FOR UPDATE;
~~~
###### 例5: (主键不明确，table lock)
~~~txt
SELECT * FROM products WHERE id LIKE '3' FOR UPDATE;
~~~
#### 存储emoji
~~~txt
mysql server版本不低于5.5.3  character使用utf8mb4
(原来mysql支持的 utf8 编码最大字符长度为 3 字节，如果遇到 4 字节的宽字符就会插入异常了。三个字节的 UTF-8 最大能编码的 Unicode 字符是 0xffff，也就是 Unicode 中的基本多文种平面(BMP)。也就是说，任何不在基本多文本平面的 Unicode字符，都无法使用 Mysql 的 utf8 字符集存储。包括 Emoji 表情)
~~~
#### mysql分区
~~~txt
查看是否支持分区:SHOW VARIABLES LIKE '%partition%';
~~~
~~~txt
1:Range分区   俗称：范围分区。根据表的字段的值，依据给定某段连续的区间来分区。
~~~
```
ALTER TABLE teacher 
partition by range(year(birthdate))
(
partition p1 values less than (1970),
partition p2 values less than (1990),
partition p3 values less than maxvalue
);
``` 
~~~txt
2:LIST分区    俗名：列表分区。其实list分区和range分区应该说都是一样的，不同的是range分区在分区是的依据是一段连续的区间；而list分区针对的分区依据是一组分布的散列值
~~~
```
create table student
 (id varchar(20) not null ,
 studentno int(20) not null,
 name varchar(20),
 age varchar(20)
 )
 partition by list(studentno)
 (
 partition p1 values in (1,2,3,4),
 partition p2 values in  (5,6,7,8),
 partition p3 values in (9,10,11)
 );
```
~~~txt
3:HASH分区    小名：哈希分区。哈希分区主要是依据表的某个字段以及指定分区的数量
~~~
```
create table user (
  id int(20) not null,
  role varchar(20) not null,
  description varchar(50) 
)
partition by hash(id) 
partitions 10;
```
~~~txt
4:key分区     类似于按HASH分区，区别在于KEY分区只支持计算一列或多列，且MySQL服务器提供其自身的哈希函数。必须有一列或多列包含整数值
~~~
```
create table role( id int(20) not null,name varchar(20) not null)
partition by linear key(id)
partitions 10;
```

#### MySQL事务隔离级别
~~~txt
SQL标准定义了4类隔离级别，包括了一些具体规则，用来限定事务内外的哪些改变是可见的，哪些是不可见的。低级别的隔离级一般支持更高的并发处理，并拥有更低的系统开销。
InnoDB默认是可重复读的（REPEATABLE READ
~~~

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| --- | --- | --- | --- |
| 未提交读 | 可能 | 可能 | 可能 |
| 已提交读 | 不可能 | 可能 | 可能 |
| 可重复读 | 不可能 | 不可能 | 可能 |
| 可串行化 | 不可能 | 不可能 | 不可能 |

>Read Uncommit：未提交读。允许脏读。
>Read Commit ： 提交读。只能读到其他事务已提交的内容。允许不重复读。
>Repeated Read ： 可重复读。同一个事务内的查询与事务开始时是一致的。允许幻读。Mysql默认的级别。
>Serializable ： 串行化的读。每次读都要获取表级锁，读写互相阻塞


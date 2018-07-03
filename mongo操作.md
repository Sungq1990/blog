---
title: mongo操作
date: 2018-06-27 17:22:00
tags: mongo
---

mongo查询操作命令
<!--more-->

~~~txt
1 基本查询：
    find({},{"title":1,"status":1,"_id":0})
    find后面加字段查询指定字段，1指定返回，0指定不返回
~~~

~~~txt
2 查询条件：
    $in  -- find({"name":{"$in":["stephen",123]}})
    $nin -- find({"name":{"$nin":["stephen2","stephen1"]}})
    $or  -- find({"$or": [{"name":"stephen1"}, {"age":35}]})   
~~~

| $lt | $lte | $gt | $gte | $ne | $in | $nin | $or | $not |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| < | <= | > | >= | != | in | not in | or | not  |

~~~txt
3.  null数据类型的查询：
    --在进行值为null数据的查询时，所有值为null，以及不包含指定键的文档均会被检索出来。
    
    --需要将null作为数组中的一个元素进行相等性判断，即便这个数组中只有一个元素。
    --再有就是通过$exists判断指定键是否存在
~~~

~~~txt
4.  正则查询(模糊查询):
    --i表示忽略大小写
    > db.test.find({"name":/stephen?/i})
~~~
~~~txt
5.  distinct
    --distinct:     distinct("字段",{条件})    
~~~


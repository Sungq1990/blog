---
title: Mysql GROUP_CONCAT方法
date: 2020-10-30 15:31:54
tags: mysql
---

公司有个业务是分表的，并且每个子表的字段还有略微的差异，字段也很多50+，具体的业务就不说了。
由于各子表的字段名不同，所以需要把表的所有字段都捞出来，这里就需要用到GROUP_CONCAT方法。

```
SET SESSION group_concat_max_len = 4294967295;
SELECT
    GROUP_CONCAT( DISTINCT COLUMN_NAME SEPARATOR ',' )
FROM
    information_schema.COLUMNS
WHERE
    table_name = 'xxxx'
GROUP BY
    table_name;
```

两个注意点，一个是group_concat_max_len，如果用默认值会出现字段获取不全的情况，第二个就是DISTINCT COLUMN_NAME，否则会出现字段重复的情况。
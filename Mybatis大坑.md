---
title: Mybatis大坑
date: 2019-05-16 10:20:14
tags: Mybatis
---

#### 需求
```
insert后返回数据库自增id
```

#### 正确的写法
dao层(重点就是不能加@Param注释)
```
Integer addUser(MpUsers user);
```
xml
```
    <insert id="addUser" useGeneratedKeys="true" keyProperty="id"  parameterType="com.sgq.user.entity.MpUsers">
        INSERT INTO mp_users (openid,platform,created_at,updated_at) values (#{openid},#{platform},#{created_at},#{updated_at})
    </insert>
```
业务层获取
```
MpUsers user = new MpUsers();
Integer id = user.getId();
```

#### 坑总结
```
1、dao层不能传参数，要传Bean
2、dao层不能使用@Param注释
```
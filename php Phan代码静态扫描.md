---
title: php Phan代码静态扫描
date: 2019-03-25 14:01:14
tags: Phan
---
Phan是一个PHP的静态分析器,它会查找常见问题，并在类型信息可用或可以推断时验证各种操作的类型兼容性
<!--more-->

#### 安装
先安装php-ast扩展
```
pecl install ast
```
composer全局安装
```
composer global require phan/phan
```

#### 使用
配置文件(phan.php)
```
<?php
return [
  // php版本
  'target_php_version' => 7.1,
  // 扫描文件夹路径
  'directory_list' => [
    'path'
  ],
  // 忽略的文件夹
  'exclude_analysis_directory_list' => [
    'path'
  ]
];
```
扫描代码
```
phan -k  phan.php -i -y 10 -o /path/output
```
参数简要说明
```
-k:配置文件路径
-o:输出结果的文件
-i:忽略未定义的方法和类
-y 10:只输出严重的错误 
```
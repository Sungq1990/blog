---
title: satis搭建一个私有composer包仓库
date: 2018-10-09 19:20:14
tags: satis composer
---

现在服务端基本都是微服务了，多个工程之间肯定有许多公共的基础组件，比如日志、队列、邮件等等，这些都是每个工程都可能要用到的，可以封装成公司的私有组件包，这样各个工程需要用到的时候只要引入这个包就可以了，这就需要搭建一个包仓库了，这里我们选用了比较简单的satis。
<!--more-->
#### 安装
***
```
cd /data/www/
composer create-project composer/satis --stability=dev --keep-vcs
mv satis packages.dev.com
cd packages.dev.com
```
#### 配置
***
satis的配置是通过satis.json进行的，我们在当前目录新建一个satis.json。
```
{
    "name": "My Composer",
    "homepage": "http://packages.dev.com",
    "repositories": [
        {"type": "vcs", "url": "https://gitee.com/xxx/package1.git"}
    ],
    "require": {
      "xxx/package1": "*"
    }
}
```

```
简单解释下这个json
name：仓库的名字，可以随便定义
homepage：仓库建立之后的的主页地址
repositories：指定去哪获取包，url中需要带.git
require：指定获取哪些包，如果想获取所有包，使用require-all: true,
```
#### 创建一个测试组件包
***
去git上新建一个工程
在项目根目录新建composer.json
内容如下
```
{
  "name": "xxx/package1",
  "description": "test",
  "keywords": [
    "test"
  ],
  "type": "library",
  "license": "MIT",
  "authors": [
    {
      "name": "sgq",
      "email": "xxx@mail.com"
    }
  ],
  "require": {
    "php": ">=5.6.4"
  },
  "autoload": {
    "classmap": [
    ],
    "psr-4": {
      "xxx\\package1\\": "src/"
    }
  }
}
```
#### 生成
***
```
php bin/satis build satis.json public/
```
#### 配置nginx
***
```
server {
    listen  80;
    server_name packages.dev.com;
    root /data/home/packages.dev.com/public;
}
```

重新加载nginx配置nginx -s reload
我们就可以通过packages.dev.com进行访问了。

#### 使用
修改相关工程的composer.json文件
```
{
    "repositories": [
      { "type": "composer", "url": "http://packages.dev.com/" }
    ],
    "require": {
        "xxx/package1": "dev-master"
    }
}
```
执行composer update即可
#### 下载
上面satis.json的配置会去我们的git中clone，有时候会比较慢，我们并不希望每次都clone，其实我们也可以缓存在我们的仓库中，这样每次update的时候就只用下载了。
在satis.json中增加
```
{
    "archive": {
        "directory": "dist",
        "format": "tar",
        "prefix-url": "http://packages.dev.com/",
        "skip-dev": true
    }
}
```
参数说明
```
directory: 必需要的，表示生成的压缩包存放的目录，会在我们build时的目录中
format: 压缩包格式, zip（默认） tar
prefix-url: 下载链接的前缀的Url,默认会从homepage中取
skip-dev: 默认为假，是否跳过开发分支
absolute-directory: 绝对目录
whitelist: 白名单，只下载哪些
blacklist: 黑名单，不下载哪些
checksum: 可选，是否验证sha1
```
再次生成
```
php bin/satis build satis.json public/
```
会发现public目录多了一个dist目录，里面有很多tar的压缩包，这就是我们的package。
之后再执行composer update就会发现快了很多
#### 其他错误
***
composer出现Your configuration does not allow connections to 
```
出现这样的问题是，镜像使用的是http，而原地址是需要https，所以配置下关掉https就好了
composer config -g secure-http false
```
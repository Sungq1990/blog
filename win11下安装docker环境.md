---
title: win11下安装docker环境
date: 2023-11-27 17:35:23
tags: win11 docker
---

docker exec -it ubuntu /bin/bash

docker run -d -p 3306:3306 --name mysql5.7 --privileged=true -v D:\developer\docker-data\mysql-5.7\conf.d:/etc/mysql/conf.d -v D:\developer\docker-data\mysql-5.7\logs:/var/log/mysql -v D:\developer\docker-data\mysql-5.7\data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root --restart=always -d mysql:5.7

docker run -it -p 9002:80 --name ubuntu -v D:\developer\docker-data\data:/data -v D:\developer\docker-data\data\hosts:/etc/hosts --restart=always -d ubuntu

docker run -d -p 6379:6379 --name redis -v D:\developer\docker-data\redis\redis.conf:/etc/redis/redis.conf -v D:\developer\docker-data\redis\data:/data --restart=always -d redis

docker run -d -p 11211:11211 --name memcached -v D:\developer\docker-data\memcached\data:/data --restart=always -d memcached

docker run -d -p 27017:27017 --name mongo -v D:\developer\docker-data\mongo\data:/data --restart=always -d mongo

docker-php-ext-install opcache pdo pdo_mysql mysqli

pecl install -o -f redis && rm -rf /tmp/pear && docker-php-ext-enable redis

pecl install -o -f mongodb && rm -rf /tmp/pear && docker-php-ext-enable mongodb

pecl install -o -f memcached && rm -rf /tmp/pear && docker-php-ext-enable memcached

wget -q https://packages.sury.org/php/apt.gpg -- I  apt-key add echo "deb https://packages.sury.org/php/stretchmain" |  tee /etc/apt/sources.list.d/php.list

apt-get install software-properties-common
add-apt-repository ppa:ondrej/php
apt-get update
apt-get install php7.3 php7.3-fpm  php7.3-mysql php7.3-cli php7.3-common php7.3-xml php7.3-curl php7.3-igbinary php7.3-opcache php7.3-phpdbg php7.3-readline php7.3-redis php7.3-memcache php7.3-mongodb php7.3-gd

php7.2 -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php7.2 composer-setup.php --install-dir=/usr/local/bin --filename=composer

composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/

service nginx start
service php7.2-fpm start

开机启动
update-rc.d nginx defaults
update-rc.d php7.2-fpm defaults

php-fpm www.conf监听文件改成监听端口

dockre network相关
docker network create 网络名称
指定网络 docker network connect 网络名称 容器名称
解除绑定 docker network disconnect 网络名称 容器名称

win下进入docker root dir的方法

docker run --privileged -it -v /var/run/docker.sock:/var/run/docker.sock jongallant/ubuntu-docker-client

docker run --net=host --ipc=host --uts=host --pid=host -it --security-opt=seccomp=unconfined --privileged --rm -v /:/host alpine /bin/sh

chroot /host

set-alias vim "D:\软件\Vim\vim90\.\vim.exe"

$VIMPATH="D:\Vim\vim90\.\vim.exe"
Set-Alias vi $VIMPATH
Set-Alias vim $VIMPATH

apt-get install mysql-server -y


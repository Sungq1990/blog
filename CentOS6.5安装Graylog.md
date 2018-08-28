---
title: CentOS6.5安装Graylog
date: 2018-08-03 16:45:00
tags: Graylog
---

先说下我的环境

```
服务器环境:CentOS release 6.5 (Final)
java:openjdk version "1.8.0_171"
mongo:2.4.4
elasticsearch:2.4.6
graylog-server:2.1.1
```
这里要注意一下(Graylog 2.x does not work with Elasticsearch 5.x）别安装太高版本的elasticsearch
<!--more-->
java和mongo的安装就不介绍了，没有安装的话自己搜索解决

安装elasticsearch

```
sudo rpm --import http://packages.elastic.co/GPG-KEY-elasticsearch
sudo vim /etc/yum.repos.d/elasticsearch.repo
写入以下内容
[elasticsearch-1.7]
name=Elasticsearch repository for 1.7.x packages
baseurl=http://packages.elastic.co/elasticsearch/1.7/centos
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
安装elasticsearch
sudo yum -y install elasticsearch
```
然后就是修改配置文件了

```
sudo vim /etc/elasticsearch/elasticsearch.yml
找到cluster.name一行，取消这一行的注释，并把值改为graylog
network.host改成localhost，只准内网访问
保存并退出
添加到系统服务
chkconfig --add elasticsearch
启动服务
service elasticsearch start
测试elasticsearch是否安装成功
curl http://127.0.0.1:9200/_cluster/health?pretty=true
返回
{
  "cluster_name" : "graylog",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 1,
  "active_shards" : 1,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
则安装成功
```

安装Graylog

```
rpm -Uvh https://packages.graylog2.org/repo/packages/graylog-2.0-repository_latest.rpm
yum install graylog-server
```

修改配置

```
vi /etc/graylog/server/server.conf
password_secret = 0b4e7a0e5fe84ad35fb5f95b9ceeac790b4e7a0e5fe84ad35fb5f95b9ceeac79 //123456
root_password_sha2 =ed02457b5c41d964dbd2f2a609d63fe1bb7528dbe55e1abf5b52c249cd735797 //aaaaaa
web_listen_uri = http://127.0.0.1:9000/
rest_listen_uri = http://127.0.0.1:9000/api/
elasticsearch_index_prefix = graylog
allow_highlighting = true
elasticsearch_cluster_name = graylog
elasticsearch_discovery_zen_ping_unicast_hosts = 127.0.0.1:9300
elasticsearch_network_host = 127.0.0.1

添加到系统服务
chkconfig --add graylog-server
启动服务
service graylog-server start
测试是否启动成功
service graylog-server status
```

下面配置nginx反向代理

```
server {
        listen       80;
        server_name  logCenter.xxx.com;
        location / {
             proxy_set_header Host $http_host;
             proxy_set_header X-Forwarded-Host $host;
             proxy_set_header X-Forwarded-Server $host;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Graylog-Server-URL http://$server_name/api;
             proxy_pass http://graylog-web-cluster;
        }
}

upstream graylog-web-cluster {
    server 127.0.0.1:9000 max_fails=3 fail_timeout=30s;  
}
```

在浏览器上打开logCenter.xxx.com就能看到登录界面了,这里注意graylog-server用的是9000端口，和php-fpm用的同一个端口，所以如果服务起不来可能是端口冲突了，可以改下php-fpm的端口

php7下修改php-fpm端口的方法

```
vi phpPath/etc/php-fpm.conf
listen = 127.0.0.1:9001
保存退出
重启php-fpm
同时记得修改nginx下vhosts php站点的配置
fastcgi_pass   127.0.0.1:9001
平滑重启nginx
修改完毕
```




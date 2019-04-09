---
title: java学习日记(一)--环境搭建
date: 2019-04-09 22:28:54
tags: java
---

#### jdk安装
```
jdk使用1.8，下载地址
https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

环境变量设置(改成自己的安装路径)
vi ~/.bash_profile
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

查看是否生效
java -version
```

#### zookeeper安装(版本3.4.13)
```
下载地址
http://www-eu.apache.org/dist/zookeeper/zookeeper-3.4.13/

配置(改成自己的安装路径)
将con文件夹下的zoo_sample.cfg拷贝一份,重命名为zoo.cfg，修改为
dataDir= /Users/sgq/data/java/zookeeper-3.4.13/data
dataLogDir= /Users/sgq/data/java/zookeeper-3.4.13/logs
clientPort=2181

启动
bin目录下./zkServer.sh
其中./zkCli.sh查看运行状态
```

#### dubbokeeper安装
```
下载源码
git clone https://github.com/dubboclub/dubbokeeper.git

用mysql存储,mysql新增一个database dubbo-monitor
初始化数据库
CREATE TABLE `application` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) NOT NULL DEFAULT '',
  `type` varchar(50) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`),
  UNIQUE KEY `应用名词索引` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
执行根目录下的
install-mysql.sh

在target目录下 监控数据存储和收集
dubbokeeper/target/
/mysql-server/conf 修改这里面的配置
配置监控的zookeeper注册中心
dubbo.application.name=mysql-monitor
dubbo.application.owner=bieber
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.protocol.name=dubbo
dubbo.protocol.port=20884

monitor.collect.interval=10000
#use netty4
dubbo.provider.transporter=netty4
monitor.write.interval=60
#mysql配置 如果是远程的localhost改为mysql所做主机mysql 端口根据需要修改。后面的是你创建用来存储监控的数据库名
dubbo.monitor.mysql.url=jdbc:mysql://localhost:3306/dubbo-monitor
dubbo.monitor.mysql.username=root
dubbo.monitor.mysql.password=123456
dubbo.monitor.mysql.pool.max=10
dubbo.monitor.mysql.pool.min=10

启动监控进程
mysql-dubbokeeper-server bin目录下./start-mysql.sh

管理和UI展示
把target下mysql-dubbokeeper-ui下的war包移到tomcat webapp目录下，重启tomcat，在webapp下发现多了个文件夹dubbokeeper-ui-1.0.1修改dubbokeeper-ui-1.0.1/WEB-INF/classes下的dubbo.propertie

设置zookeeper注册中心 此处需要和上面设置的mysql-monitor一样
dubbo.application.name=common-monitor
dubbo.application.owner=bieber
dubbo.registry.address=zookeeper://127.0.0.1:2181

#use netty4
dubbo.reference.client=netty4

#peeper config  此处可以配置多个zookeeper注册中心可以透视zookeeper服务
peeper.zookeepers=101.89.137.26:2181,101.89.177.224:2181,180.153.53.160:2181,192.168.15.203:2181
peeper.zookeeper.session.timeout=60000

#logger 日志文件
monitor.log.home=/monitor-log

monitor.collect.interval=6000

重新配置以后，删除掉war包。重启tomcat服务即可访问
```




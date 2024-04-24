---
title: Ubuntu单机多版本php切换
date: 2024-04-24 08:00:00
tags: Ubuntu php
---

#### 切换脚本(可以手动设置版本，另外开机时会检测当前cli版本，选择同一版本的fpm启动)

```txt
#!/bin/bash

# 检查是否提供了新版本作为命令行参数
if [ -n "$1" ]; then
    # 如果提供了新版本作为参数
    NEW_PHP_VERSION=$1
else
    # 如果没有提供新版本，则检测当前 PHP CLI 版本
    CURRENT_CLI_VERSION=$(php -v | grep -oP 'PHP [0-9]+\.[0-9]+' | awk '{print $2}')
    NEW_PHP_VERSION=$CURRENT_CLI_VERSION
fi

# 获取所有正在运行的 PHP-FPM 服务
RUNNING_SERVICES=$(systemctl list-units --type service --state running | grep 'php.*fpm.service')

# 从服务列表中提取当前正在运行的 PHP-FPM 版本
CURRENT_PHP_VERSION=$(echo "$RUNNING_SERVICES" | grep -oP 'php[0-9]+\.[0-9]+' | head -n 1)

# 如果找到了正在运行的 PHP-FPM 版本，停止该服务
if [ ! -z "$CURRENT_PHP_VERSION" ]; then
    echo "Stopping current PHP-FPM service (${CURRENT_PHP_VERSION}-fpm)..."
    sudo systemctl stop "${CURRENT_PHP_VERSION}-fpm.service"
fi

# 更新 PHP CLI 替代配置
sudo update-alternatives --set php /usr/bin/php$NEW_PHP_VERSION

# 启动新版本的 PHP-FPM 服务
echo "Starting PHP-FPM service (php${NEW_PHP_VERSION}-fpm)..."
sudo systemctl start "php${NEW_PHP_VERSION}-fpm.service"

echo "Switched PHP CLI to $NEW_PHP_VERSION and (re)started PHP-FPM."
```

#### 配置开机服务

cd /etc/systemd/system

vi run-on-startup.service

```txt
[Unit]
Description=Run script on startup

[Service]
Type=oneshot
ExecStart=/www/switch-php.sh

[Install]
WantedBy=multi-user.target

```
systemctl enable run-on-startup.service

systemctl start run-on-startup.service


#### 使用方法

```txt
./switch-php.sh 8.2 or ./switch-php.sh 7.3
```
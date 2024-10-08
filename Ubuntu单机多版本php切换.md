---
title: Ubuntu单机多版本php切换
date: 2024-04-24 08:00:00
tags: Ubuntu php
---

#### 切换脚本(可以手动设置版本，另外开机时会检测当前cli版本，选择同一版本的fpm启动)

```txt
#!/bin/bash

# 定义可用的 PHP 版本
PHP_VERSIONS=("7.3" "8.2")

# 提示用户选择 PHP 版本
echo "请选择要切换的 PHP 版本："
select VERSION in "${PHP_VERSIONS[@]}"; do
    if [[ -n "$VERSION" ]]; then
        NEW_PHP_VERSION="$VERSION"
        echo "你选择的版本是 PHP $NEW_PHP_VERSION"
        break
    else
        echo "无效的选择，请重试。"
    fi
done

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
sudo update-alternatives --set php "/usr/bin/php${NEW_PHP_VERSION}"

# 启动新版本的 PHP-FPM 服务
echo "Starting PHP-FPM service (php${NEW_PHP_VERSION}-fpm)..."
sudo systemctl start "php${NEW_PHP_VERSION}-fpm.service"

echo "已切换到 PHP CLI 版本 $NEW_PHP_VERSION，并（重新）启动了 PHP-FPM 服务。"
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
./switch-php.sh
```
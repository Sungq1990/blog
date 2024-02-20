---
title: Ubuntu 22.04.2 LTS安装带qt的wkhtmltopdf
date: 2024-02-20 13:16:00
tags: wkhtmltopdf
---

Ubuntu 22.04.2 LTS安装带qt的wkhtmltopdf会报libssl不存在，需要手动安装才行


```
cd /tmp

wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb

wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6-1/wkhtmltox_0.12.6-1.bionic_amd64.deb

apt install ./libssl1.1_1.1.0g-2ubuntu4_amd64.deb

apt install ./wkhtmltox_0.12.6-1.bionic_amd64.deb

apt-get install ttf-wqy-microhei ttf-wqy-zenhei xfonts-wqy

```
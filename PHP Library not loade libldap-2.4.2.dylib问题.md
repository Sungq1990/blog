---
title: PHP Library not loade libldap-2.4.2.dylib问题
date: 2021-12-30 14:22:00
tags: PHP
---

先说下我的环境，机器是Mac，系统是Monterey 12.0.1，之前用的brew安装的php@7.2

可能是之前用brew安装了ffmpeg导致PHP的CLI环境出问题了，在CLI下执行PHP会报

Library not loaded: /usr/local/opt/openldap/lib/libldap-2.4.2.dylib

说下解决方法，就是需要brew重装下php才行

由于我的brew之前重装过，直接执行brew reinstall php@7.2会报错

Error: php@7.2 has been disabled because it is deprecated upstream!

需要执行下
brew tap shivammathur/php
在执行
brew install shivammathur/php/php@7.2
即可

安装完成后问题得到解决！

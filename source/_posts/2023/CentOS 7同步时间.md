---
title: 'CentOS7同步时间'
author: ehzyil
tags:
  - CentOS
categories:
  - Linux
date: 2023-11-11
---

## 问题描述

The difference between the request time and the current time is too large。

由于minio部署在centos的docker容器中，虚拟机挂起再启动后时间不会同步，造成上述问题。

解决方法是同步虚拟机的时间。

### 设置时区（CentOS 7）

先执行命令`timedatectl status|grep 'Time zone'`查看当前时区，如果不是中国时区（Asia/Shanghai），则需要先设置为中国时区，否则时区不同会存在时差。

```bash
#已经是Asia/Shanghai，则无需设置
[root@xiaoz shadowsocks]# timedatectl status|grep 'Time zone'
       Time zone: Asia/Shanghai (CST, +0800)
```

执行下面的命令设置时区

```bash
#设置硬件时钟调整为与本地时钟一致
timedatectl set-local-rtc 1
#设置时区为上海
timedatectl set-timezone Asia/Shanghai
```

### 使用ntpdate同步时间

目前比较常用的做法就是使用ntpdate命令来同步时间，使用方法如下：

```bash
#安装ntpdate
yum -y install ntpdate
#同步时间
ntpdate -u  pool.ntp.org
#同步完成后,date命令查看时间是否正确
date
```
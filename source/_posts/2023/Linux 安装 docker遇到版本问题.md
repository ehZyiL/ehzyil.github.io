---

title: 'Linux 安装 docker遇到版本问题'
author: ehzyil
tags:
  - Linux 
  - Docker
categories:
  - 技术
date: 2023-10-09 
---

# linux 安装 docker遇到版本问题小记

> 实验环境 Centos 7

## linux 小版本更新

docker 对 linux 版本有要求(次要版本 >300 左右). 版本过低会导致一系列: 比如端口映射了,但范围不通的情况

### 找出系统上正在运行的Linux内核版本

```bash
$ uname -srm
Linux 3.10.0-327.el7.x86_64 x86_64
```

Linux 3.10.0-327.el7.x86_64 x86_64 3 - 内核版本. 10 - 主修订版本. 0-957 - 次要修订版本.

### 查询可升级最新版本

```bash
[root@localhost ~]# yum list kernel
已加载插件：fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
已安装的软件包
kernel.x86_64	3.10.0-327.el7 			@anaconda
可安装的软件包
kernel.x86_64 	3.10.0-1160.92.1.el7 	updates  
```

上面我们可以看到可以升级到 `3.10.0-1160.92.1.el7`, 于是执行命令 `yum update -y kernel` 进行小版本升级

### 重启系统

命令 `sudo init 6` 重启系统后, 在查看 linux 内核版本

```bash
[root@localhost uname -srm
Linux 3.10.0-1160.92.1.el7.x86_64 x86_64
```

卸载并重新安装Docker，问题解决。
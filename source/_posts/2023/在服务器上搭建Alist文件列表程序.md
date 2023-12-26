---
title: '在服务器上搭建Alist文件列表程序'
author: ehzyil
tags:
  - 网盘
categories:
  - 记录
date: 2023-10-22
---



在我们使用网盘的时候，有时候需要搭建一个分享页。但是默认的网盘分享页有些麻烦。在这篇教程中，我们来和大家一起在服务器上使用shell脚本一键搭建Alist文件列表程序

## 准备材料

- 服务器一台

## 部署步骤

1. SSH进入服务器的控制台，执行以下命令

```
curl -fsSL "https://alist.nn.ci/v3.sh" | bash -s install
```

2.脚本将会提示访问IP地址、管理员用户名及密码

![951fb88d39325a06ff9b0.png](https://i3.wp.com/telegra.ph/file/951fb88d39325a06ff9b0.png)

3.配置服务器安全组的规则

![9d7603b90af233400df17.png](https://i3.wp.com/telegra.ph/file/9d7603b90af233400df17.png)



4.如需要更新Alist，则使用以下命令

```
SHELL
复制成功curl -fsSL "https://alist.nn.ci/v3.sh" | bash -s update
```

5.如需要卸载Alist，则使用以下命令

```
SHELL
curl -fsSL "https://alist.nn.ci/v3.sh" | bash -s update
```



## 配置Alist

https://alist.nn.ci/zh/guide/drivers/aliyundrive.html

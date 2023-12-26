---
title: 'node的卸载与安装'
author: ehzyil
tags:
  - node.js 
  - 软件
categories:
  - 技术
date: 2023-10-09 
---
## 一、node.js卸载

1、打开cmd 输入下列命令 查看npm包路径 

```bash
#查看全局安装位置
npm root -g
```

得到npm的文件路径

![image-20230628094424573](../../../images/node%E4%B8%8Enpm%E7%9A%84%E5%8D%B8%E8%BD%BD%E4%B8%8E%E5%AE%89%E8%A3%85/image-20230628094424573.png)

下面的命令，可以用于查看本机的`npm`缓存的位置：

```bash
npm config get cache
```

2、删除C:\Users\用户名\AppData\Roaming目录下的`npm`和`npm-cache`；

3、在控制面板中找到Node.js的卸载程序，运行卸载程序。

## 二、安装

1、进入[node.js各版本下载链接](https://nodejs.org/dist/v14.17.3/)

![image-20230628094840343](../../../images/node%E4%B8%8Enpm%E7%9A%84%E5%8D%B8%E8%BD%BD%E4%B8%8E%E5%AE%89%E8%A3%85/image-20230628094840343.png)

找到要安装的版本，下载进行安装

2、查看node.js安装版本和npm版本

```
ehZyiL@DESKTOP-1H567GC MINGW64 ~/Desktop
$ node -v
v18.16.0

ehZyiL@DESKTOP-1H567GC MINGW64 ~/Desktop
$ npm -v
9.5.1
```

cmd进入命令行界面，输入`node -v` 显示node版本，输入`npm -v`显示npm版本，如果都能显示则安装成功。


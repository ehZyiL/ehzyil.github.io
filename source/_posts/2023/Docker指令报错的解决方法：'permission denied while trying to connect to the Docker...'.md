---
title: Docker指令报错的解决方法：'permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock:'
author: ehzyil
tags:
  - Docker
  - linux
categories:
  - 技术
date: 2023-12-26
---

> 
>
> 使用azure的服务器查看docker容器内的进程的时候出现一下错误，
>
> ![](https://2837c11a.ehzyil-img.pages.dev/file/8d5324c018b29df56358f.png)
>
> 意思是试图连接unix:///var/run/docker.sock:，但权限不够。



**原因分析**：这是因为你当前的用户没有这个权限。默认情况下，docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 root 用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root 用户。即我们当前的用户不是root用户。

 **解决办法**：把我们当前的用户添加到docker组中就可以了

第一步：

```
sudo gpasswd -a username docker  #将普通用户username加入到docker组中，username这个字段也可以直接换成$USER。
```

第二步：更新docker组

```
newgrp docker  #更新docker组
```

第三步：再执行你报错的命令，此时就不会报错了。



```bash
ehzyil@AzureVM:~$ docker ps
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json": dial unix /var/run/docker.sock: connect: permission denied
ehzyil@AzureVM:~$ sudo gpasswd -a ehzyil  docker
Adding user ehzyil to group docker
ehzyil@AzureVM:~$ newgrp docker
ehzyil@AzureVM:~$ docker ps
CONTAINER ID   IMAGE                             COMMAND     CREATED       STATUS       PORTS           
```


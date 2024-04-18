---
title: Git连接超时
author: ehzyil
tags:
  - Git
categories:
  - 记录
date: 2024-03-11 20:22:46
headimg:
---

## Git问题：解决“sshconnect to host github.com port 22 Connection timed out”
在写毕设的时候提交代码发现报错，查了查以为公司网禁不能提交代码，后来发现用自己热点也提交不了，于是继续在网上寻找解决方案。

> **Push failed** 
>
> ssh: connect to host github.com port 22: Connection timed out Could not read from remote repository. Please make sure you have the correct access rights and the repository exists.

### 问题原因

可能是由于电脑的防火墙或者其他网络原因导致ssh连接方式端口22被封锁。

### 解决方案

因为端口被封锁，就换成443端口

**操作方法：**

1.进入~/.ssh下

```shell
cd ~/.ssh
```

2.创建一个config文件

3.编辑文件内容：

```shell
Host github.com
User git
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443

Host gitlab.com
Hostname altssh.gitlab.com
User git
Port 443
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
```

4.保存退出

5.检查是否成功`ssh -T git@github.com`

```shell
ehZyiL@DESKTOP-1H567GC MINGW64 ~/.ssh
$ ssh -T git@github.com
Hi ehZyiL! You've successfully authenticated, but GitHub does not provide shell access.
```

---
title: git同时连接gitlab和github
date: 2024-04-01
tags:
  - git
categories:
  - 记录
author: ehzyil
description: 
cover: 
banner:
---

# GIT同时连接gitlab和github

### 1、生成秘钥


公司的 Gitlab 生成一个 SSH-Key

```
# 在~/.ssh/目录会生成id-rsa_lab和id-rsa_lab.pub私钥和公钥。
$ ssh-keygen -t rsa -C "注册的gitlab邮箱" -f ~/.ssh/id_rsa_lab
```

公网 Github 生成一个 SSH-Key

```
# 在~/.ssh/目录会生成id_rsa_hub和id_rsa_hub.pub私钥和公钥。
$ ssh-keygen -t rsa -C "注册的github邮箱" -f ~/.ssh/id_rsa_hub
```


### 2、添加 config

在~/.ssh 下添加 config 配置文件, 内容如下：
```
Host github.com
User git
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_hub
Port 443

Host git.abc.com.cn
Hostname git.abc.com.cn
User git
Port 443
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_lab

```


> `Host`行指定了主机配置的开始。它定义了 SSH 客户端连接到特定远程服务器时的设置。
> 
> `Port` 选项指定了连接到远程服务器的端口号。默认情况下，SSH 使用端口 22，可不配置。
> 
> `User` 选项指定了在远程服务器上使用的用户名，可不配置。
> 
> `HostName` 选项指定了远程服务器的主机名或 IP 地址。
>
> `PreferredAuthentications` 选项指定了 SSH 客户端连接到远程服务器时首选的身份验证方法。`publickey` 表示使用公钥身份验证。
> 
> `IdentityFile` 选项指定了包含用于公钥身份验证的私钥文件的路径。

若不确定config内的内容可以去找一个project复制clone的信息，如下@后的即为Host，一开始配置的是gitlab
```
git@git.abc.com.cn:cpb/cpb7/eps-rhc.git
```


### 3、将公钥添加到 gitlab 服务器和 github 服务器

进入生成的 ssh 目录 : `C:\Users\你的用户名\.ssh`中, 使用Git执行` cat ~/.ssh/id_rsa_lab.pub`命令
如下
```
ehZyiL@DESKTOP-1H567GC MINGW64 ~/.ssh
$ cat ~/.ssh/id_rsa_lab.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC9iuMY1dBZ1TqhjVQHDnf94dAwZqvFKAF2oLSxNuwKwuW6BmVycTKwYrYTWthfLL0HfBJ1LFiMIFpMBxPQzVW6HaH8rjC33N+uwRQPO7xZWtxs39sXHPAthBt3kFRIevnu1cBv85+SnUXydAtzjnNPWIhkB/i3/t7G6cy6PzPyLdQxDT2YaCUpUQ2H9+mgifGzd0rW3UQ8/lhhW35kLM+J17yIq/VNYUY1OEK1t2SMXXcTDe0bKwx8D8hLJdy6XtK4xypg71IMhIWpMpKYp6iwyWyEpZAkctuHPry4bIlnmdnxG3ssGJccZGFvwggSw/LGOIPvyDezidaV1KYz6BYp2oQj1iNYui085lF2qbmHwqR0X8QKkdmKB2jMZKrTPEZG93d3gfeXjXTxUG/w1OVUN+BRt+8kV+27RUMi5xmJIRJjfcOnPCynULfV+ujd4p3HBeAwnglQUjdLVGrY7k/mq+VJXLEap8k6MPj/28QsoCXFwYIcbck6aBCo2FQdz80= xxxx@qq.com
```

登录 GitLab 或 GitHub，选择 Settings
找到 SSH Keys，将上面输出的内容填写到文本域内并保存
![](assets/Pasted%20image%2020240401130706.png)

github的流程相似。
### 4.测试是否配置成功

```
ssh -T git@config配置中HostName的名称
```
在第一次测试时会显示如下内容 输入yes即可

```
C:\Users\ehZyiL\.ssh>ssh -T git@gitlab
The authenticity of host 'git.abc.com.cn (193.193.193.206)' can't be established.
RSA key fingerprint is SHA256:LNrmtcJm70MXzUGdwS9/Rvzoc7ki8PXqk2YTakkDQOc.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'git.abc.com.cn,193.193.193.206' (RSA) to the list of known hosts.
```

测试gitlab
```
C:\Users\ehZyiL\.ssh>ssh -T git@git.abc.com.cn
Enter passphrase for key 'C:\Users\ehZyiL/.ssh/id_rsa_lab':
Welcome to GitLab, @liyuanzhe!
```

测试github
```
C:\Users\ehZyiL\.ssh>ssh -T git@github.com
Hi ehZyiL! You've successfully authenticated, but GitHub does not provide shell access.
```

---
references:
  -  [GIT同时连接gitlab和github - 简书](https://www.jianshu.com/p/88005b56fb4d) 
  - 
---
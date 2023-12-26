---
title: 开启 Azure 虚拟机的 root 账户
author: ehzyil
tags:
  - Azure 
  - linux
categories:
  - 技术
date: 2023-12-26
---

> 使用finalshell连接白嫖的azure虚拟服务器发现root账号忘了，于是查询得知下面的教程。

默认情况微软 Azure 云是没有开启 root 账户的，root 账户是禁用状态。

1. 首先执行：`sudo passwd root` 命令初始化`root`用户密码

   ```
   sudo passwd root
   ```

2. 修改 `sshd_config` 文件 开启 root 访问权限

   ```
   sudo vim /etc/ssh/sshd_config
   ```

3. 在 `sshd_config` 文件里的 `Authentication` 部分加上以下内容：

   ```
   PermitRootLogin yes
   ```

4. 编辑完毕后，重启 ssh 服务，执行如下命令：

   ````
   sudo systemctl restart sshd # 重启 ssh 服务以应用更改
   ````

5. 最后尝试以 root 用户登陆，显示登陆成功，并且用户为root账户。

```bash

```

> 执行的代码和结果如下：

```
ehzyil@AzureVM:~/data/script$ sudo passwd root
New password: 
Retype new password: 
passwd: password updated successfully
ehzyil@AzureVM:~/data/script$ sudo vim /etc/ssh/sshd_config
ehzyil@AzureVM:~/data/script$ sudo systemctl restart sshd
ehzyil@AzureVM:~/data/script$ su
Password: 
root@AzureVM:/home/ehzyil/data/script# bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)

```


---
title: Frp内网穿透
author: ehzyil
tags:
  - 内网穿透
categories:
  - 技术
date: 2024-01-28 20:22:46
headimg:
---


## Frp本地安装

### 查看服务器架构



使用`cat /proc/version`查看服务器架构，可以使用下列的其他两种

```bash
root@iZj6caytd8hmoddao18kslZ:/data# cat /proc/version
Linux version 5.10.0-15-amd64 (debian-kernel@lists.debian.org) (gcc-10 (Debian 10.2.1-6) 10.2.1 20210110, GNU ld (GNU Binutils for Debian) 2.35.2) #1 SMP Debian 5.10.120-1 (2022-06-09)
root@iZj6caytd8hmoddao18kslZ:/data# uname -a
Linux iZj6caytd8hmoddao18kslZ 5.10.0-15-amd64 #1 SMP Debian 5.10.120-1 (2022-06-09) x86_64 GNU/Linux
root@iZj6caytd8hmoddao18kslZ:/data# uname -r
5.10.0-15-amd64
```

下载release包：根据设备和frp版本下载：

```sh
wget https://github.com/fatedier/frp/releases/download/v0.53.2/frp_0.53.2_linux_amd64.tar.gz
```

查看当前文件夹下的文件

```
root@iZj6caytd8hmoddao18kslZ:/data# ls
frp_0.53.2_linux_amd64.tar.gz
```

解压：

```sh
tar -zxvf frp_0.53.2_linux_amd64.tar.gz
```

相关文件：

```
root@iZj6caytd8hmoddao18kslZ:/data# cd frp_0.53.2_linux_amd64/
root@iZj6caytd8hmoddao18kslZ:/data/frp_0.53.2_linux_amd64# ls
frpc  frpc.toml  frps  frps.toml  LICENSE
```

- frps*：是frp服务器相关文件。
- frpc*：是frp客户端相关文件。

### 配置FRP服务端

修改服务端配置：

```sh
vim frps.ini
```

文件内容：

```
# [common] is integral section
[common]
server_addr = 8.210.2.174
# 如果有多个IP，可以选择绑定到不同的ip上
bind_port = 7000

# 虚拟主机配置，不能和系统中已监听的端口冲突。http和https可以设置成同一个
vhost_http_port = 7080
vhost_https_port = 7081

# 服务端web面板
dashboard_addr = 0.0.0.0
dashboard_port = 7500
# 设置用户名密码，默认都是admin，请注意做修改
dashboard_user = admin
dashboard_pwd = Li021712

# 普罗米修斯运维服务，go语言相关监控，可以关闭
enable_prometheus = false

# 设置日志文件地址
log_file = /data/frp/frps.log
# trace, debug, info, warn, error
log_level = info
log_max_days = 3
# disable log colors when log_file is console, default is false
disable_log_color = false
# DetailedErrorsToClient defines whether to send the specific error (with debug info) to frpc. By default, this value is true.
detailed_errors_to_client = true


# 最新版本支持的验证方式比较多，这里还是选用token模式
# AuthenticationMethod specifies what authentication method to use authenticate frpc with frps.
# If "token" is specified - token will be read into login message.
authentication_method = token
# AuthenticateHeartBeats specifies whether to include authentication token in heartbeats sent to frps. By default, this value is false.
authenticate_heartbeats = false
# AuthenticateNewWorkConns specifies whether to include authentication token in new work connections sent to frps. By default, this value is false.
authenticate_new_work_conns = false
# auth token 相当于密码，请注意保护
token = 123456

# 允许配置绑定的端口
# only allow frpc to bind ports you list, if you set nothing, there won't be any limit
# allow_ports = 2000-3000,3001,3003,4000-50000

# pool_count in each proxy will change to max_pool_count if they exceed the maximum value
max_pool_count = 5

# max ports can be used for each client, default value is 0 means no limit
max_ports_per_client = 0

# TlsOnly specifies whether to only accept TLS-encrypted connections. By default, the value is false.
tls_only = false

```



## Docker安装

### 配置FRP服务端

修改服务端配置：

```sh
vim frps.ini
```

文件内容：

```
# [common] is integral section
[common]
server_addr = 8.210.2.174
# 如果有多个IP，可以选择绑定到不同的ip上
bind_port = 7000

# 虚拟主机配置，不能和系统中已监听的端口冲突。http和https可以设置成同一个
vhost_http_port = 7080
vhost_https_port = 7081

# 服务端web面板
dashboard_addr = 0.0.0.0
dashboard_port = 7500
# 设置用户名密码，默认都是admin，请注意做修改
dashboard_user = admin
dashboard_pwd = Li021712

# 普罗米修斯运维服务，go语言相关监控，可以关闭
enable_prometheus = false

# 设置日志文件地址
log_file = /data/frp/frps.log
# trace, debug, info, warn, error
log_level = info
log_max_days = 3
# disable log colors when log_file is console, default is false
disable_log_color = false
# DetailedErrorsToClient defines whether to send the specific error (with debug info) to frpc. By default, this value is true.
detailed_errors_to_client = true


# 最新版本支持的验证方式比较多，这里还是选用token模式
# AuthenticationMethod specifies what authentication method to use authenticate frpc with frps.
# If "token" is specified - token will be read into login message.
authentication_method = token
# AuthenticateHeartBeats specifies whether to include authentication token in heartbeats sent to frps. By default, this value is false.
authenticate_heartbeats = false
# AuthenticateNewWorkConns specifies whether to include authentication token in new work connections sent to frps. By default, this value is false.
authenticate_new_work_conns = false
# auth token 相当于密码，请注意保护
token = 123456

# 允许配置绑定的端口
# only allow frpc to bind ports you list, if you set nothing, there won't be any limit
# allow_ports = 2000-3000,3001,3003,4000-50000

# pool_count in each proxy will change to max_pool_count if they exceed the maximum value
max_pool_count = 5

# max ports can be used for each client, default value is 0 means no limit
max_ports_per_client = 0

# TlsOnly specifies whether to only accept TLS-encrypted connections. By default, the value is false.
tls_only = false

```



### 拉取docker镜像并运行

```
docker run --restart=always --network host -d -v /data/frp/frps.ini:/etc/frp/frps.toml --name frps snowdreamtech/frps
```



### 配置FRP客户端

[下载客户端](https://github.com/fatedier/frp)

```
# FRP客户端
[common]
server_addr = 8.210.2.174
 # 与frps.ini的bind_port一致
server_port = 7000
 # 与frps.ini的token一致
token = 123456

# [一个例子_这里尽量全英文]
# type = 链接协议默认就是tcp不需要改
# local_ip = 如果你要内网穿透的就是本机地址就写127.0.0.1
# local_port = 假设你是要远程桌面(windows)那就在这里写3389
# remote_port = 如果你是按照我博客的步骤填写的是30000-40000，在这里你可以随便写一个30000-40000之间的数字

[forum]
type = tcp
local_ip = 127.0.0.1
local_port = 8080
remote_port = 80

[palword]
type = udp
local_ip = 127.0.0.1
local_port = 8211
remote_port = 36422

```

### 运行

```
frpc.exe -c frpc.ini
```


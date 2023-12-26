---
title: 'CentOS下的软件'
author: ehzyil
tags:
  - 软件
  - CentOS
categories:
  - Linux

date: 2023-10-17
---

## CentOS下MySQL


### MySQL8.0.27 安装



```
[root@localhost ~]# uname -m
x86_64
[root@localhost ~]# cat /etc/redhat-release
CentOS Linux release 7.9.2009 (Core)
#解压
tar -zxvf mysql-8.0.27-1.el7.x86_64.rpm-bundle.tar
tar -xvf mysql-8.0.27-1.el7.x86_64.rpm-bundle.tar\
#安装顺序
rpm -ivh mysql-community-common-8.0.27-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-plugins-8.0.27-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-8.0.27-1.el7.x86_64.rpm
yum install openssl-devel -y
rpm -ivh mysql-community-devel-8.0.27-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-compat-8.0.27-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-8.0.27-1.el7.x86_64.rpm
yum install net-tools
yum install -y perl-Module-Install.noarch
rpm -ivh mysql-community-server-8.0.27-1.el7.x86_64.rpm
```



### [MySQL8.0主从复制的配置](http://t.csdn.cn/mSxNR)

```
//主机
CREATE USER 'ehzyil'@'%' IDENTIFIED BY 'Li021712.';

GRANT REPLICATION SLAVE ON *.* TO 'ehzyil'@'%';

//从机
CHANGE REPLICATION SOURCE TO SOURCE_HOST='192.168.154.141',SOURCE_PORT=3306,SOURCE_USER='ehzyil',SOURCE_PASSWORD='Li021712.',SOURCE_LOG_FILE='mysql-bin.000004',SOURCE_LOG_POS=685;
start slave;

```

​	若查询slave状态，出现以下情况

```
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Connecting to source
                  Master_Host: 192.168.154.141
                  Master_User: ehzyil
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000004
          Read_Master_Log_Pos: 685
               Relay_Log_File: localhost-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000004
             Slave_IO_Running: Connecting
            Slave_SQL_Running: Yes
			...
                Last_IO_Errno: 2061
                Last_IO_Error: error connecting to master 'ehzyil@192.168.154.141:3306' - retry-time: 60 retries: 1 message: Authentication plugin 'caching_sha2_password' reported error: Authentication requires secure connection.
              ...

```

分析报错，应该是连接的用户配置有问题。

所以去主库mysql上查询当前用户信息

SELECT plugin FROM `user` where user = 'ehzyil';

原来是主库repluser的plugin是caching_sha2_password 导致连接不上，修改为mysql_native_password即可解决。

ALTER USER 'ehzyil'@'%' IDENTIFIED WITH mysql_native_password BY 'Li021712.';

```
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SELECT plugin FROM `user` where user = 'ehzyil';
+-----------------------+
| plugin                |
+-----------------------+
| caching_sha2_password |
+-----------------------+
1 row in set (0.00 sec)

mysql> ALTER USER 'ehzyil'@'%' IDENTIFIED WITH mysql_native_password BY 'Li021712.';
Query OK, 0 rows affected (0.00 sec)
```

修改后，重新配置从库binlog位置后重启状态展示如下：

```
Slave_IO_State: Waiting for source to send event
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

看到以上三行配置相同，则主从连接成功。

### [MySQL清除主从复制关系](http://t.csdn.cn/ban9g)

mysql主从复制中，需要将主从复制关系清除，需要取消其从库角色。这可通过执行RESET SLAVE ALL清除从库的同步复制信息、包括连接信息和二进制文件名、位置。从库上执行这个命令后，使用show slave status将不会有输出。
reset slave是各版本Mysql都有的功能，在stop slave之后使用。主要做：
删除master.info和relay-log.info文件；
删除所有的relay log（包括还没有应用完的日志），创建一个新的relay log文件；
从Mysql 5.5开始，多了一个all参数。如果不加all参数，那么所有的连接信息仍然保留在内存中，包括主库地址、端口、用户、密码等。这样可以直接运行start slave命令而不必重新输入change master to命令，而运行show slave status也仍和没有运行reset slave一样，有正常的输出。但如果加了all参数，那么这些内存中的数据也会被清除掉，运行show slave status就输出为空了。

```
mysql> stop slave;
Query OK, 0 rows affected, 2 warnings (0.00 sec)

mysql> reset slave all;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> show slave status\G
Empty set, 1 warning (0.00 sec)
```

## nginx



### CentOS下Nginx的安装


安装 nginx 编译需要依赖 gcc 环境:

```
yum install gcc-c++
```


安装两个安装包pcre和pcre-devel：

```
yum install -y pcre pcre-devel
```

安装zlib，Nginx的各种模块中需要使用gzip压缩：

```
yum install -y zlib zlib-devel
```

如果使用了 https，需要安装 OpenSSL 库。安装指令如下：

```
yum install -y openssl openssl-devel
```

一键安装上面四个依赖（追求安装速度可直接用这条指令）

```
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

进入usr/local 文件路径(路径可自选)创建一个文件夹nginx保存使用wget下载的nginx压缩包

```
//下载nginx压缩包
wget http://nginx.org/download/nginx-1.24.0.tar.gz
tar -zxvf nginx-1.24.0.tar.gz -C /usr/local/

cd /usr/local/nginx-1.24.0/
//创建一个文件夹
mkdir /usr/local/nginx
//安装前检查工作
./configure --prefix=/usr/local/nginx
//安装
make && make install

//文件目录树状图
nginx
├── conf							  <-- Nginx配置文件
│   ├── fastcgi.conf
│   ├── fastcgi.conf.default
│   ├── fastcgi_params
│   ├── fastcgi_params.default
│   ├── koi-utf
│   ├── koi-win
│   ├── mime.types
│   ├── mime.types.default
│   ├── nginx.conf						 <-- 这个文件我们经常操作
│   ├── nginx.conf.default
│   ├── scgi_params
│   ├── scgi_params.default
│   ├── uwsgi_params
│   ├── uwsgi_params.default
│   └── win-utf
├── html								 <-- 存放静态文件，我们后期部署项目，就要将静态文件放在这
│   ├── 50x.html
│   └── index.html						  <-- 提供的默认的页面
├── logs
└── sbin
    └── nginx							  <-- 启动文件

```

**配置环境变量**

- 使用`vim /etc/profile`命令打开配置文件，并配置环境变量，保存并退出
- 之后重新加载配置文件，使用`source /etc/profile`命令，然后我们在任意位置输入`nginx`即可启动服务，`nginx -s stop`即可停止服务

**Nginx常用命令**

1. 启动Nginx：` nginx`    启动Nginx服务器。
2. 停止Nginx：` nginx -s stop`    停止正在运行的Nginx服务器。
3. 重新加载Nginx配置：` nginx -s reload`    重新加载Nginx的配置文件，使新的配置生效，而无需停止服务器。
4. 检查Nginx配置是否正确：` nginx -t`    检查Nginx的配置文件语法是否正确，如果有错误，它会显示错误信息。
5. 查看Nginx版本：`nginx -v` 或 `nginx -V` 第一个命令会显示Nginx的版本号，第二个命令会显示编译时的详细配置信息。
6. 查看Nginx进程状态：` nginx -s status`    显示Nginx进程的状态，包括正在运行的进程数量和其它相关信息。
7. 重新打开日志文件：` nginx -s reopen`    重新打开Nginx的日志文件，可以用于实现日志文件的切割和归档。



## CentOS 7



### CentOS 7上防火墙设置

在CentOS 7上设置防火墙可以使用`firewalld`服务进行管理。

以下是在CentOS 7上设置防火墙的一些常用操作：

1. 检查防火墙状态：

```
sudo firewall-cmd --state
```

这将显示防火墙的当前状态，如果防火墙已启用，它会显示"running"。

2. 查看防火墙规则：

```
sudo firewall-cmd --list-all
```

这将显示防火墙的当前规则列表。

3. 启动防火墙服务：

```
sudo systemctl start firewalld
```

4. 停止防火墙服务：

```
sudo systemctl stop firewalld
```

5. 设置防火墙开机启动：

```
sudo systemctl enable firewalld
```

6. 停用防火墙开机启动：

```
sudo systemctl disable firewalld
```

7. 开放端口：

```
sudo firewall-cmd --zone=public --add-port=<port>/tcp --permanent
```

将`<port>`替换为要开放的端口号。此命令会永久性地在防火墙中开放指定的端口。

8. 移除已开放的端口：

```
sudo firewall-cmd --zone=public --remove-port=<port>/tcp --permanent
```

将`<port>`替换为要移除的端口号。此命令会永久性地从防火墙中移除指定的端口。

9. 重新加载防火墙配置：

```
sudo firewall-cmd --reload
```

在修改防火墙规则后，使用此命令重新加载配置，使更改生效。

10.验证规则是否生效：

```
sudo firewall-cmd --zone=public --list-all
```

检查规则列表以确认添加或删除的规则已正确应用。

### 查看yum已安装的包

在linux下如何使用yum查看安装了哪些软件包

#### 列出所有已安装的软件包

```bash 
yum list installed
```

yum针对软件包操作常用命令： 

1.使用 yum 查找软件包 
命令：`yum search `
2.列出所有可安装的软件包 
命令： `yum list  `
3.列出所有可更新的软件包 
命令： `yum list updates  `
4.列出所有已安装的软件包 
命令： `yum list installed `

####  yum 语法

```
yum [options] [command] [package ...]
```

- **options** 可选，选项包括 -h（帮助），-y（当安装过程提示选择全部为"yes"），-q（不显示安装的过程）等等。
- **command** 要进行的操作。
- **package** 操作的对象。

其它的例子就不列举了，这里说一下查看yum安装了哪些软件包

查看 yum 已安装 docker  的包

```
yum list installed docker
```

查看 yum 已安装 docker相关的包

```
 yum list installed docker*
```

```
[root@iZ2ze552628d2crm1m97ppZ ~]# yum list installed docker*
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
已安装的软件包
docker-buildx-plugin.x86_64                                                                                0.11.2-1.el7                                                                             @docker-ce-stable
docker-ce.x86_64                                                                                           3:24.0.6-1.el7                                                                           @docker-ce-stable
docker-ce-cli.x86_64                                                                                       1:24.0.6-1.el7                                                                           @docker-ce-stable
docker-ce-rootless-extras.x86_64                                                                           24.0.6-1.el7                                                                             @docker-ce-stable
docker-compose-plugin.x86_64                                                                               2.21.0-1.el7                                                                             @docker-ce-stable
[root@iZ2ze552628d2crm1m97ppZ ~]# 

```


---
title: PostgreSQL 使用记录
author: ehzyil
tags:
  - 数据库
  - PostgreSQL
  - Docker
categories:
  - 记录
date: 2024-3-22
headimg:
---

## PostgreSQL 使用记录

## Docker安装PostgreSQL 

**安装 Docker**

1. 访问 Docker 官方网站：https://docs.docker.com/desktop/

**拉取 PostgreSQL 镜像**

1. 打开终端或命令提示符。
2. 运行以下命令拉取 PostgreSQL 镜像：

```
docker pull postgres:13.0
```

**创建 PostgreSQL 容器**

运行以下命令创建 PostgreSQL 容器：

```
docker run --name postgres_db -e POSTGRES_PASSWORD=postgresql -e POSTGRES_USER=ehzyil -e POSTGRES_DB=freshrss -v /data/postgresql:/var/lib/postgresql/data -p 5433:5432 -d postgres
```

- `--name postgres_db`：将创建的容器命名为 `postgres_db`。
- `-e POSTGRES_PASSWORD=postgresql`：设置环境变量 `POSTGRES_PASSWORD`，该环境变量定义了PostgreSQL数据库的超级用户密码，这里设置为 `postgresql`。
- `-e POSTGRES_USER=ehzyil`：设置环境变量 `POSTGRES_USER`，以创建具有超级用户权限的新用户，这里用户名设为 `ehzyil`。
- `-e POSTGRES_DB=freshrss`：设置环境变量 `POSTGRES_DB`，以创建一个名为 `freshrss` 的新数据库。
- `-v /data/postgresql:/var/lib/postgresql/data`：将宿主机的 `/data/postgresql` 目录挂载到容器内的 `/var/lib/postgresql/data` 目录。这样做可以保证数据库的数据即使在容器停止后也能够持久化。
- `-p 5432:5432`：将宿主机的 `5432` 端口映射到容器内的 `5432` 端口，允许您从宿主机的该端口连接到PostgreSQL服务。
- `-d postgres`：以守护进程模式在后台启动 `postgres` 镜像的容器。 

**连接到容器**

1. 运行以下命令连接到容器：

```
docker exec -it postgres_db psql -U ehzyil -d freshrss
```

- `docker exec -it some-postgres`：`docker exec` 命令允许您在运行中的容器里执行命令，`-it` 参数让您可以交互式地使用容器的命令行接口。
- `psql`：是PostgreSQL的命令行工具。
- `-U ehzyil`：`-U` 参数用于指定要以哪个用户身份登录，这里您应使用您创建容器时设置的用户`ehzyil`。
- `-d freshrss`：`-d` 参数用于指定要连接的数据库, 这里是您创建的数据库`freshrss`。 如果您的宿主机上安装有psql客户端或其他数据库管理工具，您也可以直接使用本机的管理工具连接到容器的数据库。假设您的容器正运行在同一台宿主机上，您可以使用如下命令：

2. 已连接到容器，可以使用 `psql` 命令连接到 PostgreSQL 数据库：

```
psql -U postgres
```

3. 执行 SQL 查询，例如：

```
select now();
```





## 创建新的数据库用户

1. 进入数据库命令行

首先，以 PostgreSQL 用户身份登录，然后使用 `psql` 命令进入数据库命令行：

```
sudo su postgres
psql
```

2. 创建新用户

使用以下命令创建新的数据库用户 `dbuser`，并设置密码 `<CUSTOM PASSWORD>`：

```
CREATE USER dbuser WITH PASSWORD '<CUSTOM PASSWORD>';
```

3. 创建数据库

创建名为 `exampledb` 的新数据库，并将其所有者设置为 `dbuser`：

```
CREATE DATABASE exampledb OWNER dbuser;
```

4. 授予数据库权限

将 `exampledb` 数据库的所有权限授予 `dbuser`：

```
GRANT ALL PRIVILEGES ON DATABASE exampledb TO dbuser;
```


5. 授予表权限（可选）

如果需要授予 `dbuser` 对特定表的读写权限，请使用以下命令：

```
GRANT ALL PRIVILEGES ON TABLE mytable TO dbuser;
```
授予当前库所有表的权限
```
GRANT ALL PRIVILEGES ON all tables in schema public TO dbuser;
```
**注意：**

- 授予权限的命令必须在要操作的数据库中执行。

- 授予所有权限时，请谨慎操作，因为它会授予用户对数据库的完全控制权。

- 对于生产环境，建议使用更细粒度的权限授予策略。

  

6.删除角色

```
drop role memos; # 删除角色
```


### 删除数据库


1. **查找正在使用数据库的会话**：首先，找出哪些会话正在使用您想要删除的数据库。    
    `SELECT pid, usename, datname, query, state FROM pg_stat_activity WHERE datname = 'memos';`
    
    这将列出所有正在使用 `memos` 数据库的会话及其详细信息。
    
2. **终止会话**：知道了这些会话的 `pid`（即进程 ID），可以使用 `pg_terminate_backend` 函数来终止它们：
    
    `SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'memos';`
    
    这将终止所有正在使用 `memos` 数据库的会话。
    
3. **删除数据库**：在所有相关会话被终止后，能够成功删除数据库：
    
    `DROP DATABASE memos;`

## 如何切换用户

> 在PostgreSQL数据库中，切换用户通常指的是以不同的用户身份登录数据库。这可以通过以下几种方式实现：


1. **使用命令行或终端**：  
    如果你已经以某个用户身份登录了PostgreSQL，并且想要切换到另一个用户，你可以退出当前会话，然后使用新用户的凭据重新登录。例如：
    
    `psql -U 新用户名 -d 数据库名`
    其中，`新用户名`是你想要切换到的用户名称，`数据库名`是你想要连接的数据库。
2. **在psql会话中**：  
    如果你已经通过`psql`命令行工具登录到PostgreSQL，你可以使用`\c`命令来切换数据库和用户。例如：
    
    `\c 数据库名 新用户名`
    
    这将切换到指定的数据库，并尝试以`新用户名`登录。如果该用户没有访问该数据库的权限，切换将失败。

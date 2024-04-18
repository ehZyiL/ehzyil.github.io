---
title: Oracle数据库使用记录
author: ehzyil
tags:
  - 数据库
  - Oracle
categories:
  - 记录
date: 2024-3-22
headimg:
---

## Oracle数据库使用记录



### 一、Oracle忘记用户名和密码

　　1、打开命令提示符，输入命令sqlplus ,进入oracle控制台

　　2、用户名输入 sqlplus / as sysdba，口令：空（回车即可）

　　3、连接成功后，输入“select username from dba_users”查看用户列表

　　4、若修改某一个用户密码， 修改用户口令 格式为（注意后面的分号；）：

　　alter user 用户名 identified by 新密码；

　　以system 为例，密码修改为 123456. 可输入

　　alter user system identified by 123456;

例如

```csharp
sqlplus / as sysdba ---------sysdba为超级用户
alter user lxemr account unlock; --------- 解除锁定(必须带“;”号，注意用英文字符)
alter user lxemr  identified by lxemr12345; -------------修改密码
    alter user 'WF_TRAIN'  identified by WF_TRAIN;
```

### 二、Oracle 用户解锁

　　SQL> ALTER USER 用户名 ACCOUNT UNLOCK;

### 三、账号解锁

>  Oracle 账号被锁定，显示错误 "ORA-28000: the account is locked"。

**解决方案：**

**1. 登录数据库**

```sql
sqlplus / as sysdba
```

使用 sysdba 权限登录数据库。

**2. 查看用户列表**

```sql
select username from dba_users;
```

这将显示数据库中所有用户的列表。

**3. 解锁用户**

```sql
ALTER USER USER_NAME ACCOUNT UNLOCK;
```

将 "USER_NAME" 替换为要解锁的用户的名称。

**4. 修改登录尝试限制**

```sql
alter profile default limit failed_login_attempts unlimited;
```

修改默认配置文件，允许无限次登录尝试。



### 四、IDEA/Datagtrip连接Oracle



**先决条件：**

- 已安装 IDEA 或 DataGrip
- 已安装 Oracle 数据库并配置了监听器(默认有配置)
- 已创建 Oracle 数据库用户和密码

**步骤：**

**1. 创建数据源（IDEA）**

- 打开 IDEA，转到 "Database" > "Data Sources"。
- 单击 "加号" 图标 (+)。
- 选择 "Oracle" 作为数据库类型。



**2. 配置连接参数**

- **URL：**输入 Oracle 数据库的 JDBC URL。格式为：

  ```
  jdbc:oracle:thin:@<主机名>:<端口号>/<服务名>
  ```

  例如：

  ```
  jdbc:oracle:thin:@localhost:1521/XE
  ```

- **用户名：**输入 Oracle 数据库用户的用户名。

- **密码：**输入 Oracle 数据库用户的密码。



**3. 获取 Oracle 数据库实例的 SID**

**使用 SQLPlus**

```sql
SQL> SELECT INSTANCE_NAME FROM V$INSTANCE;

INSTANCE_NAME
----------------
orcl
```

**解释：**

- `SQLPlus` 是 Oracle 提供的命令行工具，用于与 Oracle 数据库交互。
- `SELECT INSTANCE_NAME FROM V$INSTANCE;` 查询 `V$INSTANCE` 视图以获取当前数据库实例的名称。
- `INSTANCE_NAME` 列显示数据库实例的名称，在本例中为 "orcl"。



**4. 填写用户名密码和端口**

**填充后的 URL 显示如下：**

```
jdbc:oracle:thin:@localhost:1521:orcl
```

**解释：**

- `jdbc:oracle:thin:` 是 Oracle JDBC 驱动程序的 URL 前缀。
- `@localhost` 是数据库服务器的主机名或 IP 地址。
- `1521` 是数据库服务器的监听器端口。
- `orcl` 是数据库实例的 SID，在前面的步骤中已获取。



**5. 测试连接**

- 单击 "Test Connection" 按钮。
- 如果连接成功，您将看到 "Connection successful" 消息。



### 五.oracle导入`.dmp`文件



#### **1.使用 Oracle `imp` 实用程序导入 .dmp 文件的步骤：**

1. 确保您具有导入到目标数据库的适当权限（DBA）。
2. 打开命令提示符或终端窗口。
3. 导航到包含 .dmp 文件的目录。
4. 使用以下语法运行 `imp` 命令：

```
imp username/password@database_name file=path/to/dump_file.dmp [options]
```

**选项：**

* **FULL=Y**：导入所有对象，包括用户、表、索引和约束。
* **FROMUSER/TOUSER**：指定要导入对象的源用户和目标用户。
* **TABLES**：指定要导入的特定表列表。
* **IGNORE=Y**：忽略导入期间遇到的错误。
* **LOG=path/to/log_file.log**：将导入日志写入指定的文件。

#### **2.示例：**

假设您要将名为 `my_dump.dmp` 的文件导入到名为 `new_db` 的数据库中，并且您希望将对象导入到名为 `scott` 的用户中，可以使用以下命令：

```
imp scott/tiger@new_db file=my_dump.dmp fromuser=scott touser=scott
```

**注意事项：**

* 如果导出的文件包含由其他用户创建的对象，则您需要使用 `FROMUSER/TOUSER` 选项指定目标用户。
* 如果您使用 `FULL=Y` 选项，则导入将覆盖目标数据库中的现有对象。
* 导入过程可能需要一段时间，具体取决于导出的文件大小和目标数据库的性能。
* 导入完成后，您可以使用以下查询检查导入的对象：

```
SELECT * FROM dba_objects WHERE owner = '目标用户';
```



#### 3.导入过程

第一句是从网上复制的没修改账号密码因此又输入了一遍，17行错误表明在使用 `imp` 实用程序导入 .dmp 文件时未指定必需的参数。

```
imp prophet/prophet@orcl file=C:\Users\ehZyiL\Desktop\train20201130.dmp

Import: Release 11.2.0.4.0 - Production on 星期二 3月 12 20:24:17 2024

Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.


IMP-00058: 遇到 ORACLE 错误 1017
ORA-01017: invalid username/password; logon denied用户名: scott
口令:

连接到: Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

经由常规路径由 EXPORT:V11.02.00 创建的导出文件

警告: 这些对象由 TRAIN 导出, 而不是当前用户

已经完成 ZHS16GBK 字符集和 AL16UTF16 NCHAR 字符集中的导入
IMP-00031: 必须指定 FULL=Y 或提供 FROMUSER/TOUSER 或 TABLES 参数
IMP-00000: 未成功终止导入
```

在指定导入参数后正常导入了

```
C:\Users\ehZyiL>imp prophet/prophet@orcl file=C:\Users\ehZyiL\Desktop\train20201130.dmp   full=y

Import: Release 11.2.0.4.0 - Production on 星期二 3月 12 20:25:52 2024

Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.


IMP-00058: 遇到 ORACLE 错误 1017
ORA-01017: invalid username/password; logon denied用户名: scott
口令:

连接到: Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

经由常规路径由 EXPORT:V11.02.00 创建的导出文件

警告: 这些对象由 TRAIN 导出, 而不是当前用户

已经完成 ZHS16GBK 字符集和 AL16UTF16 NCHAR 字符集中的导入
. 正在将 TRAIN 的对象导入到 SCOTT
```

https://blog.csdn.net/weixin_45740811/article/details/125841798





### 六、新建数据库

**使用 Oracle 命令行界面 (CLI) 创建 WF-TRAIN 数据库**

1. **打开命令提示符或终端窗口。**
2. **连接到数据库服务器。**
3. **运行以下命令：**

```
CREATE DATABASE WF-TRAIN
   GLOBAL UNDO
   ON <disk_group>
   DATAFILE '<datafile_location>' SIZE <size_in_MB>
   LOGFILE GROUP <log_group_name> ('<log_file_location>' SIZE <size_in_MB>)
   CHARACTER SET <character_set>
   TEMPLATE <template_name>
```

**示例：**

```
CREATE DATABASE WF-TRAIN
   GLOBAL UNDO
   ON DATA
   DATAFILE '/data/wf_train_datafile.dbf' SIZE 100M
   LOGFILE GROUP wf_train_log_group ('/data/wf_train_log_file.dbf' SIZE 20M)
   CHARACTER SET UTF8
   TEMPLATE basic
```

1. **按 Enter 键。**

**注意：**

- 替换 `<disk_group>`、`<datafile_location>`、`<log_group_name>`、`<log_file_location>`、`<character_set>` 和 `<template_name>` 处的占位符。
- 确保您具有创建数据库所需的权限。



### 七、处理'ORA-01950: 对表空间 'USERS' 无权限
> 当我们使用任何形式的数据库时，遇到错误是不可避免的。Oracle数据库常见的一个错误是`ORA-01950: 对表空间 'USERS' 无权限`。

**理解ORA-01950：造成ORA-01950的原因**

ORA-01950错误是一个标志，表示Oracle模式没有足够的权限对表空间执行特定的操作。Oracle通过特定的安全层次结构指定权限，用户配额管理是其中的一个组成部分。



**以SYSDBA权限登录**
通过以SYSDBA权限登录Oracle，你可以获得允许你执行高级操作的管理权限。

```markdown
sqlplus / as sysdba
```
"/ as sysdba"命令不需要任何密码，允许你直接以SYSDBA权限访问Oracle。



**更改用户配额**
一旦登录，你可以使用以下命令更改用户配额：

```markdown
SQL> alter user "WF-TRAIN" quota unlimited on "USERS";
```
这意味着你正在给WF-TRAIN用户在USERS表空间上授予无限的空间。

**更改成功**
正确运行命令后，Oracle将返回：

```markdown
用户已更改。
```
这个响应表示配额分配成功。

**总结**

- **什么是ORA-01950错误？**
  - 当用户在表空间没有足够的权限时，将触发ORA-01950错误。
- **什么导致了'ORA-01950: 对表空间 'USERS' 无权限'错误？**
  - 当用户在表空间没有足够的空间时，会出现此错误。

### 八、如何关闭本地启动的 Oracle

```
sqlplus / as sysdba 

SHUTDOWN IMMEDIATE
```
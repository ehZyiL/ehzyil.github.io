---
title: MySQL开启安全模式产生的错误
author: ehzyil
tags:
  - MySQL
categories:
  - 记录
date: 2024-03-11 20:22:46
headimg:
---

## MySQL错误：ERROR 1175: You are using safe update mode解决方法

```
Error updating database. Cause: java.sql.SQLException: You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column ### The error may involve UserMapper.unCollectStaffTeam-Inline 
### The error occurred while setting parameters 
### SQL: delete from t_qm_staff_collect WHERE STAFF_ID = ? AND ORGA_ID in ( ? ) 
### Cause: java.sql.SQLException: You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column ; 
uncategorized SQLException for SQL [];
SQL state [HY000]; error code [1175]; 
You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column; 
nested exception is java.sql.SQLException: You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column
```

**错误信息：**

```
你正在使用安全更新模式，你试图更新一个没有使用 KEY 列的 WHERE 子句的表
```

**错误详情：**

* **错误信息：** 你正在使用安全更新模式，你试图更新一个没有使用 KEY 列的 WHERE 子句的表
* **SQL 状态：** HY000
* **错误代码：** 1175

**原因：**

mysql有个叫SQL_SAFE_UPDATES的变量，为了数据库更新操作的安全性，此值默认为1或ON，所以才会出现更新失败的情况。

当你尝试在 MySQL 数据库上执行一个没有使用 WHERE 子句指定要更新哪些行的更新语句时，就会发生此错误。

查看设置：

```
mysql> show variables like 'sql_safe%';

+------------------+-------+

| Variable_name    | Value |

+------------------+-------+

| sql_safe_updates | ON    |

+------------------+-------+
```

下面是SQL_SAFE_UPDATES变量为0和1时的取值说明：

SQL_SAFE_UPDATES有两个取值0和1， 或ON 和OFF;

SQL_SAFE_UPDATES = 1，ON时，不带where和limit条件的update和delete操作语句是无法执行的，即使是有where和limit条件但不带key column的update和delete也不能执行。 

SQL_SAFE_UPDATES =0，OFF时，update和delete操作将会顺利执行。那么很显然，此变量的默认值是1。 

所以，出现1175错误的时候，可以先设置SQL_SAFE_UPDATES的值为0 OFF，然后再执行更新;

以下2条命令都可以修改配置；

mysql> set sql_safe_updates=0; 

mysql> set sql_safe_updates=off;  

修改后发现并没有用。

或者修改的配置并没有生效。

**解决方案：**

原来的逻辑是查询符合条件的直接删除，sql语句：

```
<delete id="unCollectstaffTeam" parameterType="java.util.Map">
  <where>
    AND STAFF_ID = #{staffId}
    AND ID IS NOT NULL
    AND ORGA_ID IN
    <foreach item="item" index="index" collection="orgIds" open="(" separator=" " close=")">
      #{item}
    </foreach>
  </where>
</delete>

```

修改后的逻辑先查询符合条件的id，再根据id进行删除，sql语句：

```
<delete id="unCollectStaffTeam" parameterType="java.util.Map">
  <where>
    ID in
    <foreach item="item" index="index" collection="queryCollectStaffTeamIds" open="(" separator="," close=")">
      #{item}
    </foreach>
  </where>
</delete>

<select id="queryCollectStaffTeamIds" parameterType="java.util.Map" resultType="java.util.Map">
  <where>
    AND STAFF_ID = #{staffId}
    AND ORGA_ID in
    <foreach item="item" index="index" collection="orgIds" open="(" separator="," close=")">
      #{item}
    </foreach>
  </where>
</select>

```


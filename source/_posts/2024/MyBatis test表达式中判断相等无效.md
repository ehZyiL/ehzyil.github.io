---
title: MyBatis test表达式中判断相等无效
author: ehzyil
tags:
  - Mybatis
categories:
  - 记录

date: 2024-03-11 20:22:46
headimg:
---

1. 现象

想通过传入一个remarkflag标志，当remarkflag为"0"时，查询数据库中remark列为空字符串的行，当remarkflag为"1"时，查询数据库中某字段为非空字符串的行。代码如下：

```
  <if test="remarkflag != '' and remarkflag == '0'">

      AND remark =''

     </if>

     <if test="remarkflag != '' and remarkflag == '1'">

      AND remark !=''

     </if>
```



结果不论remarkflag设为"0"还是"1"，上面两个判断均不成立。

2. 问题处理

查了许多文档，发现当判断的单引号内只有一个字符时，会被识别为Java语言中的char类型，上述remarkflag==‘0’，左边是字符串，右边是字符型，所以不成立，同理remarkflag == '1’也不成立。

3. 解决

强制转换下类型后，问题解决：

```


     <if test="remarkflag != '' and remarkflag == '0'.toString()">

      AND remark =''

     </if>

     <if test="remarkflag != '' and remarkflag == '1'.toString()">

      AND remark !=''

     </if>
```


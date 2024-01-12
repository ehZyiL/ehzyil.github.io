---
title: 使用StringUtils.isNotEmpty遇到的问题
author: ehzyil
tags:
  - Java
  - 踩坑
categories:
  - 记录
date: 2024-01-12
headimg:
---


> 在判断map中的key的时候遇到的问题

下面的代码想要判断params中的键`dashes`是否存在，传入的空字符串，调用`StringUtils.isNotEmpty(dashes)`方法却返回true，即不为空导致实际业务出现了问题。

```java
        HashMap<String, Object> params = new HashMap<>();
        params.put("阿萨", "");
        Object o = params.get("dashes");
        System.out.println(o);  //空
        String dashes = String.valueOf(o);
        System.out.println(dashes); // null
        System.out.println(StringUtils.isNotEmpty(dashes)); //true
        System.out.println(StringUtils.isNotBlank(dashes)); //true
        if (StringUtils.isNotEmpty(String.valueOf(params.get("dashes")))) {
            //可以进来
            System.out.println("可以进来");
        }
```

通过查看`StringUtils.isNotEmpty()`的源码

```
    public static boolean isNotEmpty(CharSequence cs) {
        return !isEmpty(cs);
    }
    public static boolean isEmpty(CharSequence cs) {
        return cs == null || cs.length() == 0;
    }
```

并没有发现什么问题，以为这个方法不能检查出问题，于是网上搜索了相关问题，发现并不是`StringUtils.isNotEmpty()`的问题。问题要归结于`String.valueOf(o)`方法，下面看看其源码：

```java
//当参数是Integer、Long等范型时
public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();
}
//当参数为基本类型时,如int
public static String valueOf(int i) {
    return Integer.toString(i);
}
```

**因此当参数是Integer、Long等泛型时，如果参数为null，此方法会会把参数转化成"null"**

**所以如果用StringUtils.isNotEmpty去判断使用String.ValueOf转化成"null"的参数，是true的，因为此方法并没有去判断传入值为"null"的情况。**



> 其实在判断一个map中的key的时候应该先判断key是否存在，再去判断它的值，下面要判断map中的一个key对应的值是否是"true"，可以使用：

```java
HashMap<String, Object> params = new HashMap<>();
params.put("dashes", "true");
if (params.containsKey("dashes") && StringUtils.isNotEmpty(params.get("dashes").toString())) {
    System.out.println("1");
}
if (params.containsKey("dashes") && "true".equals(params.get("dashes").toString())) {
    System.out.println("2");
}

Object o = Optional.ofNullable(params.get("dashes")).orElse("");
if (StringUtils.isNotEmpty(o.toString())) {
    System.out.println("3");
}
if ("true".equals(o.toString())) {
    System.out.println("4");
}
```

只有当`params.put("dashes", "true");`传入的值为"true"时才会进入if结构内。
---
title: Java String hashCode生成负值
author: ehzyil
tags:
  - Java
categories:
  - 技术
date: 2023-12-26
---



> 转载 [java String hashCode遇到的坑](https://www.cnblogs.com/qixing/p/12354378.html)

在进行数据交换时，如果主键不是整型，需要对字符串，或联合主键拼接为字符串，进行hash，再进行取模分片，使用的是String自带的hashCode()方法，本来是件很方便的事，但是有些字符串取hashCode竟然是负数，使得分片为负数，找不到对应的分片，我们先看一下String 生成hashCode的代码：

```java
/**
     * Returns a hash code for this string. The hash code for a
     * {@code String} object is computed as
     * <blockquote><pre>
     * s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
     * </pre></blockquote>
     * using {@code int} arithmetic, where {@code s[i]} is the
     * <i>i</i>th character of the string, {@code n} is the length of
     * the string, and {@code ^} indicates exponentiation.
     * (The hash value of the empty string is zero.)
     *
     * @return  a hash code value for this object.
     */
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```

主要是根据字符串中字符的ascii码值来计算的，即 31 * hash + 字符的ASCII码值，int型的值取值范围为Integer.MIN_VALUE(-2147483648)～Integer.MAX_VALUE(2147483647)，所以如果字符串比较长，计算的数值就可能超出Integer.MAX_VALUE，造成数值溢出，值变成负数

几种比较极端的字符串hashCode值：

```java
String hashStr0 = "35953305172933/";
        System.out.println(hashStr0.hashCode());            // 2147483647   Integer.MAX_VALUE
        System.out.println(Math.abs(hashStr0.hashCode()));  // 2147483647   Integer.MAX_VALUE
        System.out.println("-------------------");
        String hashStr = "359533051729330";
        System.out.println(hashStr.hashCode());             // -2147483648  Integer.MIN_VALUE
        System.out.println(Math.abs(hashStr.hashCode()));   // -2147483648  Integer.MIN_VALUE
        System.out.println("-------------------");
        String hashStr2 = "56800004874";
        System.out.println(hashStr2.hashCode());            // -2082984168
        System.out.println(Math.abs(hashStr2.hashCode()));  // 2082984168
        System.out.println("-------------------");
        String hashStr3 = "";
        System.out.println(hashStr3.hashCode());            // 0
        System.out.println(Math.abs(hashStr3.hashCode()));  // 0
        System.out.println("-------------------");
```

**当 key.hashCode() 值为 Interger.MIN_VALUE时候Maths.abs(key.hashCode()) 取 hash值为负。**

对于字符串“359533051729330”的hashCode为Integer.MIN_VALUE，我们使用取绝对值还是超出Integer.MAX_VALUE，还是Integer.MIN_VALUE，所以针对这种极端情况是不可用的

要想利用hashCode为非负数，可以Integer.MAX_VALUE和与操作，这样最大正整数的符号位为0，与任何数与操作都是0，即是正数

```
int hash = str.hashCode() & Integer.MAX_VALUE;
```






---
title: Java数组转List的三种方式
author: ehzyil
tags:
  - Java
categories:
  - 技术
date: 2023-12-26
---

## Java数组转List的三种方式



> 需求中常用到数组转集合的操作,为了避免不出错并使用高效方法去网上找了篇非常不错的文章。
>
> 转载 [Java数组转List的三种方式及对比](http://t.csdnimg.cn/8Qsh0)



### 一.最常见方式（未必最佳）
通过 Arrays.asList(strArray) 方式,将数组转换List后，不能对List增删，只能查改，否则抛异常。

关键代码： List list = Arrays.asList(strArray);

```
private void testArrayCastToListError() {
		String[] strArray = new String[2];
		List list = Arrays.asList(strArray);
		//对转换后的list插入一条数据
		list.add("1");
		System.out.println(list);
	}
```


执行结果：

```
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.AbstractList.add(AbstractList.java:148)
	at java.util.AbstractList.add(AbstractList.java:108)
	at com.darwin.junit.Calculator.testArrayCastToList(Calculator.java:19)
	at com.darwin.junit.Calculator.main(Calculator.java:44)
```

程序在list.add(“1”)处，抛出异常：UnsupportedOperationException。

原因解析：
Arrays.asList(strArray)返回值是java.util.Arrays类中一个私有静态内部类java.util.Arrays.ArrayList，它并非java.util.ArrayList类。java.util.Arrays.ArrayList类具有 set()，get()，contains()等方法，但是不具有添加add()或删除remove()方法,所以调用add()方法会报错。

> 使用场景：Arrays.asList(strArray)方式仅能用在将数组转换为List后，不需要增删其中的值，仅作为数据源读取使用。

### 二.数组转为List后，支持增删改查的方式
通过ArrayList的构造器，将Arrays.asList(strArray)的返回值由java.util.Arrays.ArrayList转为java.util.ArrayList。

关键代码：ArrayList<String> list = new ArrayList<String>(Arrays.asList(strArray)) ;

```
private void testArrayCastToListRight() {
		String[] strArray = new String[2];
		ArrayList<String> list = new ArrayList<String>(Arrays.asList(strArray)) ;
		list.add("1");
		System.out.println(list);
	}
```


执行结果：成功追加一个元素“1”。

> 使用场景：需要在将数组转换为List后，对List进行增删改查操作，在List的数据量不大的情况下，可以使用。

### 三.通过集合工具类Collections.addAll()方法(最高效)
通过Collections.addAll(arrayList, strArray)方式转换，根据数组的长度创建一个长度相同的List，然后通过Collections.addAll()方法，将数组中的元素转为二进制，然后添加到List中，这是最高效的方法。

关键代码：

```
ArrayList< String> arrayList = new ArrayList<String>(strArray.length);
Collections.addAll(arrayList, strArray);
```

测试：

```
private void testArrayCastToListEfficient(){
		String[] strArray = new String[2];
		ArrayList< String> arrayList = new ArrayList<String>(strArray.length);
		Collections.addAll(arrayList, strArray);
		arrayList.add("1");
		System.out.println(arrayList);
	}
```

执行结果：同样成功追加一个元素“1”。

[null, null, 1]

> 使用场景：需要在将数组转换为List后，对List进行增删改查操作，在List的数据量巨大的情况下，优先使用，可以提高操作速度。

### 四.Java8可通过stream流将3种基本类型数组转为List
如果JDK版本在1.8以上，可以使用流stream来将下列3种数组快速转为List，分别是int[]、long[]、double[]，其他数据类型比如short[]、byte[]、char[]，在JDK1.8中暂不支持。由于这只是一种常用方法的封装，不再纳入一种崭新的数组转List方式，暂时算是java流送给我们的常用工具方法吧。

转换代码示例如下：

```
List<Integer> intList= Arrays.stream(new int[] { 1, 2, 3, }).boxed().collect(Collectors.toList());
List<Long> longList= Arrays.stream(new long[] { 1, 2, 3 }).boxed().collect(Collectors.toList());
List<Double> doubleList= Arrays.stream(new double[] { 1, 2, 3 }).boxed().collect(Collectors.toList());
```

如果是String数组，可以使用Stream流这样转换：

```
String[] arrays = {"tom", "jack", "kate"};
List<String> stringList= Stream.of(arrays).collect(Collectors.toList());
```



### 题外话

下列代码为什么会报错？

```
 String[] strArray = new String[2];        
 Arrays.stream(strArray).boxed().collect(Collectors.toList());
```

这段代码会报错是因为在将String数组转换为Stream后，使用boxed()方法将基本数据类型的Stream转换为包装类型的Stream。然而，String本身并不是基本数据类型，因此不需要使用boxed()方法。应该直接使用Arrays.stream(strArray).collect(Collectors.toList())即可将String数组转换为List。
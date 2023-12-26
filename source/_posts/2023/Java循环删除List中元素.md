---
title: Java循环删除List中元素的正确方式
author: ehzyil
tags:
  - Java
categories:
  - 技术
date: 2023-12-26
---

## Java循环删除List中元素的正确方式



There are a few different ways to correctly delete elements from a List in Java. Here are some common methods:

1. Using an Iterator: You can use an Iterator to loop through the List and remove elements based on a condition. This is the safest way to modify a List while iterating over it.

   ```
   List<String> list = new ArrayList<>();
   list.add("A");
   list.add("B");
   list.add("C");
   
   Iterator<String> iterator = list.iterator();
   while (iterator.hasNext()) {
       String element = iterator.next();//删除之前必须显示调用
       if (element.equals("B")) {
           iterator.remove();
       }
   }
   
   System.out.println(list); // Output: [A, C]
   ```

2. Using the removeIf() method: Java 8 introduced the `removeIf()` method in the List interface, which allows you to remove elements based on a specified condition using a Predicate.

   ```
   List<String> list = new ArrayList<>();
   list.add("A");
   list.add("B");
   list.add("C");
   
   list.removeIf(element -> element.equals("B"));
   
   System.out.println(list); // Output: [A, C]
   ```

3. Using the removeAll() method: You can also use the `removeAll()` method to remove all occurrences of a specific element from the List.

   ```
   List<String> list = new ArrayList<>();
   list.add("A");
   list.add("B");
   list.add("C");
   
   list.removeAll(Collections.singleton("B"));
   
   System.out.println(list); // Output: [A, C]
   ```

4. Using a for loop in reverse order:
   You can iterate through the List in reverse order and remove elements that meet a certain condition. This approach is useful when you want to avoid using an Iterator.

   ```java
   List<String> list = new ArrayList<>();
   list.add("A");
   list.add("B");
   list.add("C");
   
   for (int i = list.size() - 1; i >= 0; i--) {
       if (list.get(i).equals("B")) {
           list.remove(i);
       }
   }
   
   System.out.println(list); // Output: [A, C]
   ```

5. Using the subList() method:
   You can utilize the `subList()` method to create a sub-list and then remove elements from it. This approach is helpful when you want to remove a range of elements from the List.

   ```java
   List<String> list = new ArrayList<>();
   list.add("A");
   list.add("B");
   list.add("C");
   
   list.subList(1, 2).clear();
   
   System.out.println(list); // Output: [A, C]
   ```

6. Using the Apache Commons Collections library:
   If you are open to using external libraries, the Apache Commons Collections library provides a `CollectionUtils` class with various methods for manipulating collections, including removing elements based on conditions.

   ```java
   List<String> list = new ArrayList<>();
   list.add("A");
   list.add("B");
   list.add("C");
   
   CollectionUtils.filter(list, object -> !object.equals("B"));
   
   System.out.println(list); // Output: [A, C]
   ```

### 题外话

其他方法

1. 通过for循环删除（错误）
2. 使用foreach循环删除（错误）

原因见:	[Java循环删除List中元素的正确方式](http://t.csdnimg.cn/NHbGV)
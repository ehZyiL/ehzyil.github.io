---
title: Java将list转字符串的方法
author: ehzyil
tags:
  - Java
categories:
  - 技术
date: 2023-12-26
---

### 第一种：join()

```
String.join(",", strList)
```

### 第二种：循环插入逗号

```
public static <T> String parseListToStr(List<T> list){
    StringBuffer sb = new StringBuffer();
    if(listIsNotNull(list)) {
        for(int i=0;i<=list.size()-1;i++){
            if(i<list.size()-1){
                sb.append(list.get(i) + ",");
            }else {
                sb.append(list.get(i));
            }
        }
    }
    return sb.toString();
}
```

### 第三种：stream流

```
public static <T> String parseListToStr3(List<T> list){
    String result = list.stream().map(String::valueOf).collect(Collectors.joining(","));
```

## 第四种：使用谷歌Joiner方法

```
import com.google.common.base.Joiner;

public static <T> String parseListToStr(List<T> list){
    String result = Joiner.on(",").join(list);
    return result;
}
```

导入com.google.common.base.Joiner;

```
public static String strategy（String strategy）{
    String result = Joiner.on（“，”）.join（list）;
    返回结果;
}
```






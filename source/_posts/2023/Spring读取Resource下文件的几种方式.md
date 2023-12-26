---
title: Spring读取Resource下文件的几种方式
author: ehzyil
tags:
  - Java
  - Spring
categories:
  - 技术
date: 2023-12-26
---



## Spring读取Resource下文件的几种方式



### 第一种：

通过当前线程的类加载器获取资源文件；

```
InputStream inputStream = Thread.currentThread().getContextClassLoader().getResourceAsStream("excleTemplate/test.xlsx");
```



### 第二种：

通过当前类的类加载器获取资源文件

```
InputStream inputStream = this.getClass().getResourceAsStream("/excleTemplate/test.xlsx");
```



> 前两种方式都是相对路径，需要注意资源文件的位置和调用代码的位置。

### 第三种：**ClassPathResource**

这是使用Spring框架提供的方式，通过`ClassPathResource`来读取资源文件。它提供了更多的灵活性和功能，比如可以直接获取输入流。

```
ClassPathResource classPathResource = new ClassPathResource("excleTemplate/test.xlsx");
InputStream inputStream =classPathResource.getInputStream();
```



### 第四种：ResourceUtils

这种方式使用了`ResourceUtils`的`getFile`方法，但是需要注意的是`ResourceUtils`的`getFile`方法在生产环境中可能会出现问题。因为它使用了`classpath:`前缀，而这种方式在生产环境中并不适用。

```
File file = ResourceUtils.getFile("classpath:excleTemplate/test.xlsx");
InputStream inputStream = new FileInputStream(file);
```



###  应用实例：spring读取json文件

```
public static com.alibaba.fastjson.JSONObject getProblemType() throws IOException {
    // 使用 ResourceUtils 工具类获取类路径下的 problemType.json 文件
    File file = ResourceUtils.getFile("classpath:data/problemType.json");
    // 使用 FileUtils 工具类将文件内容读取为字符串
    String json = FileUtils.readFileToString(file, "UTF-8");
    // 使用 fastjson 库将字符串解析为 JSONObject 对象
    com.alibaba.fastjson.JSONObject jsonObject = JSON.parseObject(json);
    // 返回解析后的 JSONObject 对象
    return jsonObject;
}
```

这段代码的作用是从类路径中读取一个名为 `problemType.json` 的文件，并将其内容解析为 `JSONObject` 对象，然后返回该对象。以下是注释的详细解释：

1. `public static com.alibaba.fastjson.JSONObject getProblemType() throws IOException {`: 这是一个公共的静态方法，返回类型为 `com.alibaba.fastjson.JSONObject`，并且可能会抛出 `IOException` 异常。
2. `File file = ResourceUtils.getFile("classpath:data/problemType.json");`: 使用 Spring 框架的 `ResourceUtils` 工具类获取类路径下的 `problemType.json` 文件。`classpath:` 前缀表示文件位于类路径下的 `data` 目录中。
3. `String json = FileUtils.readFileToString(file, "UTF-8");`: 使用 Apache Commons IO 工具类 `FileUtils` 将文件内容读取为字符串。这里指定了文件编码为 UTF-8。
4. `com.alibaba.fastjson.JSONObject jsonObject = JSON.parseObject(json);`: 使用阿里巴巴的 `fastjson` 库将字符串解析为 `JSONObject` 对象。
5. `return jsonObject;`: 返回解析后的 `JSONObject` 对象。



替换成更保险的方法
```
public static com.alibaba.fastjson.JSONObject getProblemType() throws IOException {
    // 使用 ClassPathResource 工具类获取类路径下的 problemType.json 文件
    ClassPathResource resource = new ClassPathResource("data/problemType.json");
    // 使用 IOUtils 工具类将文件内容读取为字符串
    String json = IOUtils.toString(resource.getInputStream(), "UTF-8");
    // 使用 fastjson 库将字符串解析为 JSONObject 对象
    com.alibaba.fastjson.JSONObject jsonObject = JSON.parseObject(json);
    // 返回解析后的 JSONObject 对象
    return jsonObject;
}
```


---
title: 'IntelliJ IDEA创建JSP项目'
author: ehzyil

tags:
  - 'IntelliJ IDEA'
  - 'JSP'
categories:
  - 技术
headimg:  ../../../images/IntelliJ IDEA创建JSP项目/创建JSP项目.jpg

date: 2023-10-12

---

## 创建项目

在IntelliJ IDEA中创建jsp项目的步骤如下:

1.点击”Create New Project”

2.选择“Java Enterprise”并将配置设置为箭头所指内容后点击”Next”

输入项目名称并选择存储位置，选择Web Application，选择要使用的Web框架（若为空，自行去下载Tomcat并配置），选择构建系统，选择JDK版本

<img src="../../../images/IntelliJ IDEA创建JSP项目/2023-10-12111.png" />

3.若不选择其他依赖项默认只有“Servlet”依赖，点击Create创建项目

<img src="../../../images/IntelliJ IDEA创建JSP项目/2023-10-12dsfsdfsdf.png" />

4.创建后的JSP项目的结构如下：

```java
├─.idea
│  ├─artifacts
│  └─libraries
├─.mvn
│  └─wrapper
├─src
│  ├─main
│  │  ├─java
│  │  │  └─com
│  │  │      └─example
│  │  │          └─practicaltraining
│  │  ├─resources
│  │  └─webapp
│  │      └─WEB-INF
│  └─test
│      ├─java
│      └─resources
└─target
    ├─classes
    │  └─com
    │      └─example
    │          └─practicaltraining
    ├─generated-sources
    │  └─annotations
    └─practicalTraining-1.0-SNAPSHOT
        ├─META-INF
        └─WEB-INF
            └─classes
                └─com
                    └─example
                        └─practicaltraining

```

5.点击"Run"即可运行,项目默认创建的有一个"/helloServlet"。项目运行后点击即可响应"Hello Servlet"。

## 更改默认跳转页面



1.进入`src->main->webapp->WEB-INF`下的web.xml文件。

2.更改添加`<welcome-file-list>`和`  <welcome-file>`标签并用`  <welcome-file></welcome-file>`包裹启动后要跳转的页面名，该项目默认跳转的页面为`login.jsp`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <welcome-file-list>
        <welcome-file>login.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```


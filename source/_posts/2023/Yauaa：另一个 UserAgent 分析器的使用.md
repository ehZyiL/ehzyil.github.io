---
title: 'Yauaa：另一个 UserAgent 分析器'
author: ehzyil
tags:
  - Yauaa
categories:
  - 工具
keywords:
    - 解析请求信息
date: 2023-10-09 
---

## Yauaa自述
[Yauaa: Yet Another UserAgent Analyzer](https://github.com/nielsbasjes/yauaa/blob/main/README.md#yauaa-yet-another-useragent-analyzer)

这是一个 java 库，尝试解析和分析用户代理字符串（以及用户代理客户端提示可用时）并提取尽可能多的相关属性。
可与 Java、Scala、Kotlin 配合使用，并为多种处理系统提供现成的 UDF。
完整的文档可以在这里找到[https://yauaa.basjes.nl](https://yauaa.basjes.nl/)

## 使用Yauaa

第一步 添加Yauaa依赖

如果使用基于 Maven 的项目，只需将此依赖项添加到Maven 的项目中即可。

```
<dependency>
    <groupId>nl.basjes.parse.useragent</groupId>
    <artifactId>yauaa</artifactId>
    <version>7.22.0</version>
</dependency>
```

第二步 创建**创建一个工具类**  `UserAgentAnalyzer`.

```java
public class UserAgentUtils {

    private static final UserAgentAnalyzer USER_AGENT_ANALYZER;

    static {
        USER_AGENT_ANALYZER = UserAgentAnalyzer
                .newBuilder()
                .hideMatcherLoadStats()
                .withField(UserAgent.OPERATING_SYSTEM_NAME_VERSION_MAJOR)
                .withField(UserAgent.AGENT_NAME_VERSION)
                .build();
    }

    /**
     * 从User-Agent解析客户端操作系统和浏览器版本
     */
    public static Map<String, String> parseOsAndBrowser(String userAgent) {
        UserAgent agent = USER_AGENT_ANALYZER.parse(userAgent);
        String os = agent.getValue(UserAgent.OPERATING_SYSTEM_NAME_VERSION_MAJOR);
        String browser = agent.getValue(UserAgent.AGENT_NAME_VERSION);
        Map<String, String> map = new HashMap<>(2);
        map.put("os", os);
        map.put("browser", browser);
        return map;
    }

}
```

第三步使用

对于每个UserAgent（或一组请求标头），您可以调用该`parse`方法来获得所需的结果：

```java
UserAgent agent = uaa.parse("Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.11) Gecko/20071127 Firefox/2.0.0.11");

for (String fieldName: agent.getAvailableFieldNamesSorted()) {
    System.out.println(fieldName + " = " + agent.getValue(fieldName));
}
```

请注意，并非所有字段在每次解析后都可用。因此，请准备好接收`null`或`Unknown`其他字段特定的默认值

```java
 String userAgent = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.77 Safari/537.36";
    Map<String, String> userAgentMap = userAgentUtils.parseOsAndBrowser(userAgent);
    System.err.println(userAgentMap);
    String os = userAgentMap.get("os");
    String browser = userAgentMap.get("browser");
    System.err.println(os);
    System.err.println(browser);
```

在工具类中 只获取了操作系统和浏览器的名称版本(如有其他需要可自行添加，详见`package nl.basjes.parse.useragent;`) 


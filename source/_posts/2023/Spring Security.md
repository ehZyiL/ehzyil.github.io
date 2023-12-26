---
title: 'Spring Security'
author: ehzyil
tags:
  - Spring Security
categories:
  - 技术
date: 2023-09-16
---

# Spring Security

## 1.认证

##  认证原理

SpringSecurity的原理其实就是一个过滤器链，内部包含了提供各种功能的过滤器。例如快速入门案例里面使用到的三种过滤器，如下图

监听器 -> 过滤器链 -> dispatcherservlet(前置拦截器 -> mapperHandle -> 后置拦截器 -> 最终拦截器)

![output](../../../images/Spring%20Security/output.png)



下面介绍过滤器链中主要的几个过滤器及其作用：

**SecurityContextPersistenceFilter** 这个Filter是整个拦截过程的入口和出口（也就是第一个和最后一个拦截器），会在请求开始时从配置好的 SecurityContextRepository 中获取 SecurityContext，然后把它设置给 SecurityContextHolder。在请求完成后将 SecurityContextHolder 持有的 SecurityContext 再保存到配置好的 SecurityContextRepository，同时清除 securityContextHolder 所持有的 SecurityContext；

**UsernamePasswordAuthenticationFilter** 用于处理来自表单提交的认证。该表单必须提供对应的用户名和密码，其内部还有登录成功或失败后进行处理的 AuthenticationSuccessHandler 和 AuthenticationFailureHandler，这些都可以根据需求做相关改变；

**ExceptionTranslationFilter** 能够捕获来自 FilterChain 所有的异常，并进行处理。但是它只会处理两类异常：AuthenticationException 和 AccessDeniedException，其它的异常它会继续抛出。

**FilterSecurityInterceptor** 是用于保护web资源的，使用AccessDecisionManager对当前用户进行授权访问，前面已经详细介绍过了；

## 认证流程

Spring Security的执行流程如下：



![a44ba03f-c257-4dd2-b32a-524dd19cabad](../../../images/Spring%20Security/a44ba03f-c257-4dd2-b32a-524dd19cabad.png)



1. 用户提交用户名、密码被SecurityFilterChain中的UsernamePasswordAuthenticationFilter过滤器获取到，封装为请求Authentication，通常情况下是UsernamePasswordAuthenticationToken这个实现类。
2. 然后过滤器将Authentication提交至认证管理器（AuthenticationManager）进行认证
3. 认证成功后，AuthenticationManager身份管理器返回一个被填充满了信息的（包括上面提到的权限信息，身份信息，细节信息，但密码通常会被移除）Authentication实例。
4. SecurityContextHolderAuthentication，安全上下文容器将第3步填充了信息的通 过SecurityContextHolder.getContext().setAuthentication(…)方法，设置到其中。
5. 可以看出AuthenticationManager接口（认证管理器）是认证相关的核心接口，也是发起认证的出发点，它的实现类为ProviderManager。而Spring Security支持多种认证方式，因此ProviderManager维护着一个List<AuthenticationProvider>列表，存放多种认证方式，最终实际的认证工作是由AuthenticationProvider完成的。咱们知道web表单的对应的AuthenticationProvider实现类为DaoAuthenticationProvider，它的内部又维护着一个UserDetailsService负责UserDetails的获取。最终AuthenticationProvider将UserDetails填充至Authentication。

## 实践

### 自定义security的思路

【登录】

①、自定义登录接口。用于调用ProviderManager的方法进行认证 如果认证通过生成jwt，然后把用户信息存入redis中

②、自定义UserDetailsService接口的实现类。在这个实现类中去查询数据库

【校验】

①、定义Jwt认证过滤器。用于获取token，然后解析token获取其中的userid，还需要从redis中获取用户信息，然后存入SecurityContextHolder

###  自定义security的搭建

第一步:新建Maven项目引入以下依赖

```xml
	<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.0</version>
    </parent>

    <dependencies>
        <!--还要引入这个，不然后面javax.servlet依赖找不到-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--springboot整合springsecurity-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <!--redis依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!--fastjson依赖-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.33</version>
        </dependency>

        <!--jwt依赖-->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.0</version>
        </dependency>
        
        <!--引入Lombok依赖，方便实体类开发-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
```

第一步：创建以下类

启动类

```java
@SpringBootApplication
    @MapperScan("com.ehzyil.mapper")
public class SecurityApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(SecurityApplication.class);
        System.out.println(run);
    }
}
```
测试接口

```java
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String hello(){
        return "欢迎，开始你新的学习旅程吧";
    }
}
```

 utils.FastJsonRedisSerializer

```java
package com.ehzyil.utils;


import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.parser.ParserConfig;
import com.alibaba.fastjson.serializer.SerializerFeature;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.type.TypeFactory;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.SerializationException;

import java.nio.charset.Charset;

/**
 * Redis使用FastJson序列化
 *
 * @param <T>
 */
public class FastJsonRedisSerializer<T> implements RedisSerializer<T> {

    public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");

    static {
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
    }

    private Class<T> clazz;

    public FastJsonRedisSerializer(Class<T> clazz) {
        super();
        this.clazz = clazz;
    }

    @Override
    public byte[] serialize(T t) throws SerializationException {
        if (t == null) {
            return new byte[0];
        }
        return JSON.toJSONString(t, SerializerFeature.WriteClassName).getBytes(DEFAULT_CHARSET);
    }

    @Override
    public T deserialize(byte[] bytes) throws SerializationException {
        if (bytes == null || bytes.length <= 0) {
            return null;
        }
        String str = new String(bytes, DEFAULT_CHARSET);

        return JSON.parseObject(str, clazz);
    }

    protected JavaType getJavaType(Class<?> clazz) {
        return TypeFactory.defaultInstance().constructType(clazz);
    }
}
```

JwtUtil

```
package com.ehzyil.utils;


import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtBuilder;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;

import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;
import java.util.Date;
import java.util.UUID;


//JWT工具类
public class JwtUtil {

    //有效期为
    public static final Long JWT_TTL = 60 * 60 * 1000L;// 60 * 60 *1000  一个小时
    //设置秘钥明文, 注意长度必须大于等于6位
    public static final String JWT_KEY = "huanfqc";

    public static String getUUID() {
        String token = UUID.randomUUID().toString().replaceAll("-", "");
        return token;
    }

    /**
     * 生成jtw
     *
     * @param subject token中要存放的数据（json格式）
     * @return
     */
    public static String createJWT(String subject) {
        JwtBuilder builder = getJwtBuilder(subject, null, getUUID());// 设置过期时间
        return builder.compact();
    }

    /**
     * 生成jtw
     *
     * @param subject   token中要存放的数据（json格式）
     * @param ttlMillis token超时时间
     * @return
     */
    public static String createJWT(String subject, Long ttlMillis) {
        JwtBuilder builder = getJwtBuilder(subject, ttlMillis, getUUID());// 设置过期时间
        return builder.compact();
    }

    private static JwtBuilder getJwtBuilder(String subject, Long ttlMillis, String uuid) {
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
        SecretKey secretKey = generalKey();
        long nowMillis = System.currentTimeMillis();
        Date now = new Date(nowMillis);
        if (ttlMillis == null) {
            ttlMillis = JwtUtil.JWT_TTL;
        }
        long expMillis = nowMillis + ttlMillis;
        Date expDate = new Date(expMillis);
        return Jwts.builder()
                .setId(uuid)              //唯一的ID
                .setSubject(subject)   // 主题  可以是JSON数据
                .setIssuer("huanf")     // 签发者
                .setIssuedAt(now)      // 签发时间
                .signWith(signatureAlgorithm, secretKey) //使用HS256对称加密算法签名, 第二个参数为秘钥
                .setExpiration(expDate);
    }

    /**
     * 创建token
     *
     * @param id
     * @param subject
     * @param ttlMillis
     * @return
     */
    public static String createJWT(String id, String subject, Long ttlMillis) {
        JwtBuilder builder = getJwtBuilder(subject, ttlMillis, id);// 设置过期时间
        return builder.compact();
    }

    public static void main(String[] args) throws Exception {

        //加密指定字符串，jwt是123456加密后的密文
        String jwt = createJWT("123456");
        System.out.println(jwt);

        String token = "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI0Mzg2MTUxZTFkNTE0ZmUwYjRlOWUzZDZiYjFlODA4OSIsInN1YiI6IjEyMzQ1NiIsImlzcyI6Imh1YW5mIiwiaWF0IjoxNjk0NzgzODI3LCJleHAiOjE2OTQ3ODc0Mjd9.z4QnAiSNBQcCAhZPwC5Xfzb0Py4np7qnyrUg0Ih8Qr4";
        Claims claims = parseJWT(token);
        System.out.println(claims);
    }

    /**
     * 生成加密后的秘钥 secretKey
     *
     * @return
     */
    public static SecretKey generalKey() {
        byte[] encodedKey = Base64.getDecoder().decode(JwtUtil.JWT_KEY);
        SecretKey key = new SecretKeySpec(encodedKey, 0, encodedKey.length, "AES");
        return key;
    }

    /**
     * 解析
     *
     * @param jwt
     * @return
     * @throws Exception
     */
    public static Claims parseJWT(String jwt) throws Exception {
        SecretKey secretKey = generalKey();
        return Jwts.parser()
                .setSigningKey(secretKey)
                .parseClaimsJws(jwt)
                .getBody();
    }

}
```

redis工具类 RedisCache

```java
package com.ehzyil.utils;


import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.BoundSetOperations;
import org.springframework.data.redis.core.HashOperations;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.stereotype.Component;

import java.util.*;
import java.util.concurrent.TimeUnit;

@SuppressWarnings(value = {"unchecked", "rawtypes"})
@Component
//redis工具类
public class RedisCache {
    @Autowired
    public RedisTemplate redisTemplate;

    /**
     * 缓存基本的对象，Integer、String、实体类等
     *
     * @param key   缓存的键值
     * @param value 缓存的值
     */
    public <T> void setCacheObject(final String key, final T value) {
        redisTemplate.opsForValue().set(key, value);
    }

    /**
     * 缓存基本的对象，Integer、String、实体类等
     *
     * @param key      缓存的键值
     * @param value    缓存的值
     * @param timeout  时间
     * @param timeUnit 时间颗粒度
     */
    public <T> void setCacheObject(final String key, final T value, final Integer timeout, final TimeUnit timeUnit) {
        redisTemplate.opsForValue().set(key, value, timeout, timeUnit);
    }

    /**
     * 设置有效时间
     *
     * @param key     Redis键
     * @param timeout 超时时间
     * @return true=设置成功；false=设置失败
     */
    public boolean expire(final String key, final long timeout) {
        return expire(key, timeout, TimeUnit.SECONDS);
    }

    /**
     * 设置有效时间
     *
     * @param key     Redis键
     * @param timeout 超时时间
     * @param unit    时间单位
     * @return true=设置成功；false=设置失败
     */
    public boolean expire(final String key, final long timeout, final TimeUnit unit) {
        return redisTemplate.expire(key, timeout, unit);
    }

    /**
     * 获得缓存的基本对象。
     *
     * @param key 缓存键值
     * @return 缓存键值对应的数据
     */
    public <T> T getCacheObject(final String key) {
        ValueOperations<String, T> operation = redisTemplate.opsForValue();
        return operation.get(key);
    }

    /**
     * 删除单个对象
     *
     * @param key
     */
    public boolean deleteObject(final String key) {
        return redisTemplate.delete(key);
    }

    /**
     * 删除集合对象
     *
     * @param collection 多个对象
     * @return
     */
    public long deleteObject(final Collection collection) {
        return redisTemplate.delete(collection);
    }

    /**
     * 缓存List数据
     *
     * @param key      缓存的键值
     * @param dataList 待缓存的List数据
     * @return 缓存的对象
     */
    public <T> long setCacheList(final String key, final List<T> dataList) {
        Long count = redisTemplate.opsForList().rightPushAll(key, dataList);
        return count == null ? 0 : count;
    }

    /**
     * 获得缓存的list对象
     *
     * @param key 缓存的键值
     * @return 缓存键值对应的数据
     */
    public <T> List<T> getCacheList(final String key) {
        return redisTemplate.opsForList().range(key, 0, -1);
    }

    /**
     * 缓存Set
     *
     * @param key     缓存键值
     * @param dataSet 缓存的数据
     * @return 缓存数据的对象
     */
    public <T> BoundSetOperations<String, T> setCacheSet(final String key, final Set<T> dataSet) {
        BoundSetOperations<String, T> setOperation = redisTemplate.boundSetOps(key);
        Iterator<T> it = dataSet.iterator();
        while (it.hasNext()) {
            setOperation.add(it.next());
        }
        return setOperation;
    }

    /**
     * 获得缓存的set
     *
     * @param key
     * @return
     */
    public <T> Set<T> getCacheSet(final String key) {
        return redisTemplate.opsForSet().members(key);
    }

    /**
     * 缓存Map
     *
     * @param key
     * @param dataMap
     */
    public <T> void setCacheMap(final String key, final Map<String, T> dataMap) {
        if (dataMap != null) {
            redisTemplate.opsForHash().putAll(key, dataMap);
        }
    }

    /**
     * 获得缓存的Map
     *
     * @param key
     * @return
     */
    public <T> Map<String, T> getCacheMap(final String key) {
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * 往Hash中存入数据
     *
     * @param key   Redis键
     * @param hKey  Hash键
     * @param value 值
     */
    public <T> void setCacheMapValue(final String key, final String hKey, final T value) {
        redisTemplate.opsForHash().put(key, hKey, value);
    }

    /**
     * 获取Hash中的数据
     *
     * @param key  Redis键
     * @param hKey Hash键
     * @return Hash中的对象
     */
    public <T> T getCacheMapValue(final String key, final String hKey) {
        HashOperations<String, String, T> opsForHash = redisTemplate.opsForHash();
        return opsForHash.get(key, hKey);
    }

    /**
     * 删除Hash中的数据
     *
     * @param key
     * @param hkey
     */
    public void delCacheMapValue(final String key, final String hkey) {
        HashOperations hashOperations = redisTemplate.opsForHash();
        hashOperations.delete(key, hkey);
    }

    /**
     * 获取多个Hash中的数据
     *
     * @param key   Redis键
     * @param hKeys Hash键集合
     * @return Hash对象集合
     */
    public <T> List<T> getMultiCacheMapValue(final String key, final Collection<Object> hKeys) {
        return redisTemplate.opsForHash().multiGet(key, hKeys);
    }

    /**
     * 获得缓存的基本对象列表
     *
     * @param pattern 字符串前缀
     * @return 对象列表
     */
    public Collection<String> keys(final String pattern) {
        return redisTemplate.keys(pattern);
    }
}
```



 config.RedisConfig 类

```java
package com.ehzyil.config;

import com.ehzyil.utils.FastJsonRedisSerializer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;

/**
 * @author 35238
 * @date 2023/7/11 0011 15:40
 */
@Configuration
public class RedisConfig {

    @Bean
    @SuppressWarnings(value = {"unchecked", "rawtypes"})
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);

        FastJsonRedisSerializer serializer = new FastJsonRedisSerializer(Object.class);

        // 使用StringRedisSerializer来序列化和反序列化redis的key值
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(serializer);

        // Hash的key也采用StringRedisSerializer的序列化方式
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(serializer);

        template.afterPropertiesSet();
        return template;
    }
}
```



domain.ResponseResult 

```java
package com.ehzyil.domain;

import com.fasterxml.jackson.annotation.JsonInclude;

//响应类
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ResponseResult<T> {
    /**
     * 状态码
     */
    private Integer code;
    /**
     * 提示信息，如果有错误时，前端可以获取该字段进行提示
     */
    private String msg;
    /**
     * 查询到的结果数据，
     */
    private T data;

    public ResponseResult(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public ResponseResult(Integer code, T data) {
        this.code = code;
        this.data = data;
    }

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public ResponseResult(Integer code, String msg, T data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }
}

```

在 utile 目录新建 WebUtils 类

```java
package com.ehzyil.utils;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class WebUtils {
    /**
     * 将字符串渲染到客户端
     *
     * @param response 渲染对象
     * @param string 待渲染的字符串
     * @return null
     */
    public static String renderString(HttpServletResponse response, String string) {
        try
        {
            response.setStatus(200);
            response.setContentType("application/json");
            response.setCharacterEncoding("utf-8");
            response.getWriter().print(string);
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }
        return null;
    }
}
```

在 domain目录新建 User类，写入如下

```java
package com.ehzyil.domain;

import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.io.Serializable;
import java.util.Date;


//用户表(User)实体类
@Data
@AllArgsConstructor
@NoArgsConstructor
@TableName("sys_user")
public class User implements Serializable {
    private static final long serialVersionUID = -40356785423868312L;

    /**
     * 主键
     */
    @TableId
    private Long id;
    /**
     * 用户名
     */
    private String userName;
    /**
     * 昵称
     */
    private String nickName;
    /**
     * 密码
     */
    private String password;
    /**
     * 账号状态（0正常 1停用）
     */
    private String status;
    /**
     * 邮箱
     */
    private String email;
    /**
     * 手机号
     */
    private String phonenumber;
    /**
     * 用户性别（0男，1女，2未知）
     */
    private String sex;
    /**
     * 头像
     */
    private String avatar;
    /**
     * 用户类型（0管理员，1普通用户）
     */
    private String userType;
    /**
     * 创建人的用户id
     */
    private Long createBy;
    /**
     * 创建时间
     */
    private Date createTime;
    /**
     * 更新人
     */
    private Long updateBy;
    /**
     * 更新时间
     */
    private Date updateTime;
    /**
     * 删除标志（0代表未删除，1代表已删除）
     */
    private Integer delFlag;
}
```



### 自定义security的数据库

第一步: 数据库校验用户。从之前的分析我们可以知道，我们自定义了一个UserDetailsService，让SpringSecurity使用我们的UserDetailsService。我们自己的UserDetailsService可以从数据库中查询用户名和密码。我们先创建一个用户表， 建表语句如下：

注意: 要想让用户的密码是明文存储，需要在密码前加{noop}，作用是例如等下在浏览器登陆的时候就可以用huanf作为用户名，112233作为密码来登陆了

```sql
create database if not exists huanf_security;
use huanf_security;

CREATE TABLE `sys_user` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `user_name` VARCHAR(64) NOT NULL DEFAULT 'NULL' COMMENT '用户名',
  `nick_name` VARCHAR(64) NOT NULL DEFAULT 'NULL' COMMENT '昵称',
  `password` VARCHAR(64) NOT NULL DEFAULT 'NULL' COMMENT '密码',
  `status` CHAR(1) DEFAULT '0' COMMENT '账号状态（0正常 1停用）',
  `email` VARCHAR(64) DEFAULT NULL COMMENT '邮箱',
  `phonenumber` VARCHAR(32) DEFAULT NULL COMMENT '手机号',
  `sex` CHAR(1) DEFAULT NULL COMMENT '用户性别（0男，1女，2未知）',
  `avatar` VARCHAR(128) DEFAULT NULL COMMENT '头像',
  `user_type` CHAR(1) NOT NULL DEFAULT '1' COMMENT '用户类型（0管理员，1普通用户）',
  `create_by` BIGINT(20) DEFAULT NULL COMMENT '创建人的用户id',
  `create_time` DATETIME DEFAULT NULL COMMENT '创建时间',
  `update_by` BIGINT(20) DEFAULT NULL COMMENT '更新人',
  `update_time` DATETIME DEFAULT NULL COMMENT '更新时间',
  `del_flag` INT(11) DEFAULT '0' COMMENT '删除标志（0代表未删除，1代表已删除）',
  PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COMMENT='用户表';

insert into sys_user values (1,'admin','管理员','{noop}123456','0',DEFAULT,DEFAULT,DEFAULT,DEFAULT,'0',DEFAULT,DEFAULT,DEFAULT,DEFAULT,DEFAULT);
insert into sys_user values (2,'huanf','涣沷a靑惷','{noop}112233','0',DEFAULT,DEFAULT,DEFAULT,DEFAULT,'1',DEFAULT,DEFAULT,DEFAULT,DEFAULT,DEFAULT);
```

第二步: 在pom.xml添加如下

```xml
<!--引入MybatisPuls依赖-->
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus-boot-starter</artifactId>
	<version>3.4.3</version>
</dependency>

<!--引入mysql驱动的依赖-->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
</dependency>
```

第三步: 在 src/main/resources 目录新建File，文件名为application.yml，写入如下

```yaml
server:
  port: 8089

spring:
  # 数据库连接信息
  datasource:
    url: jdbc:mysql://localhost:3306/huanf_security?characterEncoding=utf-8&serverTimezone=Asia/Shanghai
    username: root
    password: 666666
    driver-class-name: com.mysql.cj.jdbc.Driver
```

第四步: 在 src/main/java/com.ehzyil 目录新建 mapper.UserMapper 接口，写入如下

```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.ehzyil.domain.User;

@Service
public interface UserMapper extends BaseMapper<User> {
}
```

第五步: 在引导类添加如下.

```java
@MapperScan("com.ehzyil.mapper")
```

第六步: 在pom.xml添加如下

```xml
<!--引入Junit，用于测试-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
</dependency>
```

第七步: 在 src/test/java 目录新建 com.ehzyil.MapperTest类，写入如下。作用是测试mybatis-plus是否正常

```
package com.ehzyil;

import com.ehzyil.domain.User;
import com.ehzyil.mapper.UserMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest
public class MapperTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testUserMapper(){
        //查询所有用户
        List<User> users = userMapper.selectList(null);
        System.out.println(users);
    }

}
```

第八步: 运行MapperTest类的testUserMapper方法，看是否能查到数据库的所有用户。到此，可以确定数据库是没问题的，环境到此就准备好了

### 自定义security的认证实现	



上面我们已经把准备工作做好了，包括搭建、代码、数据库。接下来我们会实现让security在认证的时候，根据我们数据库的用户和密码进行认证，也就是被security拦截业务接口，出现登录页面之后，我们需要通过输入数据库里的用户和密码来登录，而不是使用security默认的用户和密码进行登录

用户提交账号和密码由DaoAuthenticationProvider调用UserDetailsService的loadUserByUsername()方法获取UserDetails用户信息。

查询DaoAuthenticationProvider的源代码如下：

![img](../../../images/Spring%20Security/1694832598400-18.png)

UserDetailsService是一个接口，如下：

```java
package org.springframework.security.core.userdetails;

public interface UserDetailsService {
    UserDetails loadUserByUsername(String var1) throws UsernameNotFoundException;
}
```

UserDetails是用户信息接口

```java
public interface UserDetails extends Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();

    String getPassword();

    String getUsername();

    boolean isAccountNonExpired();

    boolean isAccountNonLocked();

    boolean isCredentialsNonExpired();

    boolean isEnabled();
}
```



我们只要实现UserDetailsService 接口查询数据库得到用户信息返回UserDetails 类型的用户信息即可,框架调用loadUserByUsername()方法拿到用户信息之后是如何执行的，见下图：

![img](../../../images/Spring%20Security/1694832598400-17.png)

思路: 只需要新建一个实现类，在这个实现类里面实现Security官方的UserDetailsService接口，然后重写里面的loadUserByUsername方法

注意: 重写好loadUserByUsername方法之后，我们需要把拿到 '数据库与用户输入的数据' 进行比对的结果，也就是user对象这个结果封装成能被 'Security官方的UserDetailsService接口' 接收的类型，例如可以封装成我们下面写的LoginUser类型。然后才能伪装好数据，给Security官方的认证机制去对比user对象与数据库的结果是否匹配。Security官方的认证机制会拿LoginUser类的方法数据(数据库拿，不再用默认的)，跟我们封装过去的user对象进行匹配，要使匹配一致，就证明认证通过，也就是用户在浏览器页面输入的用户名和密码能被Security认证通过，就不再拦截该用户去访问我们的业务接口

第一步: 在domain目录新建LoginUser类，作为UserDetails接口(Security官方提供的接口)的实现类

```
package com.ehzyil.domain;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.Collection;


@Data //get和set方法
@NoArgsConstructor //无参构造
@AllArgsConstructor  //带参构造
//实现UserDetails接口之后，要重写UserDetails接口里面的7个方法
public class LoginUser implements UserDetails {
    
    private User user;
    
    @Override
    //用于返回权限信息。现在我们正在学'认证'，'权限'后面才学。所以返回null即可
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return null;
    }

    @Override
    //用于获取用户密码。由于使用的实体类是User，所以获取的是数据库的用户密码
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    //用于获取用户名。由于使用的实体类是User，所以获取的是数据库的用户名
    public String getUsername() {
        return user.getUserName();
    }

    @Override
    //判断登录状态是否过期。把这个改成true，表示永不过期
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    //判断账号是否被锁定。把这个改成true，表示未锁定，不然登录的时候，不让你登录
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    //判断登录凭证是否过期。把这个改成true，表示永不过期
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    //判断用户是否可用。把这个改成true，表示可用状态
    public boolean isEnabled() {
        return true;
    }
}
```

第二步: 在 src/main/java/com.ehzyil 目录新建 service.impl.MyUserDetailServiceImpl 类,实现UserDetailsService接口，写入如下

```java
package com.ehzyil.service.impl;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.ehzyil.domain.LoginUser;
import com.ehzyil.domain.User;
import com.ehzyil.mapper.UserMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.Objects;

@Service
public class MyUserDetailServiceImpl implements UserDetailsService {

    @Autowired
    private UserMapper userMapper;

    @Override
    //UserDetails是Security官方提供的接口
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        //查询用户信息。我们写的userMapper接口里面是空的，所以调用的是mybatis-plus提供的方法
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(User::getUserName,username);
        User user = userMapper.selectOne(queryWrapper);
        //如果用户传进来的用户名，但是数据库没有这个用户名，就会导致我们是查不到的情况，那么就进行下面的判断。避免程序安全问题
        if(Objects.isNull(user)){
            throw new RuntimeException("用户名或者密码错误");
        }

        //把查询到的user结果，封装成UserDetails类型，然后返回。
        //但是由于UserDetails是个接口，所以我们先需要在domino目录新建LoginUser类，作为UserDetails的实现类，再写下面那行
        return new LoginUser(user);
    }
}
```

第三步: 测试。运行引导类，浏览器输入如下，然后我们输入一下登录的用户名和密码，看是不是根据数据库来进行认证

```
http://localhost:8089/hello
```

出现以下页面

![image-20230916122841977](../../../images/Spring%20Security/image-20230916122841977.png)

输入 admin 123456登录 即可访问 http://localhost:8089/hello

### 密码加密校验问题

上面我们实现了自定义Security的认证机制，让Security根据数据库的数据，来认证用户输入的数据是否正确。但是当时存在一个问题，就是我们在数据库存入用户表的时候，插入的huanf用户的密码是 {noop}112233，为什么用112233不行呢

原因: SpringSecurity默认使用的PasswordEncoder要求数据库中的密码格式为：{加密方式}密码。对应的就是{noop}112233，实际表示的是112233

但是我们在数据库直接暴露112233为密码，会造成安全问题，所以我们需要把加密后的1234的密文当作密码，此时用户在浏览器登录时输入1234，我们如何确保用户能够登录进去呢，答案是SpringSecurity默认的密码校验，替换为SpringSecurity为我们提供的BCryptPasswordEncoder

我们只需要使用把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验。我们可以定义一个SpringSecurity的配置类，SpringSecurity要求这个配置类要继承WebSecurityConfigurerAdapter。

【首先是 '加密'，如何实现，如下】

第一步: 在config目录新建 SecurityConfig 类，写入如下。作用是根据原文，生成一个密文

```java
package com.ehzyil.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;


@Configuration
//实现Security提供的WebSecurityConfigurerAdapter类，就可以改变密码校验的规则了
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    //把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验
    //注意也可以注入PasswordEncoder，效果是一样的，因为PasswordEncoder是BCry..的父类
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

}
```



第二步: 测试。在MapperTest类，添加如下，然后运行 TestBCryptPasswordEncoder 方法

```java
@Test
public void TestBCryptPasswordEncoder(){
    
	//如果不想在下面那行new的话，那么就在该类注入PasswordEncoder，例如如下
    /**
     * @Autowired
     * private PasswordEncoder passwordEncoder;
     */
    BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        
	//模拟用户输入的密码
	String encode1 = passwordEncoder.encode("1234");
	//再模拟一次用户输入的密码
	String encode2 = passwordEncoder.encode("1234");
	//虽然这两次的密码都是一样的，但是加密后是不一样的。每次运行，对同一原文都会有不同的加密结果
	//原因:会添加随机的盐，加密结果=盐+原文+加密。达到每次加密后的密文都不相同的效果
	System.out.println(encode1);
	System.out.println(encode2);
}
```

【然后是 '校验'，如何实现，如下】

第一步(已做可跳过): 在config目录新建 SecurityConfig 类，写入如下。作用是根据原文，生成一个密文

```java
package com.ehzyil.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;


@Configuration
//实现Security提供的WebSecurityConfigurerAdapter类，就可以改变密码校验的规则了
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    //把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验
    //注意也可以注入PasswordEncoder，效果是一样的，因为PasswordEncoder是BCry..的父类
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

}
```

第二步: 测试。在MapperTest类，添加如下，然后运行 TestBCryptPasswordEncoder 方法

```java
@Test
public void TestBCryptPasswordEncoder(){
	BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();

	//模拟用户输入了1234(第一个参数)，然后我们去跟数据库的密文进行比较(第二个参数)
	boolean result = passwordEncoder.matches("1234", "$2a$10$zOitKu6UNk.b/iPFTtIj2u80sH/dfJI9vFr57qhDGteuXj/Wl8uSy");

	//看一下比对结果
	System.out.println(result);

}
```

重启测试后发现

![image-20230916150258263](../../../images/Spring%20Security/image-20230916150258263.png)

账号1登不进去，密码加密过的账号2可以登录。

### jwt工具类实现加密校验

【加密】

第一步: 使用createJWT方法生成指定字符串的密文。

第一步: 使用parseJWT方法将密文解密(校验)为原文。

```java
    public static void main(String[] args) throws Exception {
        System.out.println("***************加密****************");
        //加密指定字符串，token是123456加密后的密文
        String token = createJWT("123456");
        System.out.println(token);
        System.out.println("***************解密****************");
        //把上面那行的密文解密(校验)为原文
        Claims claims = parseJWT(token);
        System.out.println(claims);
        //输出解密后的原文
        System.out.println(claims.getSubject());
    }
```

第二步: 运行JwtUtil类的main方法

```
***************加密****************
eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiJiYzg0ZWY1NjY1MmQ0YjgzODg2MGUxOTM2MGNlY2FiIsInN1YiI6IjEyMzQ1NiIsImlzcyI6Imh1YW5mIiwiaWF0IjoxNjk0ODMzMjU3LCJleHAiOjE2OTQ4MzY4NTd9.bJDcdufq0LMhpdwlEbE5mBkZoGYZRxHIFoo1rb5MN1U
***************解密****************
{jti=bc84ef56652d4b838860e19360cecabc, sub=123456, iss=huanf, iat=1694833257, exp=1694836857}
123456

```

### 登录接口的分析

在上面的**自定义security的思路**当中，我们有一个功能需求是自定义登录接口，这个功能还没有实现，我们需要实现这个功能，但是，实现这个功能需要使用到jwt，我们刚刚也学习了使用jwt来实现加密校验，那么下面就正式学习如何实现这个登录接口，首先是分析，如下

①我们需要自定义登陆接口，也就是在controller目录新建LoginController类，在controller方法里面去调用service接口，在service接口实现AuthenticationManager去进行用户的认证，注意，我们定义的controller方法要让SpringSecurity对这个接口放行(如果不放行的话，会被SpringSecurity拦截)，让用户访问这个接口的时候不用登录也能访问。

②在service接口中我们通过AuthenticationManager的authenticate方法来进行用户认证,所以需要在SecurityConfig中配置把AuthenticationManager注入容器

③认证成功的话要生成一个jwt，放入响应中返回。并且为了让用户下回请求时能通过jwt识别出具体的是哪个用户，我们需要把用户信息存入redis，可以把用户id作为key。

### 登录接口的实现

第一步: 修改数据库的huanf用户的密码，把112233明文修改为对应的密文。密文可以用jwt工具类加密112233看一下，或者直接复制我给出的

```
UPDATE sys_user SET password = '$2a$10$YPnG.IYUk0mMechaxSibBuKmNeTzvuHdcxkqvoxizsll6WCQG9CHG' WHERE id = 2;
```

​	第二步: 在 SecurityConfig 类添加如下

```java
@Bean
@Override
public AuthenticationManager authenticationManagerBean() throws Exception {
	return super.authenticationManagerBean();
}

@Override
protected void configure(HttpSecurity http) throws Exception {
	http
		//由于是前后端分离项目，所以要关闭csrf
		.csrf().disable()
		//由于是前后端分离项目，所以session是失效的，我们就不通过Session获取SecurityContext
		.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
		.and()
		//指定让spring security放行登录接口的规则
		.authorizeRequests()
		// 对于登录接口 anonymous表示允许匿名访问
		.antMatchers("/user/login").anonymous()
		// 除上面外的所有请求全部需要鉴权认证
		.anyRequest().authenticated();
}
```

第三步: 在service目录新建 LoginService 接口，写入如下

```java
package com.ehzyil.service;

import com.ehzyil.domain.ResponseResult;
import com.ehzyil.domain.User;
import org.springframework.stereotype.Service;


@Service
public interface LoginService {
    ResponseResult login(User user);
}
```



第四步: 在service目录新建 impl.LoginServiceImpl 类，写入如下

```java
package com.ehzyil.service.impl;

@Service
//写登录的核心代码
public class LoginServiceImpl implements LoginService {

    @Autowired
    //先在SecurityConfig，使用@Bean注解重写官方的authenticationManagerBean类，然后这里才能注入成功
    private AuthenticationManager authenticationManager;

    @Autowired
    //RedisCache是我们在utils目录写好的类
    private RedisCache redisCache;

    @Override
    //ResponseResult和user是我们在domain目录写好的类
    public ResponseResult login(User user) {

        //用户在登录页面输入的用户名和密码
        UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(user.getUserName(),user.getPassword());

        //获取AuthenticationManager的authenticate方法来进行用户认证
        Authentication authenticate = authenticationManager.authenticate(authenticationToken);

        //判断上面那行的authenticate是否为null，如果是则认证没通过，就抛出异常
        if(Objects.isNull(authenticate)){
            throw new RuntimeException("登录失败");
        }

        //如果认证通过，就使用userid生成一个jwt，然后把jwt存入ResponseResult后返回
        LoginUser loginUser = (LoginUser) authenticate.getPrincipal();
        String userid = loginUser.getuser().getId().toString();
        String jwt = JwtUtil.createJWT(userid);

        //把完整的用户信息存入redis，其中userid作为key，注意存入redis的时候加了前缀 login:
        Map<String, String> map = new HashMap<>();
        map.put("token",jwt);
        redisCache.setCacheObject("login:"+userid,loginUser);
        return new ResponseResult(200,"登录成功",map);
    }
}
```

第五步: 在controller目录新建 LoginController 类，写入如下

```java
package com.ehzyil.controller;

import com.ehzyil.domain.ResponseResult;
import com.ehzyil.domain.User;
import com.ehzyil.service.LoginService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class LoginController {

    @Autowired
    //LoginService是我们在service目录写好的接口
    private LoginService loginService;

    @PostMapping("/user/login")
    //ResponseResult和user是我们在domain目录写好的类
    public ResponseResult login(@RequestBody User user){
        //登录
        return loginService.login(user);
    }

}
```

第六步: 在application.yml添加如下，作用是添加redis的连接信息

```
redis:
    host: 127.0.0.1
    port: 6379
```

第八步: 运行引导类

第九步: 测试。发送下面的POST请求

```
localhost:8089/user/login

{
    "userName":"huanf",
    "password":"112233"
}
```

```json
{
    "code": 200,
    "msg": "登录成功",
    "data": {
        "token": "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI1YjgzNTU2YjQ5OWY0YjU0ODU5M2Q5OWIjk4OTMwOCIsInN1YiI6IjIiLCJpc3MiOiJodWFuZiIsImlhdCI6MTY5NDg0ODU1OCwiZXhwIjoxNjk0ODUyMTU4fQ.Jca_ufkZFNv0vMvmap3r-AvD0QAjctUzdb5TxYcwicg"
    }
}
```

**注意：第九步的请求一定要使用未加密的密码！！！！！！！！！**（搞了一上午才解决..）

### 认证过滤器

在上面学习的**自定义security的思路**当中，我们有一个功能需求是定义Jwt认证过滤器，这个功能还没有实现，下面就正式学习如何实现这个功能。要实现Jwt认证过滤器，我们需要获取token，然后解析token获取其中的userid，还需要从redis中获取用户信息，然后存入SecurityContextHolder

为什么要有redis参与: 是为了防止过了很久之后，浏览器没有关闭，拿着token也能访问，这样不安全

认证过滤器的作用是什么: 上面我们实现登录接口的时，当某个用户登录之后，该用户就会有一个token值，我们可以通过认证过滤器，由于有token值，并且token值认证通过，也就是证明是这个用户的token值，那么该用户访问我们的业务接口时，就不会被Security拦截。简单理解作用就是登录过的用户可以访问我们的业务接口，拿到对应的资源

第一步: 定义过滤器。在 src/main/java/com.ehzyil 目录新建 filter.JwtAuthenticationTokenFilter 类，写入如下

```java
package com.ehzyil.filter;

import com.ehzyil.domain.LoginUser;
import com.ehzyil.utils.JwtUtil;
import com.ehzyil.utils.RedisCache;
import io.jsonwebtoken.Claims;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.filter.OncePerRequestFilter;
import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Objects;


@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {

    @Autowired
    private RedisCache redisCache;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        //获取token，指定你要获取的请求头叫什么
        String token = request.getHeader("token");
        //判空，不一定所有的请求都有请求头，所以上面那行的token可能为空
        //!StringUtils.hasText()方法用于检查给定的字符串是否为空或仅包含空格字符
        if (!StringUtils.hasText(token)) {
            //如果请求没有携带token，那么就不需要解析token，不需要获取用户信息，直接放行就可以
            filterChain.doFilter(request, response);
            //return之后，就不会走下面那些代码
            return;
        }
        //解析token
        String userid; //把userid定义在外面，才能同时用于下面的46行和52行
        try {
            Claims claims = JwtUtil.parseJWT(token);
            userid = claims.getSubject();
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException("token非法");
        }
        //从redis中获取用户信息
        String redisKey = "login:" + userid;
        //LoginUser是我们在domain目录写的实体类
        LoginUser loginUser = redisCache.getCacheObject(redisKey);
        //判断获取到的用户信息是否为空，因为redis里面可能并不存在这个用户信息，例如缓存过期了
        if(Objects.isNull(loginUser)){
            //抛出一个异常
            throw new RuntimeException("用户未登录");
        }

        //把最终的LoginUser用户信息，通过setAuthentication方法，存入SecurityContextHolder
        //TODO 获取权限信息封装到Authentication中
        UsernamePasswordAuthenticationToken authenticationToken =
                //第一个参数是LoginUser用户信息，第二个参数是凭证(null)，第三个参数是权限信息(null)
                new UsernamePasswordAuthenticationToken(loginUser,null,null);
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);

        //全部做完之后，就放行
        filterChain.doFilter(request, response);
    }
}
```



第二步: 修改SecurityConfig类为如下，其实也就是在configure方法加了一点代码、并且注入了JwtAuthenticationTokenFilter类

```java
@Configuration
//实现Security提供的WebSecurityConfigurerAdapter类，就可以改变密码校验的规则了
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    //把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验
    //注意也可以注入PasswordEncoder，效果是一样的，因为PasswordEncoder是BCry..的父类
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    //---------------------------认证过滤器的实现----------------------------------

    @Autowired
    //注入我们在filter目录写好的类
    private JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;

    //---------------------------登录接口的实现----------------------------------

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //由于是前后端分离项目，所以要关闭csrf
                .csrf().disable()
                //由于是前后端分离项目，所以session是失效的，我们就不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                //指定让spring security放行登录接口的规则
                .authorizeRequests()
                // 对于登录接口 anonymous表示允许匿名访问
                .antMatchers("/user/login").anonymous()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();

        //---------------------------认证过滤器的实现----------------------------------

        //把token校验过滤器添加到过滤器链中
        //第一个参数是上面注入的我们在filter目录写好的类，第二个参数表示你想添加到哪个过滤器之前
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);


    }
    }
```

第三步: 本地打开你的redis

第四步: 运行引导类

第五步: 测试。打开你的postman，发送如下的POST请求，作用是先登录一个用户，这样就能生成这个用户对应的token值

```
{
    "code": 200,
    "msg": "登录成功",
    "data": {
        "token": "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiIzZmU4MTk1YjY3MTc0MmM5YTgwODZkNzE2ZTM4OGNlNyIsInN1YiI6IjIiLCJpc3MiOiJodWFuZiIsImlhdCI6MTY5NDg0ODgzNCwiZXhwIjoxNjk0ODUyNDM0fQ.liyE_t-8Zzh2goiOg0crzQNTqMj1mNAyeg3pSQH0FZo"
    }
}
```



第六步: 测试。继续在你的postman，发送如下GET请求，作用是拿着刚刚的token值，去访问我们的业务接口，看会不会被Security拦截，如果不会拦截，那么就说明认证过滤器生效了，使用场景就是简单理解就是登录过的用户可以访问我们的业务接口，拿到对应的资源

![image-20230916152207531](../../../images/Spring%20Security/image-20230916152207531.png)

注意，由于token值我们是存在redis，所以是有默认过期时间的。注意在请求头那里，key要写token，value要写你复制的token值，然后点击发送请求。这个token值实际上就是使用jwt工具类把112233密码加密后的密文，不信你翻一下前面笔记看当时112233的密文，长得是不是跟现在的token值格式一样

### 退出登录

上面我们既测试了登录认证，又实现了基于密文的token认证，到此就完整地实现了我们在 '认证' 的 '3. 自定义security的思路' 里面的【登录】和【校验】的功能

那么，我们怎么退出登录呢，也就是让某个用户的登录状态消失，也就是让token失效 ?

实现起来也比较简单，只需要定义一个登陆接口，然后获取SecurityContextHolder中的认证信息，删除redis中对应的数据即可

注意: 我们的token其实就是用户密码的密文，token是存在redis里面

第一步: 把LoginService接口修改为如下，注意只是稍微增加了一点代码，我用虚线隔开了，增加的代码是在虚线的下方

```
package com.ehzyil.service;

import com.ehzyil.domain.ResponseResult;
import com.ehzyil.domain.User;
import org.springframework.stereotype.Service;


@Service
public interface LoginService {
    ResponseResult login(User user);

    //-----------------------------------退出登录--------------------------------

    ResponseResult logout();
}
```



第二步: 把LoginServiceImpl实现类修改为如下，注意只是稍微增加了一点代码，我用虚线隔开了，增加的代码是在虚线的下方

```java
@Service
@Slf4j
//写登录的核心代码
public class LoginServiceImpl implements LoginService {

    @Autowired
    //先在SecurityConfig，使用@Bean注解重写官方的authenticationManagerBean类，然后这里才能注入成功
    private AuthenticationManager authenticationManager;

    @Autowired
    //RedisCache是我们在utils目录写好的类
    private RedisCache redisCache;

    @Override
    //ResponseResult和user是我们在domain目录写好的类
    public ResponseResult login(User user) {

        //用户在登录页面输入的用户名和密码
        UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(user.getUserName(), user.getPassword());

        //获取AuthenticationManager的authenticate方法来进行用户认证
        Authentication authenticate = authenticationManager.authenticate(authenticationToken);

        //判断上面那行的authenticate是否为null，如果是则认证没通过，就抛出异常
        if (Objects.isNull(authenticate)) {
            throw new RuntimeException("登录失败");
        }

        //如果认证通过，就使用userid生成一个jwt，然后把jwt存入ResponseResult后返回
        LoginUser loginUser = (LoginUser) authenticate.getPrincipal();
        String userid = loginUser.getUser().getId().toString();
        String jwt = JwtUtil.createJWT(userid);

        //把完整的用户信息存入redis，其中userid作为key，注意存入redis的时候加了前缀 login:
        Map<String, String> map = new HashMap<>();
        map.put("token", jwt);
        redisCache.setCacheObject("login:" + userid, loginUser);
        log.info("将token存储到redis！");
        return new ResponseResult(200, "登录成功", map);
    }

    @Override
    public ResponseResult logout() {

        //获取我们在JwtAuthenticationTokenFilter类写的SecurityContextHolder对象中的用户id
        UsernamePasswordAuthenticationToken authentication = (UsernamePasswordAuthenticationToken) SecurityContextHolder.getContext().getAuthentication();
        //loginUser是我们在domain目录写好的实体类
        LoginUser loginUser = (LoginUser) authentication.getPrincipal();
        //获取用户id
        Long userid = loginUser.getUser().getId();

        //根据用户id，删除redis中的token值，注意我们的key是被 login: 拼接过的，所以下面写完整key的时候要带上 longin:
        redisCache.deleteObject("login:" + userid);

        return new ResponseResult(200, "注销成功");
    }
}
```

第三步: 把LoginController类修改为如下，注意只是稍微增加了一点代码，我用虚线隔开了，增加的代码是在虚线的下方

```java
    //-----------------------------------退出登录--------------------------------

    @RequestMapping("/user/logout")
    //ResponseResult是我们在domain目录写好的实体类
    public ResponseResult logout(){
        return loginService.logout();
    }
```

第四步: 本地打开你的redis

第五步: 运行引导类

第六步: 测试。打开你的postman，发送如下的POST请求，作用是先登录一个用户，这样就能生成这个用户对应的token值

第七步: 测试。继续在你的postman，发送如下GET请求，作用是拿着刚刚的token值，去访问我们的业务接口，看在有登录状态的情况下，能不能访问

注意还要带上你刚刚复制的token值，粘贴到消息头的Value输入框

第八步: 测试。继续在你的postman，发送如下GET请求，作用是退出登录，然后去访问我们的业务接口，看在没有登录状态的情况下，能不能访问



## 2.授权

### 授权的基本流程

在SpringSecurity中，会使用默认的FilterSecurityInterceptor来进行权限校验。在FilterSecurityInterceptor中会从SecurityContextHolder获取其中的Authentication，然后获取其中的权限信息。当前用户是否拥有访问当前资源所需的权限

![output](../../../images/Spring%20Security/output.png)



所以我们在项目中只需要把当前登录用户的权限信息也存入Authentication，然后设置我们的资源所需要的权限即可。

### 自定义访问路径的权限

SpringSecurity为我们提供了基于注解的权限控制方案，这也是我们项目中主要采用的方式。我们可以使用注解去指定访问对应的资源所需的权限

第一步: 在SecurityConfig配置类添加如下，作用是开启相关配置

```
@EnableGlobalMethodSecurity(prePostEnabled = true)
```

第二步: 开启了相关配置之后，就能使用@PreAuthorize等注解了。

```
  @PreAuthorize("hasAuthority('hello')")
```

在HelloController类的hello方法，添加如下注解，其中test表示自定义权限的名字

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    @PreAuthorize("hasAuthority('hello')")
    public String hello() {
        return "hello world";
    }

    @PreAuthorize("hasAuthority('test')")
    @GetMapping("/test")
    public String hello2() {
        return "hasAuthority('test')";
    }
}
```

### 带权限访问的实现

权限信息: 有特殊含义的字符串

我们前面在登录时，会调用到MyUserDetailServiceImpl类的loadUserByUsername方法，当时我们写loadUserByUsername方法时，只写了查询用户数据信息的代码，还差查询用户权限信息的代码。在登录完之后，因为携带了token，所以需要在JwtAuthenticationTokenFilter类添加 '获取权限信息封装到Authentication中' 的代码，添加到UsernamePasswordAuthenticationToken的第三个参数里面，我们当时第三个参数传的是null。

第一步: 在上面的**自定义访问路径的权限**，我们给HelloController类的"/hello"路径和"/test"路径添加了权限限制，只有用户具有叫hello或test的权限，才能访问这个路径。

第二步: 把MyUserDetailServiceImpl类修改为如下，主要是增加了查询用户权限信息的代码，权限列表暂时写死

```java
    @Override
    //UserDetails是Security官方提供的接口
    public UserDetails loadUserByUsername(String xxusername) throws UsernameNotFoundException {

        //查询用户信息。我们写的userMapper接口里面是空的，所以调用的是mybatis-plus提供的方法
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
        //eq方法表示等值匹配，第一个参数是数据库的用户名，第二个参数是我们传进来的用户名，这两个参数进行比较是否相等
        queryWrapper.eq(User::getUserName,xxusername);
        User user = userMapper.selectOne(queryWrapper);
        //如果用户传进来的用户名，但是数据库没有这个用户名，就会导致我们是查不到的情况，那么就进行下面的判断。避免程序安全问题
        if(Objects.isNull(user)){//判断user对象是否为空。当在数据库没有查到数据时，user就会为空，也就会进入这个判断
            throw new RuntimeException("用户名或者密码错误");
        }
        //--------------------------------查询用户权限信息---------------------------------

        //由于我们自定义了3个权限，所以用List集合存储。注意权限实际就是'有特殊含义的字符串'，所以下面的三个字符串就是自定义的
        //下面那行就是我们的权限集合，等下还要在LoginUser类做权限集合的转换
        List<String> list = new ArrayList<>(Arrays.asList("test","adminAuth","huanfAuth"));

        //------------------------------------------------------------------------------

        //把查询到的user结果，封装成UserDetails类型，然后返回。
        //但是由于UserDetails是个接口，所以我们先需要在domino目录新建LoginUser类，作为UserDetails的实现类，再写下面那行
        return new LoginUser(user,list); //这里传了第二个参数，表示的是权限信息
    }
```

第三步: 封装权限信息。把LoginUser类修改为如下，主要是增加了把用户权限字符串的集合，转换封装在authorities变量里面

```java
@Data //get和set方法
@NoArgsConstructor //无参构造
//实现UserDetails接口之后，要重写UserDetails接口里面的7个方法
public class LoginUser implements UserDetails {

    private User user;

    //查询用户权限信息
    private List<String> permissions;

    public LoginUser(User user, List<String> permissions) {
        this.user = user;
        this.permissions = permissions;
    }

    //我们把这个List写到外面这里了，注意成员变量名一定要是authorities，不然会出现奇奇怪怪的问题
    @JSONField(serialize = false) //这个注解的作用是不让下面那行的成员变量序列化存入redis，避免redis不支持而报异常
    private List<SimpleGrantedAuthority> authorities;

    @Override
    //用于返回权限信息。现在我们正在学'认证'，'权限'后面才学。所以返回null即可
    //当要查询用户信息的时候，我们不能单纯返回null，要重写这个方法，作用是封装权限信息
    public Collection<? extends GrantedAuthority> getAuthorities() { //重写GrantedAuthority接口的getAuthorities方法

        /* 第一种权限集合的转换写法如下，传统的方式
        //把xxpermissions中的String类型的权限信息(也就是"test","adminAuth","huanfAuth")封装成SimpleGrantedAuthority对象
        //List<GrantedAuthority> authorities = new ArrayList<>(); //简化这行如下一行，我们把authorities成员变量写到外面了
        authorities = new ArrayList<>();
        for (String yypermission : xxpermissions) {
            SimpleGrantedAuthority yyauthority = new SimpleGrantedAuthority(yypermission);
            authorities.add(yyauthority);
        }
        */

        /* 第二种权限集合的转换写法如下，函数式编程 + stream流 的方式，双引号表示方法引用 */
        //当authorities集合为空，就说明是第一次，就需要转换，当不为空就说明不是第一次，就不需要转换直接返回
        if(authorities != null){ //严谨来说这个if判断是避免整个调用链中security本地线程变量在获取用户时的重复解析，和redis存取无关
            return authorities;
        }
        //为空的话就会执行下面的转换代码
        //List<SimpleGrantedAuthority> authorities = xxpermissions //简化这行如下一行，我们把authorities成员变量写到外面了
        authorities = permissions
                .stream()
                .map(SimpleGrantedAuthority::new)
                .collect(Collectors.toList());

        //最终返回转换结果
        return authorities;
    }

    @Override
    //用于获取用户密码。由于使用的实体类是User，所以获取的是数据库的用户密码
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    //用于获取用户名。由于使用的实体类是User，所以获取的是数据库的用户名
    public String getUsername() {
        return user.getUserName();
    }

    @Override
    //判断登录状态是否过期。把这个改成true，表示永不过期
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    //判断账号是否被锁定。把这个改成true，表示未锁定，不然登录的时候，不让你登录
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    //判断登录凭证是否过期。把这个改成true，表示永不过期
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    //判断用户是否可用。把这个改成true，表示可用状态
    public boolean isEnabled() {
        return true;
    }
}
```

第四步: 把JwtAuthenticationTokenFilter类修改为如下，主要是补充了前面没写的第三个参数，写成第三步封装好的权限信息

```java
 		//把最终的LoginUser用户信息，通过setAuthentication方法，存入SecurityContextHolder
        UsernamePasswordAuthenticationToken authenticationToken =
                //第一个参数是LoginUser用户信息，第二个参数是凭证(null)，第三个参数是权限信息(null)
                //在学习封装权限信息时，就要把下面的第三个参数补充完整，getAuthorities是我们在loginUser写的方法
                new UsernamePasswordAuthenticationToken(loginUser,null,loginUser.getAuthorities());
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);
```



第五步: 测试。打开postman，发送如下的POST请求，作用是先登录一个用户，这样就能生成这个用户对应的token值

![image-20230916154240269](../../../images/Spring%20Security/image-20230916154240269.png)



继续在postman，发送如下GET请求，作用是拿着刚刚的token值，去访问我们的业务接口，看在有登录状态的情况下，能不能访问

<img src="../../../images/Spring%20Security/image-20230916154411093.png" alt="image-20230916154411093" style="zoom:67%;" />

继续访问hello路径因为没有赋予用户hello权限，因此禁止访问。

<img src="../../../images/Spring%20Security/image-20230916154450600.png" alt="image-20230916154450600" style="zoom: 67%;" />

##  3.授权-RBAC权限模型

刚刚我们实现了只有当用户具备某种权限，才能访问我们的某个业务接口。但是存在一个问题，我们在给用户设置权限的时候，是写死的，在真正的开发中，我们是需要从数据库查询权限信息，下面就来学习如何从数据库查询权限信息，然后封装给用户。这个功能需要先准备好数据库和java代码，所以，下面的 '授权-RBAC权限模型' 都是在围绕这个功能进行学习，直到实现这个功能

![image-20230916154646456](../../../images/Spring%20Security/image-20230916154646456.png)

### 介绍

RBAC权限模型 (Role-Based Access Control) ，是权限系统用到的经典模型，基于角色的权限控制。该模型由以下五个主要组成部分构成:

一、用户: 在系统中代表具体个体的实体，可以是人员、程序或其他实体。用户需要访问系统资源

二、角色: 角色是权限的集合，用于定义一组相似权限的集合。角色可以被赋予给用户，从而授予用户相应的权限

三、权限: 权限表示系统中具体的操作或功能，例如读取、写入、执行等。每个权限定义了对系统资源的访问规则

四、用户-角色映射: 用户-角色映射用于表示用户与角色之间的关系。通过为用户分配适当的角色，用户可以获得与角色相关联的权限

五、角色-权限映射: 角色-权限映射表示角色与权限之间的关系。每个角色都被分配了一组权限，这些权限决定了角色可执行的操作

截止目前，我们数据库只有 sys_user 用户表，下面我们会新增4张表，分别是权限表(每条数据是单个'粒度细的权限')、角色表(每条数据是多个'粒度细的权限')、角色表与权限表的中间表、用户表与角色表的中间表。总共5张表，组成了RBAC模型，中间表的作用是维护两张表的多对多关系.

### 数据库表的创建

第一步: 在你数据库的huanf_security 库，新建 sys_menu权限表、sys_role角色表、sys_role_menu中间表、sys_user_role中间表，并插入数据

```sql
create database if not exists huanf_security;
use huanf_security;
CREATE TABLE `sys_menu` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `menu_name` varchar(64) NOT NULL DEFAULT 'NULL' COMMENT '菜单名',
  `path` varchar(200) DEFAULT NULL COMMENT '路由地址',
  `component` varchar(255) DEFAULT NULL COMMENT '组件路径',
  `visible` char(1) DEFAULT '0' COMMENT '菜单状态（0显示 1隐藏）',
  `status` char(1) DEFAULT '0' COMMENT '菜单状态（0正常 1停用）',
  `perms` varchar(100) DEFAULT NULL COMMENT '权限标识',
  `icon` varchar(100) DEFAULT '#' COMMENT '菜单图标',
  `create_by` bigint(20) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `update_by` bigint(20) DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  `del_flag` int(11) DEFAULT '0' COMMENT '是否删除（0未删除 1已删除）',
  `remark` varchar(500) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COMMENT='权限表';

CREATE TABLE `sys_role` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(128) DEFAULT NULL,
  `role_key` varchar(100) DEFAULT NULL COMMENT '角色权限字符串',
  `status` char(1) DEFAULT '0' COMMENT '角色状态（0正常 1停用）',
  `del_flag` int(1) DEFAULT '0' COMMENT 'del_flag',
  `create_by` bigint(200) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `update_by` bigint(200) DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  `remark` varchar(500) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COMMENT='角色表';

CREATE TABLE `sys_role_menu` (
  `role_id` bigint(200) NOT NULL AUTO_INCREMENT COMMENT '角色ID',
  `menu_id` bigint(200) NOT NULL DEFAULT '0' COMMENT '菜单id',
  PRIMARY KEY (`role_id`,`menu_id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4;

CREATE TABLE `sys_user_role` (
  `user_id` bigint(200) NOT NULL AUTO_INCREMENT COMMENT '用户id',
  `role_id` bigint(200) NOT NULL DEFAULT '0' COMMENT '角色id',
  PRIMARY KEY (`user_id`,`role_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

insert into sys_user_role values (2,1);
insert into sys_role values
(1,'经理','ceo',0,0,default,default,default,default,default),
(2,'程序员','coder',0,0,default,default,default,default,default);
insert into sys_role_menu values (1,1),(1,2);
insert into sys_menu values
(1,'部门管理','dept','system/dept/index',0,0,'system:dept:list','#',default,default,default,default,default,default),
(2,'测试','test','system/test/index',0,0,'system:test:list','#',default,default,default,default,default,default)
```

第二步: 测试SQL语句，也就是确认一下你的建表、插入数据是否达到要求

```sql
# 通过用户id去查询这个用户具有的权限列表。也就是根据userid查询perms，并且限制条件为role和menu都必须正常状态么也就是等于0
SELECT 
	DISTINCT m.`perms`
FROM
	sys_user_role ur
	LEFT JOIN `sys_role` r ON ur.`role_id` = r.`id`
	LEFT JOIN `sys_role_menu` rm ON ur.`role_id` = rm.`role_id`
	LEFT JOIN `sys_menu` m ON m.`id` = rm.`menu_id`
WHERE
	user_id = 2
	AND r.`status` = 0
	AND m.`status` = 0
```





### 查询数据库的权限信息

第一步: 在 domain 目录新建 Menu 实体类，写入如下

```java
//权限表(也叫菜单表)的实体类
@TableName(value="sys_menu") //指定表名，避免等下mybatisplus的影响
@Data
@AllArgsConstructor
@NoArgsConstructor
@JsonInclude(JsonInclude.Include.NON_NULL)
//Serializable是官方提供的，作用是将对象转化为字节序列
public class Menu implements Serializable {
    private static final long serialVersionUID = -54979041104113736L;

    @TableId
    private Long id;
    /**
     * 菜单名
     */
    private String menuName;
    /**
     * 路由地址
     */
    private String path;
    /**
     * 组件路径
     */
    private String component;
    /**
     * 菜单状态（0显示 1隐藏）
     */
    private String visible;
    /**
     * 菜单状态（0正常 1停用）
     */
    private String status;
    /**
     * 权限标识
     */
    private String perms;
    /**
     * 菜单图标
     */
    private String icon;

    private Long createBy;

    private Date createTime;

    private Long updateBy;

    private Date updateTime;
    /**
     * 是否删除（0未删除 1已删除）
     */
    private Integer delFlag;
    /**
     * 备注
     */
    private String remark;
}
```



第二步: 在 mapper 目录新建 MenuMapper 接口。作用是定义mapper，其中提供一个方法可以根据userid查询权限信息

```
@Mapper
//BaseMapper是mybatisplus官方提供的接口，里面提供了很多单表查询的方法
public interface MenuMapper extends BaseMapper<Menu> {
    ////由于是多表联查，mybatisplus的BaseMapper接口没有提供，我们需要自定义方法，所以需要创建对应的mapper文件，定义对应的sql语句
    List<String> selectPermsByUserId(Long id);
}

```



第三步: 在 resources 目录新建 mapper目录，接着在这个mapper目录新建File，名字叫 MenuMapper.xml，写入如下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ehzyil.mapper.MenuMapper">

    <select id="selectPermsByUserId" resultType="java.lang.String">
        SELECT
            DISTINCT m.`perms`
        FROM
            sys_user_role ur
                LEFT JOIN `sys_role` r ON ur.`role_id` = r.`id`
                LEFT JOIN `sys_role_menu` rm ON ur.`role_id` = rm.`role_id`
                LEFT JOIN `sys_menu` m ON m.`id` = rm.`menu_id`
        WHERE
            user_id = #{userid}
          AND r.`status` = 0
          AND m.`status` = 0
    </select>

</mapper>
```



第四步: 把application.yml修改为如下，作用是告诉MP，刚刚写的MenuMapper.xml文件是在哪个地方

```yaml
mybatis-plus:
  # 配置MenuMapper.xml文件的路径
  # 也可以不写，因为默认就是在类加载路径(resouces)下的mapper目录的任意层级的后缀为xml的文件，都会被扫描到
  mapper-locations: classpath*:/mapper/**/*.xml
```

第五步: 测试。这里只是检查mybatismlus能不能拿到数据库的权限字符串。在MapperTest类添加如下

```
@Autowired
private MenuMapper menuMapper;

@Test
public void testSelectPermsByUserId(){
	//L表示Long类型
	List<String> list = menuMapper.selectPermsByUserId(2L);
	System.out.println(list);
}
```

###  RBAC权限模型的实现

不要把RBAC模型想得很难，其实难的话只是数据库表的设计和SQL语句的编写，需要5张表。数据库设计好之后就很简单了，使用mybatis-plus去查询数据库表的权限字符串(例如我们的权限字符串是放在sys_menu表)，然后把你查到的数据去替换死数据就好了。我们只剩最后一步，就是替换死数据，如下

第一步: 把MyUserDetailServiceImpl类修改为如下，我们只是增加了查询来自数据库的权限信息的代码

```
   @Autowired
    private UserMapper userMapper;

    @Autowired
    //MenuMapper是我们在mapper目录写好的接口，作用是查询来自数据库的权限信息
    private MenuMapper menuMapper;

    @Override
    //UserDetails是Security官方提供的接口
    public UserDetails loadUserByUsername(String xxusername) throws UsernameNotFoundException {

        //查询用户信息。我们写的userMapper接口里面是空的，所以调用的是mybatis-plus提供的方法
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
        //eq方法表示等值匹配，第一个参数是数据库的用户名，第二个参数是我们传进来的用户名，这两个参数进行比较是否相等
        queryWrapper.eq(User::getUserName,xxusername);
        User user = userMapper.selectOne(queryWrapper);
        //如果用户传进来的用户名，但是数据库没有这个用户名，就会导致我们是查不到的情况，那么就进行下面的判断。避免程序安全问题
        if(Objects.isNull(user)){//判断user对象是否为空。当在数据库没有查到数据时，user就会为空，也就会进入这个判断
            throw new RuntimeException("用户名或者密码错误");
        }
        //--------------------------------查询用户权限信息---------------------------------

        //由于我们自定义了3个权限，所以用List集合存储。注意权限实际就是'有特殊含义的字符串'，所以下面的三个字符串就是自定义的
        //下面那行就是我们的权限集合，等下还要在LoginUser类做权限集合的转换
//        List<String> list = new ArrayList<>(Arrays.asList("test","adminAuth","huanfAuth"));


        //-------------------------------查询来自数据库的权限信息--------------------------------

        List<String> list = menuMapper.selectPermsByUserId(user.getId());

        //------------------------------------------------------------------------------

        //把查询到的user结果，封装成UserDetails类型，然后返回。
        //但是由于UserDetails是个接口，所以我们先需要在domino目录新建LoginUser类，作为UserDetails的实现类，再写下面那行
        return new LoginUser(user,list); //这里传了第二个参数，表示的是权限信息
    }
```



第二步: 由于我们知道数据库传过来的权限字符串是 system:dept:list 和 system:test:list，所以我们要把HelloController类的权限字符串修改为如下

```
system:test:list
```

第三步: 测试。

## 自定义异常处理



面的我们学习了 '认证' 和 '授权'，实现了基本的权限管理，然后也学习了从数据库获取授权的 '授权-RBAC权限模型'，实现了从数据库获取用户具备的权限字符串。到此，我们完整地实现了权限管理的功能，但是，当认证或授权出现报错时，我们希望响应回来的json数据有实体类的code、msg、data这三个字段，怎么实现呢

我们需要学习Spring Security的异常处理机制，就可以在认证失败或者是授权失败的情况下也能和我们的接口一样返回相同结构的json，这样可以让前端能对响应进行统一的处理

![output](../../../images/Spring%20Security/output.png)



在SpringSecurity中，如果我们在认证或者授权的过程中出现了异常会被ExceptionTranslationFilter捕获到，如上图。在ExceptionTranslationFilter中会去判断是认证失败还是授权失败出现的异常，其中有如下两种情况

一、如果是认证过程中出现的异常会被封装成AuthenticationException然后调用AuthenticationEntryPoint对象的方法去进行异常处理。

二、如果是授权过程中出现的异常会被封装成AccessDeniedException然后调用AccessDeniedHandler对象的方法去进行异常处理。

总结: 如果我们需要自定义异常处理，我们只需要创建AuthenticationEntryPoint和AccessDeniedHandler的实现类对象，然后配置给SpringSecurity即可



第一步: 在 src/main/java/com.ehzyil 目录新建 handler.AuthenticationEntryPointImpl类，写入如下，作用是自定义认证的实现类

```java
@Component
//这个类只处理认证异常，不处理授权异常
public class AuthenticationEntryPointImpl implements AuthenticationEntryPoint {
    @Override
    //第一个参数是请求对象，第二个参数是响应对象，第三个参数是异常对象。把异常封装成授权的对象，然后封装到handle方法
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        //ResponseResult是我们在domain目录写好的实体类。HttpStatus是spring提供的枚举类，UNAUTHORIZED表示401状态码
        ResponseResult result = new ResponseResult(HttpStatus.UNAUTHORIZED.value(), "用户认证失败，请重新登录");
        //把上面那行拿到的result对象转换为JSON字符串
        String json = JSON.toJSONString(result);
        //WebUtils是我们在utils目录写好的类
        WebUtils.renderString(response, json);
    }
}
```



第二步: 在handler目录新建 AccessDeniedHandlerImpl 类，写入如下，作用是自定义授权的实现类

```java
@Component
//这个类只处理授权异常，不处理认证异常
public class AccessDeniedHandlerImpl implements AccessDeniedHandler {
    @Override
    //第一个参数是请求对象，第二个参数是响应对象，第三个参数是异常对象。把异常封装成认证的对象，然后封装到handle方法
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        //ResponseResult是我们在domain目录写好的实体类。HttpStatus是spring提供的枚举类，FORBIDDEN表示403状态码
        ResponseResult result = new ResponseResult(HttpStatus.FORBIDDEN.value(), "您没有权限进行访问");
        //把上面那行拿到的result对象转换为JSON字符串
        String json = JSON.toJSONString(result);
        //WebUtils是我们在utils目录写好的类
        WebUtils.renderString(response, json);
    }
}
```



第三步: 把 SecurityConfig 类修改为如下，作用是把刚刚两个异常处理的实现类配置在Spring Security里面

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
//实现Security提供的WebSecurityConfigurerAdapter类，就可以改变密码校验的规则了
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    //注入我们在filter目录写好的类
    private JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;
    @Autowired
    //注入Security提供的认证失败的处理器，这个处理器里面的AuthenticationEntryPointImpl实现类，用的不是官方的了，
    //而是用的是我们在handler目录写好的AuthenticationEntryPointImpl实现类
    private AuthenticationEntryPoint authenticationEntryPoint;

    @Autowired
    //注入Security提供的授权失败的处理器，这个处理器里面的AccessDeniedHandlerImpl实现类，用的不是官方的了，
    //而是用的是我们在handler目录写好的AccessDeniedHandlerImpl实现类
    private AccessDeniedHandler accessDeniedHandler;

    //---------------------------认证过滤器的实现----------------------------------

    @Bean
    //把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验
    //注意也可以注入PasswordEncoder，效果是一样的，因为PasswordEncoder是BCry..的父类
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    //---------------------------登录接口的实现----------------------------------

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //由于是前后端分离项目，所以要关闭csrf
                .csrf().disable()
                //由于是前后端分离项目，所以session是失效的，我们就不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                //指定让spring security放行登录接口的规则
                .authorizeRequests()
                // 对于登录接口 anonymous表示允许匿名访问
                .antMatchers("/user/login").anonymous()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();

        //---------------------------认证过滤器的实现----------------------------------

        //把token校验过滤器添加到过滤器链中
        //第一个参数是上面注入的我们在filter目录写好的类，第二个参数表示你想添加到哪个过滤器之前
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
        //---------------------------异常处理的相关配置-------------------------------

        http.exceptionHandling()
                //配置认证失败的处理器
                .authenticationEntryPoint(authenticationEntryPoint)
                //配置授权失败的处理器
                .accessDeniedHandler(accessDeniedHandler);

    }
}
```

第六步: 测试认证异常。打开你的postman，发送如下的POST请求，作用是登录一个不存在的用户，模拟认证异常

<img src="../../../images/Spring%20Security/image-20230916173432352.png" alt="image-20230916173432352" style="zoom:67%;" />



第七步: 测试授权异常。先在HelloController类修改PreAuthorize注解的权限字符串，修改成huanf用户不存在的权限字符串，接着重新运行TokenApplication引导类，然后去正常登录一个用户并访问 /hello 业务接口，必然会报权限异常，然后我们看一下响应回来的数据格式，是不是我们定义的json格式

<img src="../../../images/Spring%20Security/image-20230916173603895.png" alt="image-20230916173603895" style="zoom:67%;" />

## 跨域



### 跨域的后端解决

由于我们的SpringSecurity负责所有请求和资源的管理，当请求经过SpringSecurity时，如果SpringSecurity不允许跨域，那么也是会被拦截，所以下面我们将学习并解决跨域问题。前面我们在测试时，是在postman测试，因此没有出现跨域问题的情况，postman只是负责发请求跟浏览器没关系

浏览器出于安全的考虑，使用 XMLHttpRequest 对象发起HTTP请求时必须遵守同源策略，否则就是跨域的HTTP请求，默认情况下是被禁止的。 同源策略要求源相同才能正常进行通信，即协议、域名、端口号都完全一致。 前后端分离项目，前端项目和后端项目一般都不是同源的，所以肯定会存在跨域请求的问题

我们要实现如下两个需求 (我实际做出的效果跟教程视频不一致，第二个需求其实没必要存在，boot解决了跨域就都解决了):

1、开启SpringBoot的允许跨域访问

2、开启SpringSecurity的允许跨域访问

第一步: 开启SpringBoot的允许跨域访问。在 config 目录新建 CorsConfig 类，写入如下

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    //重写spring提供的WebMvcConfigurer接口的addCorsMappings方法
    public void addCorsMappings(CorsRegistry registry) {
        // 设置允许跨域的路径
        registry.addMapping("/**")
                // 设置允许跨域请求的域名
                .allowedOriginPatterns("*")
                // 是否允许cookie
                .allowCredentials(true)
                // 设置允许的请求方式
                .allowedMethods("GET", "POST", "DELETE", "PUT")
                // 设置允许的header属性
                .allowedHeaders("*")
                // 跨域允许时间
                .maxAge(3600);
    }
}
```



第二步: 开启SpringSecurity的允许跨域访问。在把 SecurityConfig 修改为如下。



```java
  protected void configure(HttpSecurity http) throws Exception { 
  ...
  //--------------------------- 设置security运行跨域访问 ------------------

        http.cors();
        }
```



第三步: 由于没有前端项目，所以我们下面会跑一个前端项目，然后测试后端的跨域功能

第四步: 运行前端项目。请确保你电脑有安装node.js，然后以管理员身份打开命令行窗口，输入如下

```shell
npm install
npm run serve
```

第五步: 访问前端的项目

```
http://localhost:8080/#/login
```



第六步: 由于这个前端项目的要求后端服务的端口是8888，所以我们要修改一下idea的application.yml文件，把默认的8089端口改为8888端口,为了等下直观的看出跨域的问题，我们把后端的 boot与security的跨域代码注释掉

```
server:
  # 指定后端的启动端口
  port: 8888
```

第七步: 测试。访问前端项目，登录一个数据库的真实用户，假的也行，主要是看请求会不会出现跨域问题

![image-20230916175553357](../../../images/Spring%20Security/image-20230916175553357.png)

第八步: 我后来呢，把security的跨域单独注释了，然后重新运行引导类，想验证是不是只要security不允许跨域，那么即使boot允许跨域，那最终也是不允许跨域的，原因是资源必然要经过security。但是实验结果却是security注释了之后，仅靠boot的跨域放行，前端竟然也能正常访问后端接口	

## 授权-权限校验的方法

上面学的是HelloController类的 @PreAuthorize注解 的三个方法

我们前面都是使用@PreAuthorize注解，然后在在其中使用的是hasAuthority方法进行校验。SpringSecurity还为我们提供了其它方法例如: hasAnyAuthority，hasRole，hasAnyRole等

### 1. hasAuthority方法

我们最早使用@PreAuthorize注解是在前面笔记的 '授权' 的**自定义访问路径的权限**。可以回去看看，当时并没有详细学习@PreAuthorize注解的hasAuthority方法，只是直接使用，我们下面就来学习一下hasAuthority方法的原理，然后再学习hasAnyAuthority、hasRole、hasAnyRole方法就容易理解了

hasAuthority方法: 执行到了SecurityExpressionRoot的hasAuthority，内部其实是调用authentication的getAuthorities方法获取用户的权限列表。然后判断我们存入的方法参数数据在权限列表中。hasAnyAuthority方法可以传入多个权限，只有用户有其中任意一个权限都可以访问对应资源

hasAuthority方法的执行流程如下图，图比较多，请从上往下查看

![img](../../../images/Spring%20Security/1689586885764-72155704-ba7a-4d17-8feb-3cd474fd32a1.jpg)

![img](../../../images/Spring%20Security/1689586885926-9de01534-ab7d-4153-89b3-7f087f560dfa.jpg)

![img](../../../images/Spring%20Security/1689586886073-9487ef1e-1085-4d96-a9fd-43adca4b540c.jpg)

![img](../../../images/Spring%20Security/1689586886239-534ab5b9-7023-47d0-9b79-89bb0dec9758.jpg)

![img](../../../images/Spring%20Security/1689586886382-9e028350-b4d3-4aba-a9e6-227590646b47.jpg)



### 2. hasAnyAuthority方法

hasAnyAuthority方法的执行流程跟上面的hasAuthority方法是一样的，只是传参不同。所以重点演示传参，执行流程是真的一模一样，把上面的hasAuthority方法的执行流程认真看一次就行了。hasAuthority方法只能传入一个参数，也就是一个权限字符串。hasAnyAuthority方法可以传入多个参数，也就是多个权限字符串，只要用户具有其中任意一个权限就能访问指定业务接口

第一步: 为方便演示hasAnyAuthority方法能够传入多个参数并看到最终效果，我们把HelloController类的@RequestMapping注解修改为如下

```
@PreAuthorize("hasAnyAuthority('zidingyi','huanf','system:dept:list')"); //传入3个自定义权限
```

第二步: 运行引导类，打开postman，发送登录请求，然后发送访问/hello接口的请求

### 3. hasRole方法

首先是执行流程，如下几张图，从上往下看，很精华没有尿点

![img](../../../images/Spring%20Security/1689586886881-a743d733-8c7b-46d7-99b9-140df7301898.jpg)

![img](../../../images/Spring%20Security/1689586887015-870697dc-000b-40ac-88df-054c4600011c.jpg)

![img](../../../images/Spring%20Security/1689586887160-8d38aca6-643c-4104-a7b9-b6d8df5db58d.jpg)

下图是打断点后的测试图![img](../../../images/Spring%20Security/1689586887309-ea19e33c-7576-4f27-a056-5f18eb76cabb.jpg)

然后是测试的操作，如下

第一步: 为方便演示hasAnyAuthority方法能够传入多个参数并看到最终效果，我们把HelloController类的@RequestMapping注解修改为如下

```java
@PreAuthorize("hasRole('system:dept:list')") //只能传一个权限字符串，多传会报红线
```

第二步:  运行引导类，打开postman软件，发送登录请求，然后发送访问/hello接口的请求

![img](../../../images/Spring%20Security/1689586887435-ab465af1-bb20-4f60-a281-32b781d95812.jpg)

![img](https://cdn.nlark.com/yuque/0/2023/jpg/35597420/1689586887559-fc967182-5f78-4473-aef1-f685c971bd9b.jpg)

### 4. hasAnyRole方法

执行流程跟刚刚的hasRole方法是一模一样的，只是传参不同。把上面的hasRole方法的执行流程认真看一次就行了。hasRole方法只能传入一个参数，也就是一个权限字符串。hasAnyRole方法可以传入多个参数，也就是多个权限字符串，只要用户具有其中任意一个权限就能访问指定业务接口

第一步: 为方便演示hasAnyAuthority方法能够传入多个参数并看到最终效果，我们把HelloController类的@RequestMapping注解修改为如下

```java
@PreAuthorize("hasAnyRole('zidingyi','huanf','system:dept:list')")
```

![img](../../../images/Spring%20Security/1689586886517-80a53221-6a42-4a2d-b849-ef0dff6010fb.jpg)

第二步: 运行引导类，打开postman软件，发送登录请求，然后发送访问/hello接口的请求

![img](../../../images/Spring%20Security/1689586887696-95119a19-107c-4742-9d89-e71b2e059b66.jpg)

### 5.自定义权限校验的方法

在上面的源码中，我们知道security校验权限的PreAuthorize注解，其实就是获取用户权限，然后跟业务接口的权限进行比较，最后返回一个布尔类型。自定义一个权限校验方法的话，就需要新建一个类，在类里面定义一个方法，按照前面学习的三种方法的定义格式，然后返回值是布尔类型。如下

第一步: 在 src/main/java/com.ehzyil目录新建 expression.HuanfExpressionRoot 类，写入如下

```

@Component("huanfEX")
//自定义权限校验的方法
public class ExpressionRoot {

    //自定义权限校验的方法
    public boolean huanfHasAuthority(String authority) {

        //获取用户具有的权限字符串，有可能用户具有多个权限字符串，所以获取后是一个集合
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        //LoginUser是我们在domain目录写好的实体类
        LoginUser loginUser = (LoginUser) authentication.getPrincipal();
        List<String> permissions = loginUser.getPermissions();

        //判断用户权限集合中，是否存在跟业务接口(业务接口的权限字符串会作为authority形参传进来)一样的权限
        return permissions.contains(authority);
    }

}
```

第二步: 刚刚在第一步自定义了huanfHasAuthority方法，用于权限校验，那么如何让 @PreAuthorize 注解去使用huanfHasAuthority方法，只需要在SPEL表达式来获取容器中bean的名字。修改HelloController类为如下 

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    //官方提供的4个权限校验的方法
    //@PreAuthorize("hasAuthority('system:dept:list')")
    //@PreAuthorize("hasAnyAuthority('zidingyi','huanf','system:dept:list')")
    //@PreAuthorize("hasRole('system:dept:list')")
    //@PreAuthorize("hasAnyRole('zidingyi','huanf','system:dept:list')")

    //自定义权限校验的方法，huanfHasAuthority
    @PreAuthorize("@huanfEX.huanfHasAuthority('system:dept:list')")
    public String hello() {
        return "欢迎，开始你新的学习旅程吧";
    }


    @PreAuthorize("hasAuthority('test')")
    @GetMapping("/test")
    public String hello2() {
        return "hasAuthority('test')";
    }
}
```

使用SPEL表达式，指定@PreAuthorize注解使用我们自定义的huanfHasAuthority方法

第三步: 运行引导类，打开postman软件，发送登录请求，然后发送访问/hello接口的请求

<img src="../../../images/Spring%20Security/image-20230916184850233.png" alt="image-20230916184850233" style="zoom:67%;" />



### 6.基于配置的权限控制

前面学习的权限控制是基于@PreAuthorize注解来完成的，如何使用配置的方式，也就是在配置类当中，来实现权限控制，如下

第一步: 在HelloController类添加如下，作用是新增一个接口

```
//基于配置的权限控制
@RequestMapping("/configAuth")
public ResponseResult xx(){
	return new ResponseResult(200,"访问成功");
}
```

第二步: 把SecurityConfig类修改为如下，主要就是添加权限控制相关的配置

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //由于是前后端分离项目，所以要关闭csrf
                .csrf().disable()
                //由于是前后端分离项目，所以session是失效的，我们就不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                //指定让spring security放行登录接口的规则
                .authorizeRequests()
                // 对于登录接口 anonymous表示允许匿名访问
                .antMatchers("/user/login").anonymous()

                //基于配置的的权限控制。指定接口的地址，为HelloController类里面的/configAuth接口，指定权限为system:dept:list
                .antMatchers("/configAuth").hasAuthority("system:dept:list")
                //上一行的hasAuthority方法就是security官方提供的4种权限控制的方法之一

                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();
        //---------------------------认证过滤器的实现----------------------------------

        //把token校验过滤器添加到过滤器链中
        //第一个参数是上面注入的我们在filter目录写好的类，第二个参数表示你想添加到哪个过滤器之前
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
        //---------------------------异常处理的相关配置-------------------------------

        http.exceptionHandling()
                //配置认证失败的处理器
                .authenticationEntryPoint(authenticationEntryPoint)
                //配置授权失败的处理器
                .accessDeniedHandler(accessDeniedHandler);
        //---------------------------👇 设置security运行跨域访问 👇------------------

        http.cors();
    }
```

![img](../../../images/Spring%20Security/1689586888192-b61e1da0-dff2-41c6-8bb1-384ea1c7fb3f.jpg)

第三步: 运行引导类，打开postman软件，发送登录请求，然后发送访问/configAuth接口的请求

![img](../../../images/Spring%20Security/1689586888358-09dcb1cf-6f1a-4ed9-914a-871b65df13e2.jpg)

![img](../../../images/Spring%20Security/1689586888479-4ee4ae28-4b76-4320-85a0-2518cb2b3658.jpg)

![img](../../../images/Spring%20Security/1689586888616-265846a4-d9ef-4a10-b19c-6365b46036b5.jpg)

## 防护CSRF攻击

在SecurityConfig类里面的configure方法里面，有一个配置如下，我们上面都没有去学习，下面就来了解一下

```java
http..csrf().disable(); //关闭csrf，可防护csrf攻击。如果不关闭的话
```

![img](../../../images/Spring%20Security/1689586888733-1603a2e5-87fc-4eb6-ae9e-4e696636822a.jpg)

CSRF是指跨站请求伪造（Cross-site request forgery），是web常见的攻击之一，如图

![img](../../../images/Spring%20Security/1689586888852-2edfebdb-9d54-4bce-8501-6f698d2e29ab.jpg)

```plain
详细看这篇博客: https://blog.csdn.net/freeking101/article/details/86537087
```

防护: SpringSecurity去防止CSRF攻击的方式就是通过csrf_token。后端会生成一个csrf_token，前端发起请求的时候需要携带这个csrf_token,后端会有过滤器进行校验，如果没有携带或者是伪造的就不允许访问

我们可以发现CSRF攻击依靠的是cookie中所携带的认证信息。但是在前后端分离的项目中我们的认证信息其实是token，而token并不是存储中cookie中，并且需要前端代码去把token设置到请求头中才可以，所以CSRF攻击也就不用担心了，前后端分离的项目，在配置类关闭csrf就能防范csrf攻击





<hr/>

参考：https://www.yuque.com/huanfqc/springsecurity/springsecurity

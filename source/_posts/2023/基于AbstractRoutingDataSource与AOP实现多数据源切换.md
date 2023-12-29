---
title: 基于AbstractRoutingDataSource与AOP实现多数据源切换
author: ehzyil
tags:
  - SpringBoot
  - MySQL
categories:
  - 技术
date: 2023-12-29

---





## 环境准备

### 环境配置



[代码地址](https://github.com/ehZyiL/SpringBootStudy/tree/master/multi-datasource)



```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>30.1-jre</version>
</dependency>

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.1</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.16</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```



application.yml

```
# 默认的数据库名
database:
  name: story

spring:
  dynamic:
    primary: master11 # 这个表示默认的数据源
    datasource:
      master:
        # 数据库名，从配置 database.name 中获取
        url: jdbc:mysql://127.0.0.1:3306/story?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai
        username: root
        password: 666666
        # 注意下面这个type,选择DruidDataSource时，请确保项目中添加了druid相关依赖
        type: com.alibaba.druid.pool.DruidDataSource
        #DruidDataSource自有属性
        filters: stat
        initialSize: 0
        minIdle: 1
        maxActive: 200
        maxWait: 10000
        time-between-eviction-runs-millis: 60000
        min-evictable-idle-time-millis: 200000
        testWhileIdle: true
        testOnBorrow: true
        validationQuery: select 1
      shop:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://127.0.0.1:3306/shop?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai
        username: root
        password: 666666
      blog:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://127.0.0.1:3306/sg_blog?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai
        username: root
        password: 666666


logging:
  level:
    default: debug
mybatis:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    map-underscore-to-camel-case: true
```

### 基础知识点

主要借助AbstractRoutingDataSource来实现动态数据源，简单介绍下它：

AbstractRoutingDataSource 是 Spring 提供的一个抽象类，它可以让我们动态切换数据源。在使用 AbstractRoutingDataSource 时，我们需要**实现它的 determineCurrentLookupKey() 方法，该方法返回当前线程使用的数据源的名称或标识符。**

AbstractRoutingDataSource 两个重要的成员变量：

1. defaultTargetDataSource：默认数据源，即当无法确定使用哪个数据源时，将使用该默认数据源。在AbstractRoutingDataSource中，我们可以通过调用setDefaultTargetDataSource()方法来设置默认数据源。
2. targetDataSources：一个Map类型的成员变量，其中保存了多个数据源实例。在AbstractRoutingDataSource中，我们可以通过调用setTargetDataSources()方法来设置多个数据源实例，其中Map的key为数据源标识符或名称，value为对应的数据源实例。



使用AbstractRoutingDataSource实现数据源切换的原理如下：

1. 在应用启动时，我们需要配置多个数据源，并将它们注册到AbstractRoutingDataSource中。
2. 当应用发起数据库操作时，AbstractRoutingDataSource会根据determineCurrentLookupKey()方法返回的值，查找对应的数据源。
3. 在determineCurrentLookupKey()方法中，我们可以根据不同的策略来决定使用哪个数据源。例如，可以根据当前请求的用户信息、请求的URL、当前时间等来选择不同的数据源。

其核心代码如：

```java
// 返回选中的数据源
protected DataSource determineTargetDataSource() {
    Assert.notNull(this.resolvedDataSources, "DataSource router not initialized");
    Object lookupKey = this.determineCurrentLookupKey();
    DataSource dataSource = (DataSource)this.resolvedDataSources.get(lookupKey);
    if (dataSource == null && (this.lenientFallback || lookupKey == null)) {
        dataSource = this.resolvedDefaultDataSource;
    }

    if (dataSource == null) {
        throw new IllegalStateException("Cannot determine target DataSource for lookup key [" + lookupKey + "]");
    } else {
        return dataSource;
    }
}

@Nullable
protected abstract Object determineCurrentLookupKey();
```





## 设计方案

基于上面的动态数据源的几个核心知识点，所以当我们需要实现动态数据源切换时，自然而然可以想到的一个方案就是在每次db执行之前，塞入这个希望使用的数据源key

![d82673ae-8d87-48ba-8fbc-288ebdcb237b (1)](../../../images/2023/assets/d82673ae-8d87-48ba-8fbc-288ebdcb237b (1).png)


如上的设计思路，主要借助aop的思想来实现，通过上下文来存储当前方法的执行，具体希望使用的数据源，然后再执行的sql的时候，AbstractRoutingDataSource直接从上下文中获取key，以此来抉择具体的数据源

基于上面的方案，接下来我们补充一下细节

### 1.数据源选择方式
通过AOP的方式进行拦截，写入数据源选择，这种方式适用于方法级别的粒度，基于此常见的实现方式就是通过自定义注解，来标注需要使用的数据源

如自定义注解，通过设置ds值来指定这个方法执行时选择的数据源

```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.TYPE})
public @interface DsAno {
    /**
     * 启用的数据源，默认主库
     *
     * @return
     */
    MasterSlaveDsEnum value() default MasterSlaveDsEnum.MASTER;

    /**
     * 启用的数据源，如果存在，则优先使用它来替换默认的value
     *
     * @return
     */
    String ds() default "";
}
```

### 2.上下文

通过上下文来保存当前选中的数据源，因此可以借助 ThreadLocal 来实现一个简单的数据源选择上下文，使用继承的线程上下文，支持异步时选择传递。

```
private static final ThreadLocal<String> CONTEXT_HOLDER = new InheritableThreadLocal<>();
```

### 3.动态数据源选择

```
public class MyRoutingDataSource extends AbstractRoutingDataSource {
    @Nullable
    @Override
    protected Object determineCurrentLookupKey() {
        return DsContextHolder.get();
    }

}
```

### 4.aop拦截

借助AOP，拦截方法、类上拥有DS注解的方法执行，然后从这个注解中读取选中的数据源，然后写入上下文；再方法执行完毕之后，清空上下文

```
@Aspect
public class DsAspect {

    /**
     * 切入点, 拦截类上、方法上有注解的方法，用于切换数据源
     */
    @Pointcut("@annotation(com.example.multidatasource.dal.DsAno)||@within(com.example.multidatasource.dal.DsAno)")
    public void pointcut() {
    }

    @Around("pointcut()")
    public Object around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        DsAno ds = getDsAno(proceedingJoinPoint);
        try {
            if (ds != null && (StringUtils.isNotBlank(ds.ds()) || ds.value() != null)) {
                // 当上下文中没有时，则写入线程上下文，应该用哪个DB
                DsContextHolder.set(StringUtils.isNoneBlank(ds.ds()) ? ds.ds() : ds.value().name());
            }
            return proceedingJoinPoint.proceed();
        } finally {
            // 清空上下文信息
            if (ds != null) {
                DsContextHolder.reset();
            }
        }
    }

    private DsAno getDsAno(ProceedingJoinPoint proceedingJoinPoint) {
        MethodSignature signature = (MethodSignature) proceedingJoinPoint.getSignature();
        Method method = signature.getMethod();
        DsAno ds = method.getAnnotation(DsAno.class);
        if (ds == null) {
            // 获取类上的注解
            ds = (DsAno) proceedingJoinPoint.getSignature().getDeclaringType().getAnnotation(DsAno.class);
        }
        return ds;
    }
}
```

## 多数据源实现
### 1.多数据源定义

对于多数据源，因此无法直接使用默认的数据源配置，我们借助默认的配置规则，加一个多数据源版本的设计对应的多数据源的配置规则如下

```yml
spring:
  dynamic:
    primary: master # 这个表示默认的数据源
    datasource:
      story:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://127.0.0.1:3306/${database.name}?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai
        username: root
        password: 666666
      shop:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://127.0.0.1:3306/shop?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai
        username: root
        password: 666666
```

注意上面的定义：

- spring.dynamic.primary: 指定默认启用的数据源
- spring.dynamic.datasource.数据源名.配置: 这里的数据源名不能重复（忽略大小写），数据源下的配置与默认的配置参数要求一致

对应的数据源配置加载方式直接借助了Spring的ConfigurationProperties来实现

### 2.多数据源注册

上面是数据源的配置定义与加载，但是我们需要基于上面的数据源配置来实例化对应的datasource。

```
@Slf4j
@Configuration
@ConditionalOnProperty(prefix = "spring.dynamic", name = "primary")
@EnableConfigurationProperties(DsProperties.class)
public class DataSourceConfig {

    public DataSourceConfig() {
        log.info("动态数据源初始化!");
    }

    @Bean
    public DsAspect dsAspect() {
        return new DsAspect();
    }

    /**
     * 整合主从数据源
     *
     * @param dsProperties
     * @return 1
     */
    @Bean
    @Primary
    public DataSource dataSource(DsProperties dsProperties) {
        Map<Object, Object> targetDataSources = Maps.newHashMapWithExpectedSize(dsProperties.getDatasource().size());
        dsProperties.getDatasource().forEach((k, v) -> targetDataSources.put(k.toUpperCase(), v.initializeDataSourceBuilder().build()));

        if (CollectionUtils.isEmpty(targetDataSources)) {
            throw new IllegalStateException("多数据源配置，请以 spring.dynamic 开头");
        }

        MyRoutingDataSource myRoutingDataSource = new MyRoutingDataSource();
        Object key = dsProperties.getPrimary().toUpperCase();
        if (!targetDataSources.containsKey(key)) {
            if (targetDataSources.containsKey(MasterSlaveDsEnum.MASTER.name())) {
                // 当们没有配置primary对应的数据源时，存在MASTER数据源，则将主库作为默认的数据源
                key = MasterSlaveDsEnum.MASTER.name();
            } else {
                key = targetDataSources.keySet().iterator().next();
            }
        }

        log.info("动态数据源，默认启用为： " + key);
        myRoutingDataSource.setDefaultTargetDataSource(targetDataSources.get(key));
        myRoutingDataSource.setTargetDataSources(targetDataSources);
        return myRoutingDataSource;
    }
}
```



- @ConditionalOnProperty: 这是Spring Boot框架提供的条件注解，它表示只有当指定的属性满足特定条件时，才会创建这个配置类所定义的bean。

- @EnableConfigurationProperties(DsProperties.class): 这个注解告诉Spring Boot去启用特定的配置属性类（DsProperties），使其可以被注入到其他bean中。

- @Primary: 这个注解用于标识在多个bean候选者中，优先选择被注解的bean作为自动装配的首选项。

- datasource实例化方式
  ```
  DataSourceProperties.initializeDataSourceBuilder().build()
  ```

  通过上面的方式创建的DataSource为默认的HikariDataSource

### 3.上下文定义

```
public class DsContextHolder {
    /**
     * 使用继承的线程上下文，支持异步时选择传递
     * 使用DsNode，支持链式的数据源切换，如最外层使用master数据源，内部某个方法使用slave数据源；但是请注意，对于事务的场景，不要交叉
     */
    private static final ThreadLocal<String> CONTEXT_HOLDER = new InheritableThreadLocal<>();

    private DsContextHolder() {
    }
    
    public static void set(String dbType) {
        CONTEXT_HOLDER.set(dbType);
    }

    public static String get() {
        return CONTEXT_HOLDER.get();
    }

    public static void set(DS ds) {
        set(ds.name().toUpperCase());
    }

    /**
     * 移除上下文
     */
    public static void reset() {
        CONTEXT_HOLDER.remove();
    }
}
```



### 4.AOP实现

借助AOP来简化数据源的选择，因此我们先定义一个注解DsAno，可以放在类上，表示这个类下所有的共有方法，都走某个数据源；也可以放在方法上，且方法上的优先级大于类上的注解

```
public interface DS {
    /**
     * 使用的数据源名
     *
     * @return
     */
    String name();
}
```



```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.TYPE})
public @interface DsAno {
    /**
     * 启用的数据源，默认主库
     *
     * @return
     */
    MasterSlaveDsEnum value() default MasterSlaveDsEnum.MASTER;

    /**
     * 启用的数据源，如果存在，则优先使用它来替换默认的value
     *
     * @return
     */
    String ds() default "";
}

```



```
@Aspect
public class DsAspect {
    
    /**
     * 切入点, 拦截类上、方法上有注解的方法，用于切换数据源
     */
    @Pointcut("@annotation(com.example.multidatasource.dal.DsAno)||@within(com.example.multidatasource.dal.DsAno)")
    public void pointcut() {
    }

    @Around("pointcut()")
    public Object around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        DsAno ds = getDsAno(proceedingJoinPoint);
        try {
            if (ds != null && (StringUtils.isNotBlank(ds.ds()) || ds.value() != null)) {
                // 当上下文中没有时，则写入线程上下文，应该用哪个DB
                DsContextHolder.set(StringUtils.isNoneBlank(ds.ds()) ? ds.ds() : ds.value().name());
            }
            return proceedingJoinPoint.proceed();
        } finally {
            // 清空上下文信息
            if (ds != null) {
                DsContextHolder.reset();
            }
        }
    }

    private DsAno getDsAno(ProceedingJoinPoint proceedingJoinPoint) {
        MethodSignature signature = (MethodSignature) proceedingJoinPoint.getSignature();
        Method method = signature.getMethod();
        DsAno ds = method.getAnnotation(DsAno.class);
        if (ds == null) {
            // 获取类上的注解
            ds = (DsAno) proceedingJoinPoint.getSignature().getDeclaringType().getAnnotation(DsAno.class);
        }
        return ds;
    }
}
```



上面的设计中，针对常见的主从切换做了一个简单的兼容，定义了一个主从数据源的枚举，同样也是基于简化使用体验的出发，枚举类设计如下：

```
public enum MasterSlaveDsEnum implements DS {
    MASTER, SHOP, STORY;
}
```

### 5.编程式数据源选择与小结

```
import java.util.function.Supplier;

/**
 * 手动指定数据源的用法
 */
public class DsSelectExecutor {

    /**
     * 有返回结果
     *
     * @param ds       数据源
     * @param supplier 供应商
     * @param <T>      泛型类型
     * @return 返回结果
     */
    public static <T> T submit(DS ds, Supplier<T> supplier) {
        DsContextHolder.set(ds);
        try {
            return supplier.get();
        } finally {
            DsContextHolder.reset();
        }
    }

    /**
     * 无返回结果
     *
     * @param ds   数据源
     * @param call 可调用对象
     */
    public static void execute(DS ds, Runnable call) {
        DsContextHolder.set(ds);
        try {
            call.run();
        } finally {
            DsContextHolder.reset();
        }
    }
}
```

- submit方法：接收一个数据库连接和一个Supplier接口实例作为参数，返回Supplier接口的返回值。该方法的目的是使用给定的数据库连接执行传入的Supplier接口实例，然后返回该接口的返回值。在方法中，首先通过DsContextHolder类将数据库连接设置到连接池中，然后使用supplier.get()方法获取Supplier接口的返回值。最后，在finally块中重置数据库连接池的状态，以确保数据库连接被正确释放。
- execute方法：接收一个数据库连接和一个Runnable接口实例作为参数，无返回值。该方法的目的是使用给定的数据库连接执行传入的Runnable接口实例中的代码。在方法中，首先通过DsContextHolder类将数据库连接设置到连接池中，然后使用call.run()方法执行Runnable接口的代码。最后，在finally块中重置数据库连接池的状态，以确保数据库连接被正确释放。

## 小结

小结一下整个的设计实现，主要基于AOP + 上下文的方式，配合AbstractRoutingDataSource来实现多数据源的自由选择

但是请注意，上面的实现，还有几个缺陷：

1. 如果一个方法上有数据源选择注解，但是内部开了子线程做异步处理，那么上下文存储的数据源会失效
2. 一个方法上指定了数据源，它又调用其他有也指定了数据源的方法，当前的方案会存在数据源的选择冲突
3. druid数据源怎么兼容？



## 优化

### 支持嵌套的数据源选择方案



前面提出的一个非常大的问题就是再整个链路中，只支持选择一次的数据源，显然这种方式缺陷比较大，也很容易出现问题；

若我们希望实现在选择了一个数据源之后，在执行某个代码片段时，还可以继续再选择其他的数据源，那么应该怎么实现呢？

首先我们来看一下之前的实现方案

![d82673ae-8d87-48ba-8fbc-288ebdcb237b (1)](../../../images/2023/assets/d82673ae-8d87-48ba-8fbc-288ebdcb237b (1).png)


在执行sql时，从上下文中获取当前选中的数据源,前面设计的上下文直接使用线程上线文保存

```
private static final ThreadLocal<String> CONTEXT_HOLDER = new InheritableThreadLocal<>();
```

从上面的这个实现来看，上下文赋值只能有一次，当我们希望可以实现标题的效果，自然我们应该想到的数据结构就是栈 -- 后进先出

我们定义一个DsNode节点来存储选中的数据源，区别于之前的String，它除了记录选中的数据源之外，还记录前一次选中的数据源

```
public static class DsNode {
    DsNode pre;
    String ds;

    public DsNode(DsNode parent, String ds) {
        pre = parent;
        this.ds = ds;
    }
}
```

移除上下文则不能像之前那么粗暴，而是要实现栈的弹出效果

- 如果无选中的数据源，直接返回

- 若之前选中的DsNode，存在上一个节点，则将上一个节点重新写入上下文

- 若之前选中的DsNode，不存在上一个节点，则清空上下文

  ```
      public static String get() {
          DsNode dsNode = CONTEXT_HOLDER.get();
          log.info("get dbType:{}", CONTEXT_HOLDER.get());
          return dsNode==null? null:dsNode.ds;
      }
  
      public static void set(String dbType) {
          DsNode current = CONTEXT_HOLDER.get();
          CONTEXT_HOLDER.set(new DsNode(current,dbType));
      }
  
      public static void set(DS ds) {
  
          set(ds.name().toUpperCase());
      }
  
      /**
       * 移除上下文
       */
      public static void reset() {
          DsNode ds = CONTEXT_HOLDER.get();
          if (ds == null) {
              return;
          }
  
          if (ds.pre != null) {
              // 退出当前的数据源选择，切回去走上一次的数据源配置
              CONTEXT_HOLDER.set(ds.pre);
          } else {
              CONTEXT_HOLDER.remove();
          }
      }
  
  ```

### Druid数据源支持

```
    dsProperties.getDatasource().forEach((k, v) -> targetDataSources.put(k.toUpperCase(), initDataSource(k, v)));
```

上面代码中使用`v.initializeDataSourceBuilder().build()`初始化的数据源默认是`HikarDataSource`,当我们使用Druid数据源时就无法适配，因此需要做以下修改。

1.检查是否引入Druid相关资源

```
public class DruidCheckUtil {

    /**
     * 判断是否包含durid相关的数据包
     *
     * @return
     */
    public static boolean hasDuridPkg() {
        return ClassUtils.isPresent("com.alibaba.druid.pool.DruidDataSource", DataSourceConfig.class.getClassLoader());
    }

}
```

2.创建InitDatasourse方法用来判断是否初始化Druid数据源

```
   /**
     * 初始化数据源
     * @param prefix 数据源前缀
     * @param properties 数据源属性
     * @return 数据源
     */
    public DataSource initDataSource(String prefix, DataSourceProperties properties) {
        // 检查是否包含Druid包
        if (!DruidCheckUtil.hasDuridPkg()) {
            log.info("实例化HikarDataSource: {}", prefix);
            // 使用数据源属性初始化数据源构建器并构建数据源
            return properties.initializeDataSourceBuilder().build();
        }

        // 如果数据源类型为空或者数据源类型不是DruidDataSource类
        if (properties.getType() == null || !properties.getType().isAssignableFrom(DruidDataSource.class)) {
            log.info("实例化HikarDataSource: {}", prefix);
            // 使用数据源属性初始化数据源构建器并构建数据源
            return properties.initializeDataSourceBuilder().build();
        }

        log.info("实例化DruidDataSource: {}", prefix);
        // 使用Spring Binder 绑定或创建一个 "spring.dynamic.datasource." + prefix 的DruidDataSource对象
        return Binder.get(environment).bindOrCreate("spring.dynamic.datasource." + prefix, DruidDataSource.class);
    }
```

上面方法会读取spring.dynamic.datasource.xxx的配置信息，我们只需添加type属性和Druid的配置信息即可初始化DruidDataSource。



配置文件如下：

```
# 默认的数据库名
database:
  name: story

spring:
  dynamic:
    primary: master # 这个表示默认的数据源
    datasource:
      master:
        # 数据库名，从配置 database.name 中获取
        url: jdbc:mysql://127.0.0.1:3306/${database.name}?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai
        username: root
        password: 666666
        # 注意下面这个type,选择DruidDataSource时，请确保项目中添加了druid相关依赖
        type: com.alibaba.druid.pool.DruidDataSource
        #DruidDataSource自有属性
        filters: stat
        initialSize: 0
        minIdle: 1
        maxActive: 200
        maxWait: 10000
        time-between-eviction-runs-millis: 60000
        min-evictable-idle-time-millis: 200000
        testWhileIdle: true
        testOnBorrow: true
        validationQuery: select 1
      story:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://127.0.0.1:3306/${database.name}?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai
        username: root
        password: 666666
      shop:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://127.0.0.1:3306/shop?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai
        username: root
        password: 666666
      blog:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://127.0.0.1:3306/sg_blog?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai
        username: root
        password: 666666
```


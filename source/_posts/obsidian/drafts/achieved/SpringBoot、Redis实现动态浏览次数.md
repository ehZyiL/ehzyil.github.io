# SpringBoot、Redis实现动态浏览次数

## 1. 项目-思路分析

在用户浏览博文时要实现对应博客浏览量的增加。我们只需要在每次用户浏览博客时更新对应的浏览数即可

但是如果直接操作博客表的浏览量的话，在并发量大的情况下会出现什么问题呢？如何去优化呢？如下四点

①在应用启动时把博客的浏览量存储到redis中 - 项目启动的预处理功能

②更新浏览量时去更新redis中的数据

③每隔3分钟把Redis中的浏览量更新到数据库中 - 定时任务功能

④读取文章浏览量时从redis读取

在实现 '浏览次数' 功能之前，我们先学习一下必要的基础知识，如下

## 2. 基础-启动预处理

如果希望在SpringBoot应用启动时进行一些初始化操作可以选择使用CommandLineRunner接口来进行处理。

我们只需要实现CommandLineRunner接口，并且把对应的bean注入容器。把相关初始化的代码重新到需要重新的方法中。

这样就会在应用启动的时候执行对应的代码。

第一步: 在ehzyil-blog工程的src/main/java/com.ehzyil目录新建runner.myCommandLineRunner类，写入如下

```java
package com.ehzyil.runner;

import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
//项目启动时，该类负责预处理一些代码。CommandLineRunner是spring提供的接口
public class myCommandLineRunner implements CommandLineRunner {
    
    @Override
    public void run(String... args) throws Exception {
        System.out.println("程序初始化");
    }
}
```

第二步: 运行ehzyil-blog工程的BlogApplication类，查看控制台。只是了解基础知识，跟项目暂时无关

## 3. 基础-定时任务

定时任务的实现方式有很多，比如XXL-Job等。但是其实核心功能和概念都是类似的，很多情况下只是调用的API不同而已

这里就先用SpringBoot为我们提供的定时任务的API来实现一个简单的定时任务，先对定时任务里面的一些核心概念有个大致的了解

第一步: 在ehzyil-blog工程的BlogApplication类添加如下，作用是在配置类添加@EnableScheduling注解，就能开启定时任务功能

```java
package com.ehzyil;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@MapperScan("com.ehzyil.mapper")
@EnableScheduling//@EnableScheduling是spring提供的定时任务的注解
public class BlogApplication {

    public static void main(String[] args) {
        SpringApplication.run(BlogApplication.class,args);
    }
}
```

第二步: 在src/main/java/com.ehzyil目录新建cronjob.MyJob类，写入如下

```
package com.ehzyil.cronjob;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import java.time.LocalTime;

@Component//当前这个类要作为bean，且注入容器，否则不会生效
//定时任务
public class MyJob {

    @Scheduled(cron = "0/5 * * * * ?")//在哪个方法添加了@Scheduled注解，哪个方法就会定时去执行
    //上面那行@Scheduled注解的cron属性就是具体的定时规则。从每一分钟的0秒开始，每隔5秒钟就会执行下面那行的xxJob()方法
    public void xxJob(){
        //要定时执行的代码
        System.out.println("定时任务执行了，现在的时间是: "+ LocalTime.now());
    }
}
```

第三步: 运行ehzyil-blog工程的BlogApplication类，查看控制台。只是了解基础知识，跟项目暂时无关

## 4. 基础-cron表达式

```
常用表达式例子

  （1）0/2 * * * * ?   表示每2秒 执行任务

  （1）0 0/2 * * * ?    表示每2分钟 执行任务

  （1）0 0 2 1 * ?   表示在每月的1日的凌晨2点调整任务

  （2）0 15 10 ? * MON-FRI   表示周一到周五每天上午10:15执行作业

  （3）0 15 10 ? 6L 2002-2006   表示2002-2006年的每个月的最后一个星期五上午10:15执行作

  （4）0 0 10,14,16 * * ?   每天上午10点，下午2点，4点 

  （5）0 0/30 9-17 * * ?   朝九晚五工作时间内每半小时 

  （6）0 0 12 ? * WED    表示每个星期三中午12点 

  （7）0 0 12 * * ?   每天中午12点触发 

  （8）0 15 10 ? * *    每天上午10:15触发 

  （9）0 15 10 * * ?     每天上午10:15触发 

  （10）0 15 10 * * ?    每天上午10:15触发 

  （11）0 15 10 * * ? 2005    2005年的每天上午10:15触发 

  （12）0 * 14 * * ?     在每天下午2点到下午2:59期间的每1分钟触发 

  （13）0 0/5 14 * * ?    在每天下午2点到下午2:55期间的每5分钟触发 

  （14）0 0/5 14,18 * * ?     在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发 

  （15）0 0-5 14 * * ?    在每天下午2点到下午2:05期间的每1分钟触发 

  （16）0 10,44 14 ? 3 WED    每年三月的星期三的下午2:10和2:44触发 

  （17）0 15 10 ? * MON-FRI    周一至周五的上午10:15触发 

  （18）0 15 10 15 * ?    每月15日上午10:15触发 

  （19）0 15 10 L * ?    每月最后一日的上午10:15触发 

  （20）0 15 10 ? * 6L    每月的最后一个星期五上午10:15触发 

  （21）0 15 10 ? * 6L 2002-2005   2002年至2005年的每月的最后一个星期五上午10:15触发 

  （22）0 15 10 ? * 6#3   每月的第三个星期五上午10:15触发
```

在刚刚的定时任务，spring提供的@Scheduled注解里面的cron属性，是控制定时规则的。我们下面有必要学习一下cron属性的基本语法、特色符号

cron表达式是用来设置定时任务执行时间的表达式。

不会写的话，去这个cron表达式生成器网站 -> https://www.bejson.com/othertools/cron/

cron表达式由七部分组成，中间由空格分隔，这七部分从左往右依次是:

```
秒(0~59)，分钟(0~59)，小时(0~23)，日期(1-月最后一天)，月份(1-12)，星期几(1-7,1表示星期日)，年份(可不写)
```

(1) *星号用来代表任意值，如下表示 "每年每月每天每时每分每秒"

```
* * * * * ?
```

(2) ,逗号可以用来定义列表，如下表示 "每年每月每天每时每分的每个第1秒，第2秒，第3秒"

```
1,2,3 * * * * ?
```

(3) -短斜杠用来定义范围，如下表示 "每年每月每天每时每分的第1秒至第3秒"

```
1-3 * * * * ?
```

(4) /正斜杠用来代表每隔多少，如下表示 "每年每月每天每时每分，从第5秒开始，每10秒一次 "。正斜杠的左侧是开始值(0可以省略不写)，右侧是间隔

```
5/10 * * * * ?
```

日期部分还可允许特殊字符： ? L W

星期部分还可允许的特殊字符: ? L #

(1) ?问号只能用在日期和星期部分。表示没有具体的值，使用?要注意冲突。日期和星期两个部分如果其中一个部分设置了值，则另一个必须设置为 " ? "

【正确写法】

```
0\* * * 2 * ?    每个月的第2天，每分钟都执行任务
0\* * * ? * 2    每周的星期二，每分钟都执行任务
```

【错误写法-同时使用?和同时不使用?都是不对的】

```
* * * 2 * 2
* * * ? * ?
```

(2) W字母只能用在日期中，表示当月中最接近某天的工作日

```
0 0 0 31W * ?
```

表示最接近31号的工作日，如果31号是星期六，则表示30号，即星期五，如果31号是星期天，则表示29号，即星期五。如果31号是星期三，则表示31号本身，即星期三

(3) L字母只能用在日期和星期中，表示最后。在日期中表示每月最后一天，在一月份中表示31号，在六月份中表示30号

也可以表示每月倒是第N天。例如： L-2表示每个月的倒数第2天

在星期中表示7即星期六

```
0 0 0 ? * L
表示每个星期六
0 0 0 ? * 6L
若前面有其他值的话，则表示最后一个星期几，即每月的最后一个星期五
```

(4) LW可以连起来用，表示每月最后一个工作日，即每月最后一个星期五，如下

```
0 0 0 LW * ?
```

(5) #字符只能用在星期中，表示第几个星期几

```
0 0 0 ? * 6#3
表示每个月的第三个星期五。
```

## 5. 项目-接口分析

在用户浏览博文时要实现对应博客浏览量的增加。我们只需要在每次用户浏览博客时更新对应的浏览数即可

| 请求方式 | 请求地址                      | 请求头            |
| -------- | ----------------------------- | ----------------- |
| PUT      | /article/updateViewCount/{id} | 不需要token请求头 |

参数: 请求路径中携带文章id

响应格式:

```
{
	"code":200,
	"msg":"操作成功"
}
```

## 6. 项目-代码实现

下面是 '浏览次数' 功能的具体实现。

一、启动预处理

当项目启动时，就把博客的浏览量(id和viewCount字段)以map的形式存储到redis中

第一步: 在ehzyil-blog工程的src/main/java/com.ehzyil目录新建runner.ViewCountRunner类，写入如下

```java
package com.ehzyil.runner;

import com.ehzyil.domain.Article;
import com.ehzyil.mapper.ArticleMapper;
import com.ehzyil.utils.RedisCache;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;

@Component
//当项目启动时，就把博客的浏览量(id和viewCount字段)存储到redis中。也叫启动预处理。CommandLineRunner是spring提供的接口
public class ViewCountRunner implements CommandLineRunner {

    @Autowired
    //用于操作redis
    private RedisCache redisCache;

    @Autowired
    //用于操作数据库的article表
    private ArticleMapper articleMapper;

    //@Override
    ////下面的写法是stream流+函数式编程
    //public void run(String... args) throws Exception {
    //    //查询数据库中的博客信息，注意只需要查询id、viewCount字段的博客信息
    //    List<Article> articles = articleMapper.selectList(null);//为null即无条件，表示查询所有
    //    articles.stream()
    //            //由于我们需要key、value的数据，所有可以通过stream流，转为map集合。new Function表示函数式编程
    //            .collect(Collectors.toMap(new Function<Article, Long>() {
    //                @Override
    //                public Long apply(Article article) {
    //                    return article.getId();
    //                }
    //            }, new Function<Article, Integer>() {//new Function表示函数式编程
    //                @Override
    //                public Integer apply(Article article) {
    //                    return article.getViewCount().intValue();
    //                }
    //            }));


    @Override
    //下面的写法是stream流+方法引用+Lambda。我们用这种写法使得代码更简短，跟上面那种写法的效果是一样的
    public void run(String... args) throws Exception {
        //查询数据库中的博客信息，注意只需要查询id、viewCount字段的博客信息
        List<Article> articles = articleMapper.selectList(null);//为null即无条件，表示查询所有
        Map<String, Integer> viewCountMap = articles.stream()
                //由于我们需要key、value的数据，所有可以通过stream流

                //下面toMap方法的第一个参数用了方法引用，第二个参数用了Lambda
                //.collect(Collectors.toMap(Article::getId, article -> article.getViewCount().intValue()));

                //由于上面那行Article::getId返回值是Long类型，而我们需要String类型，为了方便转换类型，我们要写成Lambda表达式
                .collect(Collectors.toMap(article -> article.getId().toString(), article -> article.getViewCount().intValue()));


        //把查询到的id作为key，viewCount作为value，一起存入Redis
        redisCache.setCacheMap("article:viewCount",viewCountMap);
    }
}
```

第三步: 测试。运行ehzyil-blog工程的BlogApplication类。然后去redis看一下数据

### 二、浏览量递增

刚刚我们实现了把不同id的文章，和对应的viewCount浏览量存入了redis。接下来就实现如何让用户每从mysql根据文章id查询一次浏览量，那么redis里这篇文章的浏览量就增加1

第一步: 在ehzyil-framework工程的RedisCache工具类添加如下，是redis的浏览量增加1的核心代码

```
/**
  * 对redis中，某个hash结构里面的value进行递增操作
  *
  * @param key 操作的是哪个hash结构
  * @param hKey 对hash结构里面的哪个key进行操作
  * @param v key对应的value值会递增多少
  */
public void incrementCacheMapValue(String key,String hKey,long v){
	redisTemplate.opsForHash().increment(key, hKey, v);
}
```

第二步: 在ehzyil-framework工程的ArticleService接口修改为如下，增加了根据id从mysql查询文章的接口

```
package com.ehzyil.service;

import com.baomidou.mybatisplus.extension.service.IService;
import com.ehzyil.domain.Article;
import com.ehzyil.domain.ResponseResult;

public interface ArticleService extends IService<Article> {

    //文章列表
    ResponseResult hotArticleList();

    //分类查询文章列表
    ResponseResult articleList(Integer pageNum, Integer pageSize, Long categoryId);

    //根据id查询文章详情
    ResponseResult getArticleDetail(Long id);

    //根据文章id从mysql查询文章
    ResponseResult updateViewCount(Long id);
}
```

第三步: 在ehzyil-framework工程的ArticleServiceImpl类添加如下，作用是根据id从mysql查询文章，并让redis里对应文章的浏览量+1

```
@Autowired
private RedisCache redisCache;

@Override
public ResponseResult updateViewCount(Long id) {
	//更新redis中的浏览量，对应文章id的viewCount浏览量。article:viewCount是ViewCountRunner类里面写的
	//用户每从mysql根据文章id查询一次浏览量，那么redis的浏览量就增加1
	redisCache.incrementCacheMapValue("article:viewCount",id.toString(),1);
	return ResponseResult.okResult();
}	
```

第四步: 在ehzyil-blog工程的ArticleController类添加如下，作用是让用户访问从mysql根据文章id查询文章浏览量的接口

```
@PutMapping("/updateViewCount/{id}")
@mySystemlog(businessName = "根据文章id从mysql查询文章")//接口描述，用于'日志记录'功能
public ResponseResult updateViewCount(@PathVariable("id") Long id){
	return articleService.updateViewCount(id);
}
```

第六步: 测试。运行ehzyil-blog工程的BlogApplication类。然后启动vue前端项目，随便浏览一篇文章，最后去redis看一下数据

 三、数据同步 

把redis的数据同步到mysql数据库。避免redis宕机，丢失浏览量数据。要求通过定时任务实现每隔3分钟把redis中的浏览量更新到mysql数据库中

第一步: 把ehzyil-framework工程的Article类，修改为如下，增加了Article()构造方法

```java
package com.ehzyil.domain;

import java.util.Date;

import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
@TableName("sg_article")
public class Article {

    @TableId
    private Long id;
    //标题
    private String title;
    //文章内容
    private String content;
    //文章摘要
    private String summary;
    //所属分类id
    private Long categoryId;

    //增加一个字段，为categoryName，由categoryId来查询出
    //由于数据库没有category_name字段，所以要用注解指定一下字段
    @TableField(exist = false)//代表这个字段在数据库中不存在，避免MyBatisPlus在查询时报错
    private String categoryName;

    //缩略图
    private String thumbnail;
    //是否置顶（0否，1是）
    private String isTop;
    //状态（0已发布，1草稿）
    private String status;
    //访问量
    private Long viewCount;
    //是否允许评论 1是，0否
    private String isComment;

    private Long createBy;

    private Date createTime;

    private Long updateBy;

    private Date updateTime;
    //删除标志（0代表未删除，1代表已删除）
    private Integer delFlag;

    //把redis的浏览量数据更新到mysql数据库
    public Article(Long id, long viewCount) {
        this.id = id;
        this.viewCount=viewCount;
    }
}
```

第二步: 在src/main/java/com.ehzyil目录新建cronjob.UpdateViewCount类，写入如下

```java
package com.ehzyil.cronjob;

import com.ehzyil.domain.Article;
import com.ehzyil.service.ArticleService;
import com.ehzyil.utils.RedisCache;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalTime;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@Component
//通过定时任务实现每隔3分钟把redis中的浏览量更新到mysql数据库中
public class UpdateViewCount {

    @Autowired
    //操作redis。RedisCache是我们在ehzyil-framework工程写的工具类
    private RedisCache redisCache;

    @Autowired
    //操作数据库。ArticleService是我们在ehzyil-framework工程写的接口
    private ArticleService articleService;

    //每隔3分钟，把redis的浏览量数据更新到mysql数据库
    @Scheduled(cron = "0/10 * * * * ?")
    public void updateViewCount(){
        //获取redis中的浏览量，注意得到的viewCountMap是HashMap双列集合
        Map<String, Integer> viewCountMap = redisCache.getCacheMap("article:viewCount");
        //让双列集合调用entrySet方法即可转为单列集合，然后才能调用stream方法
        List<Article> articles = viewCountMap.entrySet()
                .stream()
                .map(entry -> new Article(Long.valueOf(entry.getKey()), entry.getValue().longValue()))
                //把最终数据转为List集合
                .collect(Collectors.toList());
        //把获取到的浏览量更新到mysql数据库中。updateBatchById是mybatisplus提供的批量操作数据的接口
        articleService.updateBatchById(articles);
        //方便在控制台看打印信息
        System.out.println("redis的文章浏览量数据已更新到数据库，现在的时间是: "+ LocalTime.now());
    }
}
```

第四步: 测试。运行ehzyil-blog工程的BlogApplication类。启动vue前端项目，随便浏览一篇文章，3分钟后去看redis和mysql的浏览量数据是否一致

### 四、从redis查询

前面我们实现了根据id去数据库查询文章浏览量，同时用户每查一次文章，那么redis的浏览量就递增1。每隔3分钟，redis的浏览量就会同步到数据库。但是还存在一个问题，由于redis每隔3分钟才把数据同步到数据库，那么用户在页面看到的浏览量是每隔3分钟才有变化，我们希望浏览量是实时更新的

解决: 用户每次根据文章id查询文章浏览量时，都是直接查数据库，我们希望用户从redis里面查询文章的浏览量

第一步: 把ehzyil-framework工程的ArticleServiceImpl类修改为如下，修改了hotArticleList、hotArticleList方法，让用户查询文章浏览量时从redis查

```java
package com.ehzyil.service.impl;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.ehzyil.constants.SystemCanstants;
import com.ehzyil.domain.Article;
import com.ehzyil.domain.Category;
import com.ehzyil.domain.ResponseResult;
import com.ehzyil.mapper.ArticleMapper;
import com.ehzyil.service.ArticleService;
import com.ehzyil.service.CategoryService;
import com.ehzyil.utils.BeanCopyUtils;
import com.ehzyil.utils.RedisCache;
import com.ehzyil.vo.ArticleDetailVo;
import com.ehzyil.vo.ArticleListVo;
import com.ehzyil.vo.HotArticleVO;
import com.ehzyil.vo.PageVo;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Objects;
import java.util.function.Function;
import java.util.stream.Collectors;


@Service
//ServiceImpl是mybatisPlus官方提供的
public class ArticleServiceImpl extends ServiceImpl<ArticleMapper, Article> implements ArticleService {

    @Autowired
    //操作数据库。ArticleService是我们在ehzyil-framework工程写的接口
    private ArticleService articleService;


    @Override
    public ResponseResult hotArticleList() {

        //-------------------每调用这个方法就从redis查询文章的浏览量，展示在热门文章列表------------------------
        
        //获取redis中的浏览量，注意得到的viewCountMap是HashMap双列集合
        Map<String, Integer> viewCountMap = redisCache.getCacheMap("article:viewCount");
        //让双列集合调用entrySet方法即可转为单列集合，然后才能调用stream方法
        List<Article> xxarticles = viewCountMap.entrySet()
                .stream()
                .map(entry -> new Article(Long.valueOf(entry.getKey()), entry.getValue().longValue()))
                //把最终数据转为List集合
                .collect(Collectors.toList());
        //把获取到的浏览量更新到mysql数据库中。updateBatchById是mybatisplus提供的批量操作数据的接口
        articleService.updateBatchById(xxarticles);
        
        //-----------------------------------------------------------------------------------------

        //查询热门文章，封装成ResponseResult返回。把所有查询条件写在queryWrapper里面
        LambdaQueryWrapper<Article> queryWrapper = new LambdaQueryWrapper<>();

        //查询的不能是草稿。也就是Status字段不能是0
        queryWrapper.eq(Article::getStatus, SystemCanstants.ARTICLE_STATUS_NORMAL);
        //按照浏览量进行排序。也就是根据ViewCount字段降序排序
        queryWrapper.orderByDesc(Article::getViewCount);
        //最多只能查询出来10条消息。当前显示第一页的数据，每页显示10条数据
        Page<Article> page = new Page<>(SystemCanstants.ARTICLE_STATUS_CURRENT,SystemCanstants.ARTICLE_STATUS_SIZE);
        page(page,queryWrapper);

        //获取最终的查询结果，把结果封装在Article实体类里面会有很多不需要的字段
        List<Article> articles = page.getRecords();

        //解决: 把结果封装在HotArticleVo实体类里面，在HotArticleVo实体类只写我们要的字段
        //List<HotArticleVO> articleVos = new ArrayList<>();
        //for (Article xxarticle : articles) {
        //    HotArticleVO xxvo = new HotArticleVO();
        //    //使用spring提供的BeanUtils类，来实现bean拷贝。第一个参数是源数据，第二个参数是目标数据，把源数据拷贝给目标数据
        //    BeanUtils.copyProperties(xxarticle,xxvo); //xxarticle就是Article实体类，xxvo就是HotArticleVo实体类
        //    //把我们要的数据封装成集合
        //    articleVos.add(xxvo);
        //}
        //注释掉，使用我们定义的BeanCopyUtils工具类的copyBeanList方法。如下

        //一行搞定
        List<HotArticleVO> articleVos = BeanCopyUtils.copyBeanList(articles, HotArticleVO.class);

        return ResponseResult.okResult(articleVos);
    }

    //----------------------------------分页查询文章的列表---------------------------------

    @Autowired
    //注入我们写的CategoryService接口
    private CategoryService categoryService;

    @Override
    public ResponseResult articleList(Integer pageNum, Integer pageSize, Long categoryId) {

        LambdaQueryWrapper<Article> lambdaQueryWrapper = new LambdaQueryWrapper<>();

        //判空。如果前端传了categoryId这个参数，那么查询时要和传入的相同。第二个参数是数据表的文章id，第三个字段是前端传来的文章id
        lambdaQueryWrapper.eq(Objects.nonNull(categoryId)&&categoryId>0,Article::getCategoryId,categoryId);

        //只查询状态是正式发布的文章。Article实体类的status字段跟0作比较，一样就表示是正式发布的
        lambdaQueryWrapper.eq(Article::getStatus,SystemCanstants.ARTICLE_STATUS_NORMAL);

        //对isTop字段进行降序排序，实现置顶的文章(isTop值为1)在最前面
        lambdaQueryWrapper.orderByDesc(Article::getIsTop);

        //分页查询
        Page<Article> page = new Page<>(pageNum,pageSize);
        page(page,lambdaQueryWrapper);

        /**
         * 解决categoryName字段没有返回值的问题。在分页之后，封装成ArticleListVo之前，进行处理。第一种方式，用for循环遍历的方式
         */
        ////用categoryId来查询categoryName(category表的name字段)，也就是查询'分类名称'
        //List<Article> articles = page.getRecords();
        //for (Article article : articles) {
        //    //'article.getCategoryId()'表示从article表获取category_id字段，然后作为查询category表的name字段
        //    Category category = categoryService.getById(article.getCategoryId());
        //    //把查询出来的category表的name字段值，也就是article，设置给Article实体类的categoryName成员变量
        //    article.setCategoryName(category.getName());
        //
        //}

        /**
         * 解决categoryName字段没有返回值的问题。在分页之后，封装成ArticleListVo之前，进行处理。第二种方式，用stream流的方式
         */
        //用categoryId来查询categoryName(category表的name字段)，也就是查询'分类名称'
        List<Article> articles = page.getRecords();

        articles.stream()
                .map(new Function<Article, Article>() {
                    @Override
                    public Article apply(Article article) {
                        //'article.getCategoryId()'表示从article表获取category_id字段，然后作为查询category表的name字段
                        Category category = categoryService.getById(article.getCategoryId());
                        String name = category.getName();
                        //把查询出来的category表的name字段值，也就是article，设置给Article实体类的categoryName成员变量
                        article.setCategoryName(name);
                        //把查询出来的category表的name字段值，也就是article，设置给Article实体类的categoryName成员变量
                        return article;
                    }
                })
                .collect(Collectors.toList());


        //把最后的查询结果封装成ArticleListVo(我们写的实体类)。BeanCopyUtils是我们写的工具类
        List<ArticleListVo> articleListVos = BeanCopyUtils.copyBeanList(page.getRecords(), ArticleListVo.class);


        //把上面那行的查询结果和文章总数封装在PageVo(我们写的实体类)
        PageVo pageVo = new PageVo(articleListVos,page.getTotal());
        return ResponseResult.okResult(pageVo);
    }

    //---------------------------------根据id查询文章详情----------------------------------

    @Override
    public ResponseResult getArticleDetail(Long id) {

        //根据id查询文章
        Article article = getById(id);

        //-------------------从redis查询文章的浏览量，展示在文章详情---------------------------
        
        //从redis查询文章浏览量
        Integer viewCount = redisCache.getCacheMapValue("article:viewCount", id.toString());
        article.setViewCount(viewCount.longValue());
        
        //-----------------------------------------------------------------------------

        //把最后的查询结果封装成ArticleListVo(我们写的实体类)。BeanCopyUtils是我们写的工具类
        ArticleDetailVo articleDetailVo = BeanCopyUtils.copyBean(article, ArticleDetailVo.class);

        //根据分类id，来查询分类名
        Long categoryId = articleDetailVo.getCategoryId();
        Category category = categoryService.getById(categoryId);
        //如果根据分类id查询的到分类名，那么就把查询到的值设置给ArticleDetailVo实体类的categoryName字段
        if(category!=null){
            articleDetailVo.setCategoryName(category.getName());
        }

        //封装响应返回。ResponseResult是我们在ehzyil-framework工程的domain目录写的实体类
        return ResponseResult.okResult(articleDetailVo);
    }

    //--------------------------------根据文章id从mysql查询文章----------------------------

    @Autowired
    private RedisCache redisCache;

    @Override
    public ResponseResult updateViewCount(Long id) {
        //更新redis中的浏览量，对应文章id的viewCount浏览量。article:viewCount是ViewCountRunner类里面写的
        //用户每从mysql根据文章id查询一次浏览量，那么redis的浏览量就增加1
        redisCache.incrementCacheMapValue("article:viewCount",id.toString(),1);
        return ResponseResult.okResult();
    }
}
```

第四步: 测试。运行ehzyil-blog工程的BlogApplication类。启动vue前端项目，浏览文章，看一下前端展示的浏览量是否实时
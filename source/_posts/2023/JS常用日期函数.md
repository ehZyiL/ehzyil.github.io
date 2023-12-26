---
title: JS常用日期函数
author: ehzyil
tags:
  - Java Script
categories:
  - 技术
date: 2023-12-26

---

## JS常用日期函数



```
 
 //获取完整的日期
 var date=new Date;
 var year=date.getFullYear(); 
 var month=date.getMonth()+1;
 month =(month<10 ? "0"+month:month); 
 var mydate = (year.toString()+month.toString());
注意，year.toString()+month.toString()不能写成year+month。不然如果月份大于等于10，则月份为数字，会和年份相加，如201210，则会变为2022，需要加.toString()
以下是搜到的有用内容：
var myDate = new Date();
myDate.getYear(); //获取当前年份(2位)
myDate.getFullYear(); //获取完整的年份(4位,1970-????)
myDate.getMonth(); //获取当前月份(0-11,0代表1月)
myDate.getDate(); //获取当前日(1-31)
myDate.getDay(); //获取当前星期X(0-6,0代表星期天)
myDate.getTime(); //获取当前时间(从1970.1.1开始的毫秒数)
myDate.getHours(); //获取当前小时数(0-23)
myDate.getMinutes(); //获取当前分钟数(0-59)
myDate.getSeconds(); //获取当前秒数(0-59)
myDate.getMilliseconds(); //获取当前毫秒数(0-999)
myDate.toLocaleDateString(); //获取当前日期
var mytime=myDate.toLocaleTimeString(); //获取当前时间
myDate.toLocaleString( ); //获取日期与时间
<script language="JavaScript">function monthnow(){
 var now   = new Date();
 var monthn = now.getMonth();
 var yearn  = now.getYear();
 window.location.href="winnNamelist.jsp?getMonth="+monthn+"&getYear="+yearn;
}</script>
 
 
```


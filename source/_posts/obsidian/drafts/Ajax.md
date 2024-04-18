0## 一.什么是AJAX

> - ①AJAX, 即Asynchronous JavaScript and XML(异步的 JavaScript 和 XML).
> - ②AJAX 不是新的编程语言, 而是一种使用现有标准的新方法或技术.
> - ③AJAX 是与服务器交换数据并更新部分网页的技术, 在不重新加载整个页面的情况下.

现在, 凡是允许浏览器与服务器通信而无须刷新当前页面的技术都被称作 Ajax.  
Ajax最早是由Google使用的.

### 1.如何实现Ajax

- Flash
- Java applet
- 框架: 如果使用一组框架构造了一个网页, 可以只更新其中一个框架, 而不必惊动整个页面
- 隐藏的iframe
- XMLHttpRequest: 该对象是对JS的一个扩展, 可以使网页与服务器进行通信. 这是创建Ajax应用的最佳选择

实际上, 通常把Ajax当成XMLHttpRequest

### 2.Ajax原理

![ajax](https://cdn.jsdelivr.net/gh/crainyday/blog@master/imgs/JS/ajax.png)

### 3.Ajax工具包

> Ajax并不是一项新技术, 它实际上是几种技术, 每种技术各尽其职, 以一种全新的方式聚合在一起.

- 服务端语言: 服务器需具备向浏览器发送特定信息的能力. Ajax与服务器端语言无关
- XML(eXtensible Markup Language, 可扩展标记语言)是一种描述数据的格式. Ajax程序需要某种格式化的格式来在服务器和客户端之间传递消息, XML是其中一种选择.
- XHTML(eXtensible HyperText Markup Language,可扩展超文本标记语言)(现在是HTML5.0)和CSS(Cascading Style Sheets,层叠样式表)标准化呈现在前端
- DOM(Document Object Model,文档对象模型)实现动态的显示和交互
- 使用XMLHttpRequest对象进行异步数据读取
- 使用JS绑定和处理数据

### 4.Ajax的缺陷

- 由JS和Ajax引擎导致的浏览器的兼容问题
- 页面局部刷新, 导致后退等功能失效
- 对流媒体的支持没有Flash、Java applet好
- 一些手持设备(如手机等)支持性差

## 二.XMLHttpRequest

> - XMLHttpRequest最早在IE5中以ActiveX组建形式出现, 非W3C标准
> - 创建XMLHttpRequest对象的方法不一样: IE5将其实现为一个ActiveX对象, 其他浏览器实现为一个本地JS对象
> - 虽然在不同浏览器上的实现方法是兼容的, 但是XMLHttpRequest实例的属性和方法是相同的.

### 1.创建方式

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9|COPY<br><br>function getHTTPObject(){  <br>    var xhr = false;  <br>    if(window.XMLHttpRequset){  <br>        xhr = new XMLHttpRequest();  <br>    }else if(window.ActiveXObject){  <br>        xhr = new ActiveXObject("Microsoft.XMLHTTP");  <br>    }  <br>    return xhr;  <br>}|

### 2.XMLHttpRequest的方法和属性

|方法|描述|
|---|---|
|open(“method”,“url”)|建立对服务器的调用.Method参数可以是GET、POST或PUT.URL参数可以是相对URL或绝对URL|
|send(content)|向服务器发送请求|
|setRequestHeader(“header”,“value”)|把指定首部设置为所提供的值, 在设置任何首部之前必须调用open()|
|abort()|停止当前请求|
|getResponseHeader(’‘header’’)|返回指定首部的值|
|getAllResponseHeaders()|把http请求的所有响应首部作为键/值返回|

|属性|描述|
|---|---|
|onreadystatechange|每个状态的改变都会触发这个事件的处理器, 通常会调用一个JS函数|
|readyState|请求状态, 有五个: 0->未初始化, 1->正在加载, 2->已经加载, 3->交互中, 4->完成.|
|responseText|服务器的响应, 表示一个串|
|responseXML|服务器的响应, 表示为XML, 这个对象可解析为DOM对象|
|status|服务器的http状态码(200->OK, 404->NotFound, 等)|
|statusText|http状态码的相应文本(OK或NotFound, 等)|

### 3.简单的发送请求

> - 利用XMLHttpRequest实例与服务器通信包含以下3个关键部分:
> 
> 1. onreadystatechange事件处理函数
> 2. open方法
> 3. send方法

```
<head>  
    <meta charset="UTF-8">  
    <script type="text/javascript">  
        window.onload = function() {  
            //1.获取a节点并为其添加一个onclick响应函数  
            document.getElementsByTagName("a")[0].onclick = function(){  
                //3.创建一个 XMLHttpRequest 对象  
                var request = new XMLHttpRequest();  
                //4.准备发送请求的数据: url  
                var url = this.href;  
                var method = "GET";  
                //5.调用 XMLHttpRequest 对象的open方法,准备请求  
                request.open(method,url);  
                //6.调用 XMLHttpRequest 对象的send方法,发送请求  
                request.send(null);  
                //7.为 XMLHttpRequest 对象添加 onreadystatechange响应函数  
                request.onreadystatechange = function(){  
                //8.判断响应是否完成: XMLHttpRequest 对象的readyState属性值为4时  
                    if(request.readyState==4){  
                //9.再判断响应是否可用:  XMLHttpRequest 对象status属性值为200  
                        if(request.status==200||request.status==304){  
                //10.打印相应结果: responseText  
                            alert(request.responseText);  
                        }  
                    }  
                };  
                //2.取消a节点的默认行为  
                return false;  
            };  
        };  
    </script>  
</head>  
<body>  
    <!--在html文件的同级目录下的ajax.txt文件,内容:Hello Ajax!-->  
    <a href="ajax.txt">发送</a>  
</body>
```
注意: readyState 的值为4时响应成功, 但并不代表响应可用, 当status值(200，即OK)表示响应可用时才可使用

> onreadystatechange:
> 
> - 该事件处理函数由服务器触发, 而不是用户触发
> - 在Ajax执行过程中, 服务器会通知客户端当前的通信状态. 这依靠更新 XMLHttpRequest 对象的readyState属性来实现. 改变readyState属性是服务器对客户端连续操作的一种方式. 每次readyState属性值的改变都会触发onreadystatechange事件

> open(method,url,asynch):
> 
> - 该方法允许我们用一个Ajax调用向服务器发送请求(其实是建立一个请求).
> - method: 请求类型, 类似"GET"或"POST"的字符串. 若只想从服务器获取信息, 而不需发送信息, 使用GET(GET请求也可通过URL附加的查询字符串来发送数据, 但有长度限制). 若需向服务器发送数据, 用POST请求.
> - 某些情况下, 有些浏览器会把多个XMLHttpRequest请求的结果缓存在同一个URL下. 如果对每个请求的响应不同, 就会带来不好的效果. 此时, 我们会在请求的URL后附加时间戳, 来确保URL的唯一性, 起到禁用缓存的效果.
> - asynch: 表示请求是否异步, 一般取默认值true

> send(data):  
> GET请求时, data = null; POST请求时, data为相应数据.

### 4.POST请求

只需要在GET的基础上将method改为"POST",并在open()和send()方法之间添加下面语句:

```
setRequestHeader("ContentType","application/x-www-form-urlencoded");
```
注意: 在开发过程中, 我们一般不用原生的方式来向服务器发送请求.一般用框架来实现, 例如: jQuery、axios等

## 一.HTML格式

> HTML由一些普通文本组成.若服务器通过XMLHttpRequest发送HTML, 文本将存储在responseText属性中.
> 
> 可以直接将responseText赋值给节点的innerHTML属性.
> 
> 优点:
> 
> 1. 从服务器发送的HTML代码在浏览器端不需要用JS再解析
> 2. HTML的可读性好
> 3. HTML代码块与innerHTML属性搭配, 效率高
> 
> 缺点:
> 
> 1. 若需通过Ajax更新一篇文档的多个部分, HTML不合适
> 2. innerHTML并非DOM标准

同目录下四个html文件: index.html、sun.html、zhu.html、sha.html
```
<!--index.html-->  
<head>  
    <meta charset="UTF-8">  
    <title>index</title>  
    <script type="text/javascript">  
        window.onload = function() {  
            var briefA = document.getElementsByTagName("a");  
            for(var i = 0; i < briefA.length; i++) {  
                briefA[i].onclick = function() {  
                    var request = new XMLHttpRequest();  
                    request.open("GET", this.href, true);  
                    request.send(null);  
                    request.onreadystatechange = function() {  
                        if(request.readyState == 4) {  
                            if(request.status == 200) {  
                                document.getElementsByClassName("detail")[0].innerHTML = request.responseText;  
                            }  
                        }  
                    };  
                    return false;  
                };  
            }  
        };  
    </script>  
    <style type="text/css">  
        .brief {  
            width: 200px;  
            margin: 0 auto;  
        }  
        .brief a{  
            font-size: 20px;  
        }  
        .detail {  
            width: 200px;  
            margin: 0 auto;  
        }  
    </style>  
</head>  
<body>  
    <div class="brief">  
        <a href="sun.html">孙悟空</a>  
        <a href="zhu.html">猪八戒</a>  
        <a href="sha.html">沙和尚</a>  
    </div>  
    <div class="detail">  
    </div>  
</body>

<!--sun.html-->  
<p>姓名:孙悟空</p>  
<p>网址:<a href="https://baike.baidu.com/item/%E5%AD%99%E6%82%9F%E7%A9%BA/5576?fr=aladdin" target="_blank">简介</a></p>  
<p>住址:花果山</p>

<!--zhu.html-->  
<p>姓名:猪八戒</p>  
<p>网址:<a href="https://baike.baidu.com/item/%E7%8C%AA%E5%85%AB%E6%88%92/769?fr=aladdin" target="_blank">简介</a></p>  
<p>住址:高老庄</p>

<!--sha.html-->  
<p>姓名:沙和尚</p>  
<p>网址:<a href="https://baike.baidu.com/item/%E6%B2%99%E6%82%9F%E5%87%80/29248?fromtitle=%E6%B2%99%E5%92%8C%E5%B0%9A&fromid=6472043&fr=aladdin" target="_blank">简介</a></p>  
<p>住址:流沙河</p>
```

## 二.XML

> XML就是为了传输数据而生的  
> 优点:
> 
> 1. XML是一种通用的数据格式
> 2. XML可以为数据自定义标记
> 3. 利用DOM就可以完全掌控文档
> 
> 缺点:
> 
> 1. 若XML文档来自服务器, 就必须得保证文档文档含有正确的部首信息. 若文档类型不正确, 那么responseXML的值将为空
> 2. 当浏览器接收到的XML很大时, DOM解析可能会很复杂

同目录下四个文件: index.html、sun.xml、zhu.xml、sha.xml
```
<head>  
    <meta charset="UTF-8">  
    <title>index</title>  
    <script type="text/javascript">  
        window.onload = function() {  
            var briefA = document.getElementsByTagName("a");  
            var detail = document.getElementsByClassName("detail")[0];  
            for(var i = 0; i < briefA.length; i++) {  
                briefA[i].onclick = function() {  
                    var request = new XMLHttpRequest();  
                    request.open("GET", this.href, true);  
                    request.send(null);  
                    request.onreadystatechange = function() {  
                        if(request.readyState == 4&&request.status == 200) {  
                            //1.接受XML格式数据  
                            var result = request.responseXML;  
                            //2.解析XML数据  
                            /* 目标格式  
								 * <p>姓名:</p>  
								 * <p>网址:<a href="">简介</a></p>  
								 * <p>住址:</p>  
								 */  
                            detail.innerHTML = "";  
                            var name = result.getElementsByTagName("name")[0].firstChild.nodeValue;  
                            var web = result.getElementsByTagName("website")[0].firstChild.nodeValue;  
                            var address = result.getElementsByTagName("address")[0].firstChild.nodeValue;  
                            var p1 = document.createElement("p");  
                            p1.innerHTML = "姓名:"+name;  
                            var p2 = document.createElement("p");  
                            p2.innerHTML = "网址:";  
                            var p3 = document.createElement("p");  
                            p3.innerHTML = "住址:" + address;  
                            var a = document.createElement("a");  
                            a.href = web;  
                            a.target = "_blank";  
                            a.innerHTML = "简介";  
                            p2.appendChild(a);  
                            detail.appendChild(p1);  
                            detail.appendChild(p2);  
                            detail.appendChild(p3);  
                        }  
                    };  
                    return false;  
                };  
            }  
        };  
    </script>  
    <style type="text/css">  
        .brief {width: 200px;margin: 0 auto;}  
        .brief a{font-size: 20px;}  
        .detail {width: 200px;margin: 0 auto;}  
    </style>  
</head>  
<body>  
    <div class="brief">  
        <a href="sun.xml">孙悟空</a>  
        <a href="zhu.xml">猪八戒</a>  
        <a href="sha.xml">沙和尚</a>  
    </div>  
    <div class="detail">  
    </div>  
</body>


<!--sun.xml-->  
<!--一定不要忘记XML的部首信息-->  
<?xml version='1.0' encoding='UTF-8'?>  
<detail>  
    <name>孙悟空</name>  
    <website>https://baike.baidu.com/item/%E5%AD%99%E6%82%9F%E7%A9%BA/5576?fr=aladdin</website>  
    <address>花果山</address>  
</detail>

<!--zhu.xml-->  
<?xml version='1.0' encoding='UTF-8'?>  
<detail>  
    <name>猪八戒</name>  
    <website>https://baike.baidu.com/item/%E7%8C%AA%E5%85%AB%E6%88%92/769?fr=aladdin</website>  
    <address>高老庄</address>  
</detail>

<!--sha.xml-->  
<?xml version='1.0' encoding='UTF-8'?>  
<detail>  
    <name>沙和尚</name>  
    <website>https://baike.baidu.com/item/%E6%B2%99%E6%82%9F%E5%87%80/29248?fr=aladdin</website>  
    <address>流沙河</address>  
</detail>
```

## 三.JSON

> 优点:
> 
> 1. 也是一种数据传输的格式, JSON与XML很相似, 但是JSON更轻巧
> 2. JSON不需要从服务端发送含有特定内容类型的首部信息.
> 
> 缺点:
> 
> 1. 语法严谨
> 2. 代码不太易读
> 3. 用eval函数解析JSON字符串存在风险

同目录下四个文件: index.html、sun.json、zhu.json、sha.json

```
<head>  
    <meta charset="UTF-8">  
    <title>index</title>  
    <script type="text/javascript">  
        window.onload = function() {  
            var briefA = document.getElementsByTagName("a");  
            var detail = document.getElementsByClassName("detail")[0];  
            for(var i = 0; i < briefA.length; i++) {  
                briefA[i].onclick = function() {  
                    var request = new XMLHttpRequest();  
                    request.open("GET", this.href, true);  
                    request.send(null);  
                    request.onreadystatechange = function() {  
                        if(request.readyState == 4&&request.status == 200) {  
                            //1.接受json格式数据  
                            var result = request.responseText;  
                            var obj = JSON.parse(result);  
                            detail.innerHTML = "";  
                            var p1 = document.createElement("p");  
                            p1.innerHTML = "姓名:"+obj.person.name;  
                            var p2 = document.createElement("p");  
                            p2.innerHTML = "网址:";  
                            var p3 = document.createElement("p");  
                            p3.innerHTML = "住址:" + obj.person.address;  
                            var a = document.createElement("a");  
                            a.href = obj.person.website;  
                            a.target = "_blank";  
                            a.innerHTML = "简介";  
                            p2.appendChild(a);  
                            detail.appendChild(p1);  
                            detail.appendChild(p2);  
                            detail.appendChild(p3);  
                        }  
                    };  
                    return false;  
                };  
            }  
        };  
    </script>  
    <style type="text/css">  
        .brief {  
            width: 200px;  
            margin: 0 auto;  
        }  
        .brief a{  
            font-size: 20px;  
        }  
        .detail {  
            width: 200px;  
            margin: 0 auto;  
        }  
    </style>  
</head>  
<body>  
    <div class="brief">  
        <a href="sun.json">孙悟空</a>  
        <a href="zhu.json">猪八戒</a>  
        <a href="sha.json">沙和尚</a>  
    </div>  
    <div class="detail">  
    </div>  
</body>

{"person":{  
    "name":"孙悟空",  
    "website":"https://baike.baidu.com/item/%E5%AD%99%E6%82%9F%E7%A9%BA/5576?fr=aladdin",  
    "address":"花果山"  
    }  
}

{"person":{  
    "name":"猪八戒",  
    "website":"https://baike.baidu.com/item/%E7%8C%AA%E5%85%AB%E6%88%92/769?fr=aladdin",  
    "address":"高老庄"  
    }  
}

{"person":{  
    "name":"沙和尚",  
    "website":"https://baike.baidu.com/item/%E6%B2%99%E6%82%9F%E5%87%80/29248?fr=aladdin",  
    "address":"流沙河"  
    }  
}
```


## 1.注意


jQuery中的Ajax, 最底层$.ajax() 第二层$.load() $.get() $.post(), 第三层$.getScript() $.getJSON()

最常用的方法是$.load() $.get() $.post()
## 一.`$.load()`

> load()方法是jQuery中最为简单和常见的Ajax方法, 能载入远程的HTML代码并插入到DOM中.  
> 我们只需用jQuery选择器为HTML片段指定目标位置即可.  
> 默认使用 GET 方式 - 传递附加参数时自动转换为 POST 方式
> 
> 如果我们只需要返回的全部HTML代码的一部分, 可以将url参数设置为"url selector"(注意url与选择器间有一个空格)
> 
> 任何一个HTML的节点都可以使用load()方法来加载Ajax, 结果将直接插入到该节点中的innerHTML中

|参数|说明|
|---|---|
|url|待装入 HTML 网页网址|
|data(可选,对象)|发送至服务器的 key/value 数据. 在jQuery 1.3中也可以接受一个字符串了.|
|callback(可选)|载入成功时回调函数|
```
<head>  
    <meta charset="UTF-8">  
    <title>exer</title>  
    <script src="js/jquery-3.4.0.min.js"></script>  
    <script type="text/javascript">  
        window.onload = function() {  
            $("a").click(function(){  
                var url = this.href;  
                //var url = this.href + " a";//只选择返回的html代码中的a  
                //var args = {"time":new Date()};//args为JSON格式  
                //若向本地服务器发送POST请求时,本地服务器需要具备处理POST请求的能力  
                $(".detail").load(url);//get请求  
                //$(".detail").load(url,args);//post请求  
                return false;  
            });  
        };  
    </script>  
    <style type="text/css">  
        .brief {  
            width: 200px;  
            margin: 0 auto;  
        }  
        .brief a{  
            font-size: 20px;  
        }  
        .detail {  
            width: 200px;  
            margin: 0 auto;  
        }  
    </style>  
</head>  
<body>  
    <div class="brief">  
        <a href="sun.html">孙悟空</a>  
        <a href="zhu.html">猪八戒</a>  
        <a href="sha.html">沙和尚</a>  
    </div>  
    <div class="detail">  
    </div>  
</body>
```
## 二.`$.get()和$.post()`

> $.get()方法使用GET方式来进行异步请求. 结构: $.get(url[,data][,callback][,type])
> 
> $.get()方法中的回调函数只有两个参数: data表示返回的数据, 可以是XML、JSON、HTML等; textstatus表示请求状态, 其值可能是: success, error, notmodify, timeout 4种
> 
> 请求成功时可调用回调函数, 如果需要在出错时执行函数, 请使用 $.ajax()
> 
> $.get()和$.post()方法是jQuery中的全局函数, 而find()等方法都是对jQuery对象进行操作的方法.

| 参数       | 描述                                             |
| -------- | ---------------------------------------------- |
| url      | 待载入页面的URL地址                                    |
| data     | 待发送 Key/value 参数                               |
| callback | 载入成功时回调函数                                      |
| type     | 返回内容格式:xml, html, script, json, text, _default |
```
<head>  
    <meta charset="UTF-8">  
    <title>index</title>  
    <script src="js/jquery-3.4.0.min.js"></script>  
    <script type="text/javascript">  
        $(function(){  
            $("a").click(function(){  
                var url = this.href;  
                var args = {"time":new Date()};  
                $.get(url,args,function(data){  
                    //现在的$.get()方法会根据url识别data格式,此处为xml  
                    //用$(data)包装data,变为jQuery对象  
                    //若data为json格式,这样用:var name = data.person.name;  
                    var name = $(data).find("name").text();  
                    var website = $(data).find("website").text();  
                    var address = $(data).find("address").text();  
                    $(".detail").empty()  
                        .append("<p>姓名:"+name+"</p>")  
                        .append("<p>网址:<a href="+ website +" target='_blank'>简介</a></p>")  
                        .append("<p>住址:"+address+"</p>");  
                });//为了明确,建议在第四个参数处指明返回数据的格式,例:"json"  
                return false;  
            });  
        });  
    </script>  
    <style type="text/css">  
        .brief {  
            width: 200px;  
            margin: 0 auto;  
        }  
        .brief a{  
            font-size: 20px;  
        }  
        .detail {  
            width: 200px;  
            margin: 0 auto;  
        }  
    </style>  
</head>  
<body>  
    <div class="brief">  
        <a href="sun.xml">孙悟空</a>  
        <a href="zhu.xml">猪八戒</a>  
        <a href="sha.xml">沙和尚</a>  
    </div>  
    <div class="detail">  
    </div>  
</body>
```

`$.post()方法和$.get()方法类似, 但是若向本地服务器发送POST请求, 本地服务器需要具备处理POST请求的能力`
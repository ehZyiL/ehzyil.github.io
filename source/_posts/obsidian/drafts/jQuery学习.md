---
title: jQuery学习
date: 2024-03-25
tags:
  - 前端
  - jQuery
categories:
  - 记录
author: ehzyil
description: 
cover: 
banner:
---

# jQuery简介

## jQuery简介

> 为了简化JS开发, 出现了一些JS库. 这些库封装了很多预定义的对象和实用函数. 能帮助使用者建立有高难度交互的Web 2.0特性的富客户端页面, 并兼容大部分浏览器.
>
> 常见的JS库: prototype、dojo、Ext JS、mootools、jQuery

> jQuery是继prototype之后又一个优秀的JS库
>
> jQuery理念: 写的少, 做的多. 优点:
>
> - 轻量级
> - 强大的选择器
> - 出色的DOM操作封装
> - 可靠的事件处理机制
> - 完善的Ajax
> - 出色的浏览器兼容性
> - 链式操作方式
>   …

## jQuery的Hello World

> 应用jQuery, 第一步就是引入jQuery库
>
> jQuery的 $(function(){}) 就相当于 JS 的window.onload
>
> jQuery的神奇之处: 有非常多的、非常有用的选择器

```
ED<head>
    <meta charset="UTF-8">
    <title>index</title>
    <!--1.必须引入jQuery库-->
    <script src="https://code.jquery.com/jquery-3.7.1.js"         integrity="sha256-eKhayi8LEQwp4NKxN+CfCh+3qOVUtJn3QNZ0TciWLP4=" crossorigin="anonymous"></script>
    <script type="text/javascript">
        $(function(){
            //1.选取btn按钮
            //2.为id为btn添加onclick响应函数
            $("#btn").click(function(){
                //3.弹出Hello World!
                alert("Hello World!");
            });
        });
    </script>
</head>
<body>
    <button id="btn">按钮</button>
</body>
```

## jQuery对象

> jQuery对象就是通过jQuery的 $() 包装的DOM对象后产生的对象
>
> jQuery对象是jQuery特有的. jQuery对象可以使用jQuery里的方法, 例: $("#box").html()
>
> jQuery对象无法使用DOM对象的任何方法, 同样DOM对象无法使用jQuery对象里的任何方法
>
> 约定: 若获取的对象为jQuery对象, 在变量面前加上$表示.例: var $variable = jQuery对象;
> var variable = DOM对象;

### 1.jQuery对象—>DOM对象

> jQuery对象不能用DOM中的方法, 但若jQuery没有封装想要的方法, 不得不用DOM中的方法时, 要将jQUery对象转化为DOM对象.
>
> ①jQuery对象是一个数组对象, 可以通过[index]的方法的到对应的DOM对象.
> var $cr = $("#cr");
> var cr = $cr[0];
>
> ②使用jQuery中的get(index)方法得到相应的DOM对象
> var $cr = $("#cr");
> var cr = $cr.get(0);

### 2.DOM对象—>jQuery对象

> 对于一个DOM对象, 只需要用 $() 把DOM对象包装起来, 就可以得到一个jQuery对象, 之后就可以用jQuery对象的方法.
>
> var cr = document.getElementById(“cr”);
> var $cr = $(cr);

## jQuery对象的几个常用方法

| 方法             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| val([value])     | 获取或设置表单元素的value属性值                              |
| attr(name[,val]) | 获取或设置元素节点的指定属性值                               |
| each()           | 对jQuery对象进行遍历,其参数为function,函数内部this代表当前DOM对象 |
| text([str])      | 获取或设置元素节点的文本节点的值                             |
| html([str])      | 获取或设置元素的内部html代码                                 |
| height()         | 设置元素CSS的高度                                            |
| width()          | 设置元素CSS的宽度                                            |
| css(attr,val)    | 设置元素指定属性(attr)的值为val                              |
| removeAttr()     | 删除指定元素的指定属性                                       |

# jQuery选择器

## 基本选择器

> 基本选择器是jQuery中最常用的选择器, 也是最简单的选择器, 它通过元素的id, class 和标签名来查找DOM元素(注意: 在一个网页中id只能用一次, class允许重复)

| 选择器                     | 描述                                   | 返回     |
| -------------------------- | -------------------------------------- | -------- |
| $("#id")                   | 根据id匹配一个元素                     | 单个元素 |
| $(".class")                | 根据class匹配多个元素                  | 集合元素 |
| $(“element”)               | 根据元素名匹配元素                     | 集合元素 |
| $("*")                     | 匹配所有元素                           | 集合元素 |
| $(“selector1,…,selectorn”) | 将每一个选择器匹配到的元素合并一起返回 | 集合元素 |

上述选择器直接放在 $("") 的引号之间

**示例**

html部分:

```
<head>
    <meta charset="UTF-8">
    <title>index</title>
    <script src="https://code.jquery.com/jquery-3.7.1.js"         integrity="sha256-eKhayi8LEQwp4NKxN+CfCh+3qOVUtJn3QNZ0TciWLP4=" crossorigin="anonymous"></script>
    <style type="text/css">
        div, span, p {
            width: 140px;
            height: 140px;
            margin: 5px;
            background: #aaa;
            border: #000 1px solid;
            float: left;
            font-size: 17px;
            font-family: Verdana;
        }
        div.mini {
            width: 55px;
            height: 55px;
            background-color: #aaa;
            font-size: 12px;
        }
        div.hide {
            display: none;
        }
    </style>
</head>
<body>
    <input type="button" value="选择 id 为 one 的元素" id="btn1" />
    <input type="button" value="选择 class 为 mini 的所有元素" id="btn2" />
    <input type="button" value="选择 元素名是 div 的所有元素" id="btn3" />
    <input type="button" value="选择 所有的元素" id="btn4" />
    <input type="button" value="选择 所有的 span 元素和id为two的元素" id="btn5" />
    <br><br>
    <div class="one" id="one">
        id 为 one,class 为 one 的div
        <div class="mini">class为mini</div>
    </div>
    <div class="one" id="two" title="test">
        id为two,class为one,title为test的div
        <div class="mini" title="other">class为mini,title为other</div>
        <div class="mini" title="test">class为mini,title为test</div>
    </div>
    <div class="one">
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini"></div>
    </div>
    <div class="one">
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini" title="tesst">class为mini,title为tesst</div>
    </div>
    <div style="display:none;" class="none">style的display为"none"的div</div>
    <div class="hide">class为"hide"的div</div>
    <div>
        包含input的type为"hidden"的div<input type="hidden" size="8">
    </div>
    <span id="span">^^span元素^^</span>
</body>
```

JS部分:

```
ED//一定不要忘记引入jQuery库
$(function(){
    $("#btn1").click(function(){
        $("#one").css("background-color","#bfa");
    });
    $("#btn2").click(function(){
        $(".mini").css("background-color","#bfa");
    });
    $("#btn3").click(function(){
        $("div").css("background-color","#bfa");
    });
    $("#btn4").click(function(){
        $("*").css("background-color","#bfa");
    });
    $("#btn5").click(function(){
        $("span,#two").css("background-color","#bfa");
    });
});
```

## 层次选择器

> 若想通过DOM元素之间的层次关系来获取特定的元素, 例如: 后代元素 子元素 相邻元素 兄弟元素等, 则需要用层次选择器.

| 选择器                   | 描述                                   | 返回     |
| ------------------------ | -------------------------------------- | -------- |
| $(“ancestor descendant”) | 选取ancestor的所有descendant(后代)元素 | 集合元素 |
| $(“parent > child”)      | 选取parent的所有child(子)元素          | 集合元素 |
| $(“prev + next”)         | 选取紧接在prev元素后的下一个next元素   | 集合元素 |
| $(“prev ~ siblings”)     | 选取prev元素后的所有siblings元素       | 集合元素 |

> 注意: $(“prev ~ siblings”)选择器只能选择#prev元素后的同辈元素; 而jQuery中的 siblings()方法与前后位置无关, 只要是同辈节点即可选取.
>
> \> + - ~ 前后都有一个空格

**示例**

html部分:

```
<head>
    <meta charset="UTF-8">
    <title>index</title>
    <script src="https://code.jquery.com/jquery-3.7.1.js"         integrity="sha256-eKhayi8LEQwp4NKxN+CfCh+3qOVUtJn3QNZ0TciWLP4=" crossorigin="anonymous"></script>
    <style type="text/css">
        div, span, p {
            width: 140px;
            height: 140px;
            margin: 5px;
            background: #aaa;
            border: #000 1px solid;
            float: left;
            font-size: 17px;
            font-family: Verdana;
        }
        div.mini {
            width: 55px;
            height: 55px;
            background-color: #aaa;
            font-size: 12px;
        }
        div.hide {
            display: none;
        }			
    </style>
</head>
<body>		
    <input type="button" value="选择 body 内的所有 div 元素" id="btn1" />
    <input type="button" value="在 body 内, 选择子元素是 div 的." id="btn2" />
    <input type="button" value="选择 id 为 one 的下一个 div 元素" id="btn3" />
    <input type="button" value="选择 id 为 two 的元素后面的所有 div 兄弟元素" id="btn4" />
    <input type="button" value="选择 id 为 two 的元素所有 div 兄弟元素" id="btn5" />
    <input type="button" value="选择 id 为 one 的下一个 span 元素" id="btn6" />
    <input type="button" value="选择 id 为 two 的元素前边的所有的 div 兄弟元素" id="btn7" />
    <br><br>
    <div class="one" id="one">
        id 为 one,class 为 one 的div
        <div class="mini">class为mini</div>
    </div>
    <div class="one" id="two" title="test">
        id为two,class为one,title为test的div
        <div class="mini" title="other">class为mini,title为other</div>
        <div class="mini" title="test">class为mini,title为test</div>
    </div>
    <div class="one">
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini"></div>
    </div>
    <div class="one">
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini" title="tesst">class为mini,title为tesst</div>
    </div>
    <div style="display:none;" class="none">style的display为"none"的div</div>
    <div class="hide">class为"hide"的div</div>
    <div>
        包含input的type为"hidden"的div<input type="hidden" size="8">
    </div>
    <span id="span">^^span元素^^</span>
    <span id="span">--span元素--</span>
</body>
```

JS部分:

```
ED$(function(){
    $("#btn1").click(function(){
        $("body div").css("background-color","#bfa");
    });
    $("#btn2").click(function(){
        $("body > div").css("background-color","#bfa");
    });
    $("#btn3").click(function(){
        $("#one + div").css("background-color","#bfa");
    });
    $("#btn4").click(function(){
        $("#two ~ div").css("background-color","#bfa");
    });
    $("#btn5").click(function(){
        $("#two").siblings("div").css("background-color","#bfa");
    });
    $("#btn6").click(function(){
        //下面的选择器选择的是与 #one 相邻的 span 元素
        //若 span 与 #one 不相邻,则选择器无效
        //$("#one + span").css("background-color","#bfa");
        $("#one").nextAll("span:first").css("background-color","#bfa");
    });
    $("#btn7").click(function(){
        $("#two").prevAll("div").css("background-color","#bfa");
    });
});
```

## 过滤选择器

> 通过特定的规则来筛选出所需的DOM元素, 该选择器都以 “:” 开头
>
> 按不同过滤规则, 过滤选择器可分为: 基本过滤、内容过滤、可见性过滤、属性过滤、子元素过滤和表单对象属性过滤选择器.

### 1.基本过滤选择器

| 选择器         | 描述                                  | 返回     |
| -------------- | ------------------------------------- | -------- |
| :first         | 选取第一个元素                        | 单个元素 |
| :last          | 选取最后一个元素                      | 集合元素 |
| :not(selector) | 去除所有与给定选择器匹配的元素        | 集合元素 |
| :even          | 选取索引为偶数的所有元素, 索引从0开始 | 集合元素 |
| :odd           | 选取索引为奇数的所有元素, 索引从0开始 | 集合元素 |
| :eq(index)     | 选取索引等于index, 索引从0开始        | 集合元素 |
| :gt(index)     | 选取索引大于index, 索引从0开始        | 集合元素 |
| :lt(index)     | 选取索引小于index, 索引从0开始        | 集合元素 |
| :header        | 选取所有的标题元素, 如: h1,h2等       | 集合元素 |
| :animated      | 选取当前正在执行动画的所有元素        | 集合元素 |

示例: html部分:

```
<head>
    <meta charset="UTF-8">
    <title>index</title>
    <script src="https://code.jquery.com/jquery-3.7.1.js"         integrity="sha256-eKhayi8LEQwp4NKxN+CfCh+3qOVUtJn3QNZ0TciWLP4=" crossorigin="anonymous"></script>
    <style type="text/css">
        div, span, p {
            width: 140px;
            height: 140px;
            margin: 5px;
            background: #aaa;
            border: #000 1px solid;
            float: left;
            font-size: 17px;
            font-family: Verdana;
        }
        div.mini {
            width: 55px;
            height: 55px;
            background-color: #aaa;
            font-size: 12px;
        }
        div.hide {
            display: none;
        }			
    </style>
</head>
<body>
    <input type="button" value="选择第一个 div 元素" id="btn1" />
    <input type="button" value="选择最后一个 div 元素" id="btn2" />
    <input type="button" value="选择class不为 one 的所有 div 元素" id="btn3" />
    <input type="button" value="选择索引值为偶数的 div 元素" id="btn4" />
    <input type="button" value="选择索引值为奇数的 div 元素" id="btn5" />
    <input type="button" value="选择索引值为大于 3 的 div 元素" id="btn6" />
    <input type="button" value="选择索引值为等于 3 的 div 元素" id="btn7" />
    <input type="button" value="选择索引值为小于 3 的 div 元素" id="btn8" />
    <input type="button" value="选择所有的标题元素" id="btn9" />
    <input type="button" value="选择当前正在执行动画的所有元素" id="btn10" />
    <input type="button" value="选择 id 为 two 的下一个 span 元素" id="btn11" />
    <h3>基本选择器.</h3>
    <br><br>
    <div class="one" id="one">
        id 为 one,class 为 one 的div
        <div class="mini">class为mini</div>
    </div>
    <div class="one" id="two" title="test">
        id为two,class为one,title为test的div
        <div class="mini" title="other">class为mini,title为other</div>
        <div class="mini" title="test">class为mini,title为test</div>
    </div>
    <div class="one">
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini"></div>
    </div>
    <div class="one">
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini" title="tesst">class为mini,title为tesst</div>
    </div>
    <div style="display:none;" class="none">style的display为"none"的div</div>
    <div class="hide">class为"hide"的div</div>
    <div>
        包含input的type为"hidden"的div<input type="hidden" size="8">
    </div>
    <span id="span">^^span元素 111^^</span>
    <span id="span">^^span元素 222^^</span>
    <div id="mover">正在执行动画的div元素.</div>
</body>
```

JS部分:

```
ED$(document).ready(function(){
    //之后会说jQuery的动画
    function anmateIt(){
        $("#mover").slideToggle("slow", anmateIt);
    }
    anmateIt();
    $("#btn1").click(function(){
        $("div:first").css("background", "#bfa");
    });
    $("#btn2").click(function(){
        $("div:last").css("background", "#bfa");
    });
    $("#btn3").click(function(){
        $("div:not(.one)").css("background", "#bfa");
    });
    $("#btn4").click(function(){
        $("div:even").css("background", "#bfa");
    });
    $("#btn5").click(function(){
        $("div:odd").css("background", "#bfa");
    });
    $("#btn6").click(function(){
        $("div:gt(3)").css("background", "#bfa");
    });
    $("#btn7").click(function(){
        $("div:eq(3)").css("background", "#bfa");
    });
    $("#btn8").click(function(){
        $("div:lt(3)").css("background", "#bfa");
    });
    $("#btn9").click(function(){
        $(":header").css("background", "#bfa");
    });
    $("#btn10").click(function(){
        $(":animated").css("background", "#bfa");
    });
    $("#btn11").click(function(){
        $("#two").nextAll("span:first").css("background", "#bfa");
    });
});
```





### 2.内容过滤选择器

> 内容过滤选择器的过滤规则主要体现在它所包含的子元素和文本上

| 选择器          | 描述                             | 返回     |
| --------------- | -------------------------------- | -------- |
| :contains(text) | 选取含有文本内容为text的元素     | 集合元素 |
| :empty          | 选取不包含子元素或文本为空的元素 | 集合元素 |
| :has(selector)  | 选取含有选择器所匹配的元素的元素 | 集合元素 |
| :parent         | 选取含有子元素或者文本的元素     | 集合元素 |

html部分:

```
<head>
    <meta charset="UTF-8">
    <title>index</title>
    <script src="https://code.jquery.com/jquery-3.7.1.js"         integrity="sha256-eKhayi8LEQwp4NKxN+CfCh+3qOVUtJn3QNZ0TciWLP4=" crossorigin="anonymous"></script>
    <style type="text/css">
        div, span, p {
            width: 140px;
            height: 140px;
            margin: 5px;
            background: #aaa;
            border: #000 1px solid;
            float: left;
            font-size: 17px;
            font-family: Verdana;
        }
        div.mini {
            width: 55px;
            height: 55px;
            background-color: #aaa;
            font-size: 12px;
        }
        div.hide {
            display: none;
        }			
    </style>
</head>
<body>		
    <input type="button" value="选择 含有文本 'di' 的 div 元素" id="btn1" />
    <input type="button" value="选择不包含子元素(或者文本元素) 的 div 空元素" id="btn2" />
    <input type="button" value="选择含有 class 为 mini 元素的 div 元素" id="btn3" />
    <input type="button" value="选择含有子元素(或者文本元素)的div元素" id="btn4" />

    <br><br>
    <div class="one" id="one">
        id 为 one,class 为 one 的div
        <div class="mini">class为mini</div>
    </div>
    <div class="one" id="two" title="test">
        id为two,class为one,title为test的div
        <div class="mini" title="other">class为mini,title为other</div>
        <div class="mini" title="test">class为mini,title为test</div>
    </div>
    <div class="one">
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini"></div>
    </div>
    <div class="one">
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini" title="tesst">class为mini,title为tesst</div>
    </div>
    <div style="display:none;" class="none">style的display为"none"的div</div>
    <div class="hide">class为"hide"的div</div>
    <div>
        包含input的type为"hidden"的div<input type="hidden" size="8">
    </div>
    <div id="mover">正在执行动画的div元素.</div>
</body>
```

JS部分:

```
$(document).ready(function(){
    $("#btn1").click(function(){
        $("div:contains('di')").css("background", "#bfa");
    });
    $("#btn2").click(function(){
        $("div:empty").css("background", "#bfa");
    });
    $("#btn3").click(function(){
        $("div:has(.mini)").css("background", "#bfa");
    });
    $("#btn4").click(function(){
        $("div:parent").css("background", "#bfa");
        //$("div:not(:empty)").css("background", "#bfa");
    });
});
```

### 3.可见性过滤选择器

> 可见性过滤选择器是根据元素的可见和不可见状态来选择相应的元素
>
> 可见选择器: hidden不仅包括样式属性display为none的元素, 也包含文本隐藏域(<input type=‘hidden’/>)和 visible:hidden之类的元素

| 选择器   | 描述               | 返回     |
| -------- | ------------------ | -------- |
| :hidden  | 选取所有不可见元素 | 集合元素 |
| :visible | 选取所有可见元素   | 集合元素 |

html部分:

```
$(document).ready(function(){
    $("#btn1").click(function(){
        $("div:visible").css("background", "#bfa");
    });
    $("#btn2").click(function(){
        //alert($("div:hidden").length);
        //show(time):可以使不可见的元素变为可见,time表示时间,以ms为单位
        //jQuery的很多方法支持方法的连缀,即一个方法的返回值仍然是调用该
        //方法的 jQuery 对象:可以继续调用该对象的其他方法.
        $("div:hidden").show(2000).css("background","#bfa");
    });
    $("#btn3").click(function(){
        //attr()方法,传一个参数表示获取属性值
        //val() 方法可以返回某一个表单元素的 value 属性值. 
        alert($("input:hidden").val());
    });
});
```

JS部分:

```
$(document).ready(function(){
    $("#btn1").click(function(){
        $("div:visible").css("background", "#bfa");
    });
    $("#btn2").click(function(){
        //alert($("div:hidden").length);
        //show(time):可以使不可见的元素变为可见,time表示时间,以ms为单位
        //jQuery的很多方法支持方法的连缀,即一个方法的返回值仍然是调用该
        //方法的 jQuery 对象:可以继续调用该对象的其他方法.
        $("div:hidden").show(2000).css("background","#bfa");
    });
    $("#btn3").click(function(){
        //attr()方法,传一个参数表示获取属性值
        //val() 方法可以返回某一个表单元素的 value 属性值. 
        alert($("input:hidden").val());
    });
});
```

### 4.属性过滤选择器

> 属性过滤选择器的过滤规则是通过元素的属性来获取相应的元素, 属性过滤选择器不以":"开头

| 选择器                  | 描述                                                         | 返回     |
| ----------------------- | ------------------------------------------------------------ | -------- |
| [attribute]             | 选取拥有此属性的元素                                         | 集合元素 |
| [attribute=value]       | 选取指定属性的值为value的元素                                | 集合元素 |
| [attribute!=value]      | 选取指定属性的值不等于value的元素                            | 集合元素 |
| [attribute^=value]      | 选取指定属性的值以value开始的元素                            | 集合元素 |
| [attribute$=value]      | 选取指定属性的值以value结束的元素                            | 集合元素 |
| [attribute*=value]      | 选取指定属性的值含有value的元素                              | 集合元素 |
| [selector1]…[selectorn] | 用属性选择器合并成一个复合属性选择器,满足多个条件,每选择一次,缩小一次范围 | 集合元素 |

html部分:

```
<head>
    <meta charset="UTF-8">
    <title>index</title>
    <script src="https://code.jquery.com/jquery-3.7.1.js"         integrity="sha256-eKhayi8LEQwp4NKxN+CfCh+3qOVUtJn3QNZ0TciWLP4=" crossorigin="anonymous"></script>
    <style type="text/css">
        div, span, p {
            width: 140px;
            height: 140px;
            margin: 5px;
            background: #aaa;
            border: #000 1px solid;
            float: left;
            font-size: 17px;
            font-family: Verdana;
        }
        div.mini {
            width: 55px;
            height: 55px;
            background-color: #aaa;
            font-size: 12px;
        }
        div.hide {
            display: none;
        }			
    </style>
</head>
<body>
    <input type="button" value="选取含有 属性title 的div元素." id="btn1"/>
    <input type="button" value="选取 属性title值等于'test'的div元素." id="btn2"/>
    <input type="button" value="选取 属性title值不等于'test'的div元素(没有属性title的也将被选中)." id="btn3"/>
    <input type="button" value="选取 属性title值 以'te'开始 的div元素." id="btn4"/>
    <input type="button" value="选取 属性title值 以'est'结束 的div元素." id="btn5"/>
    <input type="button" value="选取 属性title值 含有'es'的div元素." id="btn6"/>
    <input type="button" value="组合属性选择器,首先选取有属性id的div元素，然后在结果中 选取属性title值 含有'es'的 div 元素." id="btn7"/>
    <input type="button" value="选取 含有 title 属性值, 且title 属性值不等于 test 的 div 元素." id="btn8"/>
    <br><br>
    <div class="one" id="one">
        id 为 one,class 为 one 的div
        <div class="mini">class为mini</div>
    </div>
    <div class="one" id="two" title="test">
        id为two,class为one,title为test的div
        <div class="mini" title="other">class为mini,title为other</div>
        <div class="mini" title="test">class为mini,title为test</div>
    </div>
    <div class="one">
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini"></div>
    </div>
    <div class="one">
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini" title="tesst">class为mini,title为tesst</div>
    </div>
    <div style="display:none;" class="none">style的display为"none"的div</div>
    <div class="hide">class为"hide"的div</div>
    <div>
        包含input的type为"hidden"的div<input type="hidden" value="123456789" size="8">
    </div>
    <div id="mover">正在执行动画的div元素.</div>
</body>
```

JS部分:

```
$(function(){
    $("#btn1").click(function(){
        $("div[title]").css("background", "#bfa");
    });
    $("#btn2").click(function(){
        $("div[title='test']").css("background", "#bfa");
    });
    $("#btn3").click(function(){
        //选取的元素中包含没有 title 的 div 元素. 
        $("div[title!='test']").css("background", "#bfa");
    });
    $("#btn4").click(function(){
        $("div[title^='te']").css("background", "#bfa");
    });
    $("#btn5").click(function(){
        $("div[title$='est']").css("background", "#bfa");
    });
    $("#btn6").click(function(){
        $("div[title*='es']").css("background", "#bfa");
    });
    $("#btn7").click(function(){
        $("div[id][title*='es']").css("background", "#bfa");
    });
    $("#btn8").click(function(){
        $("div[title][title!='test']").css("background", "#bfa");
    });
})
```



### 5.子元素过滤选择器

| 选择器                              | 描述                                                  | 返回     |
| ----------------------------------- | ----------------------------------------------------- | -------- |
| :nth-child(index/even/odd/equation) | 选取每个父元素下的第index个子元素或奇偶元素(index>=1) | 集合元素 |
| :first-child                        | 选取每个父元素的第一个子元素                          | 集合元素 |
| :last-child                         | 选取每个父元素的最后一个子元素                        | 集合元素 |
| :only-child                         | 若某元素是其父元素中唯一的子元素, 那么它将被匹配      | 集合元素 |

> :nth-child()详解:
>
> - :nth-child(even/odd):能选取每个父元素下的索引值为偶/奇数的元素
> - :nth-child(2):能选取每个父元素下的索引值为2的元素
> - :nth-child(3n):能选取每个父元素下索引值为3的倍数的元素
> - :nth-child(3n+1):能选取每个父元素下索引值为3n+1的元素
>
> 选取子元素时, 需要在选择器前添加一个空格

html部分:

```
<head>
    <meta charset="UTF-8">
    <title>index</title>
    <script src="js/jquery-3.4.0.min.js"></script>
    <style type="text/css">
        div, span, p {
            width: 140px;
            height: 140px;
            margin: 5px;
            background: #aaa;
            border: #000 1px solid;
            float: left;
            font-size: 17px;
            font-family: Verdana;
        }
        div.mini {
            width: 55px;
            height: 55px;
            background-color: #aaa;
            font-size: 12px;
        }
        div.hide {
            display: none;
        }			
    </style>
</head>
<body>		
    <input type="button" value="选取每个class为one的div父元素下的第2个子元素." id="btn1"/>
    <input type="button" value="选取每个class为one的div父元素下的第一个子元素." id="btn2"/>
    <input type="button" value="选取每个class为one的div父元素下的最后一个子元素." id="btn3"/>
    <input type="button" value="如果class为one的div父元素下的仅仅只有一个子元素，那么选中这个子元素." id="btn4"/>
    <br><br>
    <div class="one" id="one">
        id 为 one,class 为 one 的div
        <div class="mini">class为mini</div>
    </div>
    <div class="one" id="two" title="test">
        id为two,class为one,title为test的div
        <div class="mini" title="other">class为mini,title为other</div>
        <div class="mini" title="test">class为mini,title为test</div>
    </div>
    <div class="one">
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini"></div>
    </div>
    <div class="one">
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini">class为mini</div>
        <div class="mini" title="tesst">class为mini,title为tesst</div>
    </div>
    <div style="display:none;" class="none">style的display为"none"的div</div>
    <div class="hide">class为"hide"的div</div>
    <div>
        包含input的type为"hidden"的div<input type="hidden" value="123456789" size="8">
    </div>
    <div id="mover">正在执行动画的div元素.</div>
</body>
```

JS部分:

```
ED$(document).ready(function(){
    $("#btn1").click(function(){
        //选取子元素, 需要在选择器前添加一个空格. 
        $("div.one :nth-child(2)").css("background", "#ffbbaa");
    });
    $("#btn2").click(function(){
        $("div.one :first-child").css("background", "#ffbbaa");
    });
    $("#btn3").click(function(){
        $("div.one :last-child").css("background", "#ffbbaa");
    });
    $("#btn4").click(function(){
        $("div.one :only-child").css("background", "#ffbbaa");
    });
});
```

### 6.表单对象属性过滤选择器

| 选择器    | 描述                                | 返回     |
| --------- | ----------------------------------- | -------- |
| :enabled  | 选取所有可用元素                    | 集合元素 |
| :disabled | 选取所有不可用元素                  | 集合元素 |
| :checked  | 选取所有被选中的元素(单选框,复选框) | 集合元素 |
| :selected | 选取所有被选中选项元素(下拉列表)    | 集合元素 |

html部分:

```
<head>
    <meta charset="UTF-8">
    <title>index</title>
    <script src="https://code.jquery.com/jquery-3.7.1.js"         integrity="sha256-eKhayi8LEQwp4NKxN+CfCh+3qOVUtJn3QNZ0TciWLP4=" crossorigin="anonymous"></script>
    <style type="text/css">
        div, span, p {
            width: 140px;
            height: 140px;
            margin: 5px;
            background: #aaa;
            border: #000 1px solid;
            float: left;
            font-size: 17px;
            font-family: Verdana;
        }
        div.mini {
            width: 55px;
            height: 55px;
            background-color: #aaa;
            font-size: 12px;
        }
        div.hide {
            display: none;
        }			
    </style>
</head>
<body>
    <h3>表单对象属性过滤选择器</h3>
    <button id="btn1">对表单内 可用input 赋值操作.</button>
    <button id="btn2">对表单内 不可用input 赋值操作.</button><br /><br />
    <button id="btn3">获取多选框选中的个数.</button>
    <button id="btn4">获取多选框选中的内容.</button><br /><br />
    <button id="btn5">获取下拉框选中的内容.</button><br /><br />
    <form id="form1" action="#">			
        可用元素: <input name="add" value="可用文本框1"/><br>
        不可用元素: <input name="email" disabled="true" value="不可用文本框"/><br>
        可用元素: <input name="che" value="可用文本框2"/><br>
        不可用元素: <input name="name" disabled="true" value="不可用文本框"/><br>
        <br>
        多选框: <br>
        <input type="checkbox" name="newsletter" checked="checked" value="test1" />test1
        <input type="checkbox" name="newsletter" value="test2" />test2
        <input type="checkbox" name="newsletter" value="test3" />test3
        <input type="checkbox" name="newsletter" checked="checked" value="test4" />test4
        <input type="checkbox" name="newsletter" value="test5" />test5
        <br><br>
        下拉列表1: <br>
        <select name="test" multiple="multiple" style="height: 100px">
            <option>浙江</option>
            <option selected="selected">辽宁</option>
            <option>北京</option>
            <option selected="selected">天津</option>
            <option>广州</option>
            <option>湖北</option>
        </select>
        <br><br>
        下拉列表2: <br>
        <select name="test2">
            <option>浙江</option>
            <option>辽宁</option>
            <option selected="selected">北京</option>
            <option>天津</option>
            <option>广州</option>
            <option>湖北</option>
        </select>
        <textarea rows="" cols=""></textarea>
    </form>		
</body>
```

js部分:

```
ED$(function(){
    $("#btn1").click(function(){
        //使所有的可用的单行文本框的 value 值变为 已重新赋值
        alert($(":text:enabled").val());
        $(":text:enabled").val("已重新赋值");
    });
    $("#btn2").click(function(){
        $(":text:disabled").val("www.baidu.com");
    });
    $("#btn3").click(function(){
        var num = 
            $(":checkbox[name='newsletter']:checked").length;
        alert(num);
    });
    $("#btn5").click(function(){
        //实际被选择的不是 select, 而是 select 的 option 子节点
        //所以要加一个 空格. 
        //var len = $("select :selected").length
        //alert(len);
        //因为 $("select :selected") 选择的是一个数组
        //当该数组中有多个元素时, 通过 .val() 方法就只能获取第一个被选择的值了. 
        //alert($("select :selected").val());
        //jQuery 对象遍历的方式使 each, 在 each 内部的 this 是正在
        //得到的 DOM 对象, 而不是一个 jQuery 对象
        $("select :selected").each(function(){
            alert(this.value);
        });
    });
    $("#btn4").click(function(){
        $(":checkbox[name='newsletter']:checked").each(function(){
            alert(this.value);
        });
    });
})
```



## 表单选择器

| 选择器    | 描述                                        | 返回     |
| --------- | ------------------------------------------- | -------- |
| :input    | 选取所有的input, textarea,select,button元素 | 集合元素 |
| :text     | 选取所有的单行文本框                        | 集合元素 |
| :password | 选取所有的密码框元素                        | 集合元素 |
| :radio    | 选取所有的单选框                            | 集合元素 |
| :checkbox | 选取所有的多选框                            | 集合元素 |
| :submit   | 选取所有的提交按钮                          | 集合元素 |
| :image    | 选取所有的图像按钮                          | 集合元素 |
| :reset    | 选取所有的重置按钮                          | 集合元素 |
| :button   | 选取所有的按钮                              | 集合元素 |
| :file     | 选取所有的上传域                            | 集合元素 |
| :hidden   | 选取所有的不可见元素                        | 集合元素 |



# jQuery中的事件

## 加载DOM

> 在页面加载完毕后, 浏览器会通过JS为DOM元素添加事件. 在常规JS代码中, 通常使用window.onload方法, 而在jQuery中使用$(document).ready()方法, 可简写为$().

> 两者区别:
>
> 1. window.onload必须等待网页中的所有内容加载完毕后(包括图片)才能执行其中代码, 而且只会有一个window.onload会执行, 因为后续修改window.onload会覆盖之前的
> 2. $(document).ready()在网页中的DOM结构绘制完毕之后便会执行, 可能DOM元素关联的东西并没有加载完成, 而且这个函数可编写多个, 这多个中的代码均会执行.

**还可以使用`$(function (){} `**
## 绑定事件

html部分:

```
<head>
    <meta charset="UTF-8">
    <title>index</title>
    <script src="https://code.jquery.com/jquery-3.7.1.js"         integrity="sha256-eKhayi8LEQwp4NKxN+CfCh+3qOVUtJn3QNZ0TciWLP4=" crossorigin="anonymous"></script>
    <style type="text/css">
        *{
            margin: 0;
            padding: 0;
        }
        .panel{
            width: 400px;
            margin: 0 auto;
        }
        .head{
            width: 400px;
            background-color: #bfa;
        }
        .head:hover{
            cursor: pointer;
        }
        .content{
            width: 400px;
            background-color: #98C8EF;
            display: none;
        }
    </style>
</head>
<body>
    <div class="panel">
        <h2 class="head">什么是jQuery?</h2>
        <div class="content">
            jQuery是一个快速、简洁的JavaScript框架，是继Prototype之后又一个优秀的JavaScript代码库（或JavaScript框架）。jQuery设计的宗旨是“write Less，Do More”，即倡导写更少的代码，做更多的事情。它封装JavaScript常用的功能代码，提供一种简便的JavaScript设计模式，优化HTML文档操作、事件处理、动画设计和Ajax交互。
        </div>
    </div>
</body>
```

### 1.直接绑定

> is()方法: 用来判断给定的某个jQuery对象是否符合指定的jQuery选择器

```
$(function(){
    $(".head").click(function(){
        var flag = $(".content").is(":hidden");
        if(flag){
            $(".content").show();
        }else{
            $(".content").hide();
        }
    });
});
```

### 2.bind()绑定

```
$(function(){
    $(".head").bind("click",function(){
        var flag = $(".content").is(":hidden");
        if(flag){
            $(".content").show();
        }else{
            $(".content").hide();
        }
    });
});
```

## 合成事件

> hover(): 模拟光标悬停事件. 当光标移动到元素上时, 会触发指定的第一个函数, 当光标移出这个元素时, 会触发指定的第二个函数.
>
> toggle(): 用于模拟鼠标连续单击事件. 第一次单击元素, 触发第一个函数, 再次单击同一个元素, 触发第二个函数. 之后依次类推. 现在废除了之前说的作用, 只用来切换元素的可见性

### 1.hover()

```
//html代码还是上方的
$(function(){
    $(".head").hover(function(){
        $(".content").show();
    },function(){
        $(".content").hide();
    });
});
```

## 移除事件

> unbind()方法:
> 移除某按钮上所有的click事件: $("#btn").unbind(“click”);
> 移除某按钮上所有的事件: $("#btn").unbind();
>
> one()方法:
> 该方法可以为元素绑定处理函数. 当处理函数触发一次后, 便会被立即删除. 即在每个对象上, 事件处理函数只会被执行一次.

## 五.事件冒泡

> 事件会按照DOM层次结构像水泡一样不断向上, 直到传递到顶端
>
> 解决: 在事件处理函数中返回false, 会对事件停止冒泡, 还可以停止元素的默认行为.

由于事件的冒泡, 以下例子中, 点击p标签会向上传递点击事件, body和div的点击事件也会被触发.

```
<head>
    <meta charset="UTF-8">
    <title>index</title>
    <script src="https://code.jquery.com/jquery-3.7.1.js"         integrity="sha256-eKhayi8LEQwp4NKxN+CfCh+3qOVUtJn3QNZ0TciWLP4=" crossorigin="anonymous"></script>
    <style type="text/css">
        .content{
            width: 200px;
            height: 200px;
            background-color: #bfa;
        }
        p{
            width: 100px;
            height: 100px;
            background-color: red;
        }
    </style>
    <script type="text/javascript">
        $(function() {
            $("body").click(function(){
                alert("body click");
            });
            $(".content").click(function(){
                alert("div click");
            });
            $("p").click(function(){
                alert("p click");
                //return false;//阻止事件冒泡
            });
        });
    </script>
</head>
<body>body
    <div class="content">div
        <p>p</p>
    </div>
</body>
```

# jQuery的DOM

## DOM简介

> DOM(Document Object Model—文档对象模型): 一种与浏览器、平台、语言无关的接口, 使用该接口可以轻松地访问页面中所有的标准组件
>
> DOM操作的分类:
>
> - DOM Core: DOM Core并不专属于JS, 任何一种支持DOM的程序设计语言都可以使用它. 它的用途并非仅限于处理网页, 也可以用来处理任何一种用标记语言编写的文档, 如:XML
> - HTML DOM: 使用JS和DOM为HTML文件编写脚本时, 有许多专属于HTML-DOM的属性
> - CSS-DOM: 针对于CSS操作, 在JS中, CSS-DOM主要用于获取和设置style对象的各种属性

## jQuery操作HTML-DOM

### 1.查找结点

> 1. 查找属性结点: 通过jQuery选择器完成
> 2. 操作属性节点: 查找到元素对应节点后, 通过jQuery对象的attr()方法来获取它的各属性值
> 3. 操作文本节点: 通过text()方法

### 2.创建节点

> 使用jQuery的工厂函数$(str); 会根据传入的html格式str字符串, 创建一个DOM对象, 并把这个DOM对象包装成一个jQuery对象返回.
>
> 注意:
>
> 1. 创建的新元素节点不会被自动添加到文档中, 需要手动调用jQuery的方法插入到当前文档.
>
> 2. 当创建单个元素时, 标签开始, 必须要结束该标签.例: $("  ")
>
> 创建文本节点就是在创建元素节点时直接将文本内容写进去; 创建属性节点也是这样.

### 3.插入节点

| 方法        | 描述                                         |
| ----------- | -------------------------------------------- |
| append()    | 向每个匹配的元素的内容的结尾处追加内容       |
| appendTo()  | 将每个匹配的元素追加到指定元素的内部的结尾处 |
| prepend()   | 向每个匹配的元素的内容的开始处插入内容       |
| prependTo() | 将每个匹配的元素插入到指定元素的内部的开始处 |

这些方法插入的节点都将成为文档中某个节点的子节点.

| 方法             | 描述                |
| -------------- | ----------------- |
| after()        | 向每个匹配的元素的之后插入内容   |
| insertAfter()  | 将匹配的元素的插入到指定的元素之后 |
| before()       | 向每个匹配的元素的之前插入内容   |
| insertBefore() | 将匹配的元素的插入到指定的元素之首 |

以上方法不但能将新创建的DOM元素插入到文档中, 也能对原有的DOM元素进行移动.

```
//下面两种方法等价:
$("#id").append("<p>我是追加的内容</p>");
$("<p>我是追加的内容</p>").appendTo($("#id"));
//prepend与prependTo和上述方法类似
//after与insertAfter和上述方法类似
//before与insertBefore和上述方法类似
```

### 4.删除节点

> remove(): 从DOM中删除所有匹配的元素, 传入的参数用于根据jQuery表达式来筛选元素. 当某个节点用remove()方法删除后, 该节点以及所包含的所有后代节点将被同时删除. 这个方法的返回值是一个指向已被删除的节点的引用.

> empty(): 清空节点—清空元素中的所有后代节点(不包括属性节点).

### 5.复制节点

> clone(): 克隆匹配的DOM元素, 返回值为克隆后的副本. 此时复制的节点不具有任何行为.
>
> clone(true): 复制元素的同时也可复制元素中的事件.

### 6.替换节点

> replaceWith(): 将所有匹配到的元素都替换为指定的HTML或DOM元素
>
> replaceAll(): 颠倒了的replaceWith()方法, 与append()与appendTo()方法类似
>
> 注意: 若在替换之前, 已在元素上绑定了事件, 替换之后原先绑定的事件会与原来的元素一起消失.
>
> 若想互换文档中的两个节点的话, 要先克隆其中一个节点.

### 7.包裹节点

> 做一些网页特效时, 往往会用到这些方法.
>
> wrap(): 将匹配到的所有元素一个个用其他元素包裹起来
>
> wrapAll(): 将匹配到的所有元素作为整体用一个元素包裹起来
>
> wrapInner(): 将每一个匹配元素的内容用其他结构化标记包裹起来

```
//例:
$("li").wrap("<font color='red'></font>");
//之后所有的li将变为:<font color='red'><li>...</li></font>
```

## jQuery操作CSS-DOM

### 1.操作class属性

> 获取和设置class属性都可以使用attr()方法, 也可以使用专门的

| 方法            | 描述                      |
| ------------- | ----------------------- |
| addClass()    | 为class属性添加一个值           |
| removeClass() | 从匹配元素中删除全部或指定class属性的值  |
| toggleClass() | 切换class属性的值, 有则删除, 无则添加 |
| hasClass()    | 判断元素class属性是否有某个值       |

### 2.操作元素样式

> 操作元素的css样式, 均可通过 css() 方法实现
>
> 1. 通过opacity属性操作元素透明度: css(“opacity”,value);
> 2. 通过width和height属性操作元素宽高, 也可直接用width()和height()方法操作
>
> 获取元素在当前视窗中的相对位移: offset()方法. 其返回的对象包含两个属性: top和left. 该方法只对可见元素有效

# jQuery中的动画

## hide()与show()

> 通过hide()与show()方法实现动画, 其中hide()相当于css(“display”,“none”).
>
> 以上两个方法不带任何参数时没有任何动画效果, 可以通过添加一个速度参数使元素"动起来". 这两个方法会同时改变内容的高度、宽度、透明度

```
<head>
    <meta charset="UTF-8">
    <title>index</title>
    <script src="https://code.jquery.com/jquery-3.7.1.js"         integrity="sha256-eKhayi8LEQwp4NKxN+CfCh+3qOVUtJn3QNZ0TciWLP4=" crossorigin="anonymous"></script>
    <style type="text/css">
        * {
            margin: 0;
            padding: 0;
        }
        .panel {
            width: 400px;
            margin: 0 auto;
        }
        .head {
            width: 400px;
            background-color: #bfa;
        }
        .head:hover {
            cursor: pointer;
        }
        .content {
            width: 400px;
            background-color: #98C8EF;
            display: none;
        }
    </style>
    <script type="text/javascript">
        $(function() {
            $(".head").hover(function() {
                $(".content").show(1000);
            }, function() {
                $(".content").hide(1000);
            });
        });
    </script>
</head>
<body>
    <div class="panel">
        <h2 class="head">什么是jQuery?</h2>
        <div class="content">
            jQuery是一个快速、简洁的JavaScript框架，是继Prototype之后又一个优秀的JavaScript代码库（或JavaScript框架）。jQuery设计的宗旨是“write Less，Do More”，即倡导写更少的代码，做更多的事情。它封装JavaScript常用的功能代码，提供一种简便的JavaScript设计模式，优化HTML文档操作、事件处理、动画设计和Ajax交互。
        </div>
    </div>
</body>
```

## slideDown()与slideUp()

> 这两个方法只会改变元素的高度, 也可以传一个速度参数

```
//html其余代码同上
$(function() {
    $(".head").hover(function() {
        $(".content").slideDown(1000);
    }, function() {
        $(".content").slideUp(1000);
    });
});
```

## fadeIn()与fadeOut()

> 这两个方法只会改变元素的透明度

```
//html其余代码同上
$(function() {
    $(".head").hover(function() {
        $(".content").fadeIn();
    }, function() {
        $(".content").fadeOut();
    });
});
```

## 切换元素的可见性

### 1.toggle()

> 直接切换元素可见状态, 不传参数的话, 没有动画的效果.传一个速度参数, 便可看见动画的效果
>
> 这个方法相当于show()与hide()方法的组合

```
//html其余代码同上
$(function() {
    $(".head").click(function(){
        $(".content").toggle("slow");
    });
});
```

### 2.slideToggle()

> 通过高度的变化来切换元素的可见状态
>
> 想当于slideDown()与slideUp()方法的组合

```
//html其余代码同上
$(function() {
    $(".head").click(function(){
        $(".content").slideToggle("slow");
    });
});
```

### 3.fadeToggle()

> 通过透明度来切换元素的可见状态
>
> 相当于fadeIn()与fadeOut()方法的组合

```
//html其余代码同上
$(function() {
    $(".head").click(function(){
        $(".content").fadeToggle("slow");
    });
});
```

### 4.fadeTo()

> 把元素的透明度以渐近的方式调整到指定值(0-1之间)

```
//html其余代码同上
$(function() {
    $(".content").show();
    var i = 1;
    $(".head").click(function(){
        //每次点击透明度减少0.2
        $(".content").fadeTo("slow",i);
        i -= 0.2;
    });
});
```




---
references:
-  [jQuery选择器 - CRainyDay](https://crainyday.gitee.io/jQuery1-1.html) 
- [jQuery中的事件 - CRainyDay](https://crainyday.gitee.io/jQuery1-5.html) 
-  [jQuery的DOM - CRainyDay](https://crainyday.gitee.io/jQuery1-4.html) 
-  [jQuery中的事件 - CRainyDay](https://crainyday.gitee.io/jQuery1-5.html) 
  
---

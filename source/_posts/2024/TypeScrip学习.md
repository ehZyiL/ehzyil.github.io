---
title: TypeScript学习
author: ehzyil
tags:
  - TypeScript
categories:
  - 记录
date: 2024-03-11 20:22:46
headimg:
---
## TypeScript学习

### 创建TS项目



**前提**
需要先安装npm

**安装**typescript

```
npm install -g typescript
```

命令框cmd,输入 tsc -v后，显示了版本信息后，就说明安装成功

**安装ts-node**

```
npm install -g 
ts-node -v
```

命令框cmd,输入ts-node -v后，显示了版本信息后，就说明安装成功

**创建项目**

1. 新建一个叫 demo的文件夹
2. 在demo路径下打开cmd,并输入tsc --init 生成tsconfig.json文件
3. 同样在cmd中输入npm init -y 生成package.json文件

**运行**

新建 **index.ts** 文件，并写下这样的代码：

```tsx
const hello: string = 'HelloWorld';
console.log(hello);
```

在cmd中输入`tsc index.ts `后会生成文件，然后`node  .\index.js`命令行窗口即会显示HelloWorld

### TS 中的基础类型和任意类型

> 基础类型：Boolean、Number、String、`null`、`undefined` 以及 ES6 的 [Symbol](http://es6.ruanyifeng.com/#docs/symbol) 和 ES10 的 [BigInt](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/BigInt)。

#### 1.字符串类型

```tsx
//string
let a:string='123'
let b:string='ddd${a}'
console.log(a);
console.log(b);
```

#### 2.数字类型

```tsx
//number
// let c:number  ='123'  //error
let d:number=123;
let e:number=Infinity;
let notAnumber:number=NaN;
let decimal:number=0.5;
let hex:number=0xf11;
let binary:number=0b10101010;
let octal:number=0o744;
```



#### 3.布尔类型

```tsx
//boolean
let f:boolean=false;

let t1:boolean=Boolean(true);
let t2:boolean=Boolean(1);
console.log(t1);
console.log(t2);

//不能将类型“Boolean”分配给类型“boolean”。
// “boolean”是基元，但“Boolean”是包装器对象。如可能首选使用“boolean”
// let createdBoolean1: boolean = new Boolean(1) //error
let createdBoolean2: Boolean = new Boolean(1)
```



#### 4.空值类型

```tsx
//void
function voidFn():void {
    console.log('test void')
}
```

#### 5.Null和undefined类型

```tsx
let u:undefined=undefined;
let n:null=null;

//Null和undefined类型
let u1:null=null;
let n1:undefined=undefined;

```

#### 6.Any 类型 和 unknown 顶级类型

```tsx

//any
let a1:any='123';
a1=123;
a1=true;

let a2;
a2=123;
a2=true;

//unknown
let value:unknown;
value=true;
value=42;
value='123'
value=[ ];
value={ };
value=null;
value=undefined;
value=undefined;
value=Symbol('test');
// unknow类型不能作为子类型只能作为父类型 any可以作为父类型和子类型

//unknown类型不能赋值给其他类型
let uk1:unknown='123';
// let uk3:string=uk1; //error

//any类型是可以的
let an1:any='123';
let an2:string=an1; 


let uk3:unknown='123';
let an3:any='321';

//unknown可赋值对象只有unknown 和 any
let uk4:unknown=an3;
let an4:any=uk3;
an3=uk3;

//如果是any类型在对象没有这个属性的时候还在获取是不会报错的
let an5:any={a:1};
an5.b;

// 如果是unknow 是不能调用属性和方法
let uk5:unknown={a:1,ccc:():number=>123};
// uk5.b; //error
// uk5.ccc(); /error


```

> **tips:**
>
> 1.  unknow类型不能作为子类型只能作为父类型,any可以作为父类型和子类型
> 2.  unknown可赋值对象只有unknown 和 any

### 接口和对象类型

在[typescript](https://so.csdn.net/so/search?from=pc_blog_highlight&q=typescript)中，我们定义对象的方式要用关键字**interface**（接口），我的理解是使用**interface**来定义一种约束，让数据的结构满足约束的格式。定义方式如下：

```tsx
interface Person {
    name: string;
    age: number;
}

//这样写是会报错的 因为我们在person定义了name，age但是对象里面缺少age属性
//使用接口约束的时候不能多一个属性也不能少一个属性
//必须与接口保持一致
// const ErrorPerson:Person={
//     name:'张三',
// }

const person:Person={
    name:'张三',
    age:18
}

```



```tsx
//重名interface  可以合并
interface A {
    name: string;
}
interface A {
    age: string;
}

var a:A={
    name:'张三',
    age:'18'
}
```



#### 继承

```tsx
//继承
interface B extends A {
    sex: string;
}

var b:B={
    name:'张三',
    age:'18',
    sex:'男'
}

```

#### 可选属性

```tsx
//可选属性 使用?操作符
interface C {
    b?:string,
    a:string
}

const c:C  = {
    a:"213"
}
```



#### 任意属性

```tsx
//任意属性 [propName: string]
// 需要注意的是，一旦定义了任意属性，那么确定属性和可选属性的类型都必须是它的类型的子集：

interface D1 {
    name: string
    age: number
    [propName: string]: any
  }
  
const d1:D1  = {
    name:"123",
    age:111,
    sex:"男"
}

//下面这种写法的对象的所有属性的得变成string
// interface D1 {
//     name: string
//     age: number
//     [propName: string]: string
//   }
```



#### 只读属性

```tsx
//只读属性 readonly
//readonly 只读属性是不允许被赋值的只能读取
interface E {
    name?: string
    readonly age: string
    [propName: string]: any
  }

// const e:E = {
//     name:"123",
//     age:111, age是只读的不允许重新赋值
//     sex:"男"
// }

```

添加函数

```tsx
// 添加函数
interface F {
    name?: string
    cb:()=>string
  }

let f:F = {
    name:"123",
    cb:()=>{
        return 'function'
    }
}
```



### 数组类型

```tsx

//类型加中括号
let arr1:number[]=[1,2,3]
//这样会报错定义了数字类型出现字符串是不允许的
// let arr2:number[] = [1,2,3,'1'] //error
//操作方法添加也是不允许的
let arr2:number[] = [1,2,3,]
// arr.unshift('1') //error


var arr3: number[] = [1, 2, 3]; //数字类型的数组
var arr4: string[] = ["1", "2"]; //字符串类型的数组
var arr5: any[] = [1, "2", true]; //任意类型的数组

// 数组泛型
let arr6:Array<number>=[1,2,3,4,5]
let arr7:Array<string>=['1','3','4']

//用接口表示数组
interface arr7{
    [index:number]:number
}

let arrs: arr7 = [1, 1, 2, 3, 5];
// 表示：只要索引的类型是数字时，那么值的类型必须是数字。

// 多维数组
let data:number[][]=[[1,1], [2,2],[3,3]]

// arguments类数组
//error
// function Arr(...args:any):void{
//     console.log(arguments)
//     //错误的arguments 是类数组不能这样定义
//     let arr:number[] = arguments
// }


function Arr(...args:any): void {
    console.log(arguments) 
    //ts内置对象IArguments 定义
    let arr:IArguments = arguments
}
Arr(111, 222, 333)

//其中 IArguments 是 TypeScript 中定义好了的类型，它实际上就是：
interface IArguments {
    [index: number]: any;
    length: number;
    callee: Function;
    }


// any 在数组中的应用
let list:any[ ]=['a',1,true]

```



### 函数类型

```
// 注意，参数不能多传，也不能少传 必须按照约定的类型来
const fn1 =(name:string,age:number):string=>{
    return `姓名是${name}，年龄是${age}`
}
var r1=fn1('张三',18)
console.log(r1)

// 函数的可选参数?
const fn2 =(name:string,age:number,sex?:string):string=>{
    return `姓名是${name}，年龄是${age}+${sex}`
}
var r2=fn2('张三',18)
console.log(r2)

// 函数参数的默认值
const fn3 =(name:string,age:number,sex:string='男'):string=>{
    return `姓名是${name}，年龄是${age}${sex}`
}
var r3=fn3('张三',18)
console.log(r3)

const fn4 = (name: string = "我是默认值"): string => {
    return name
}
var r4=fn4()
console.log(r4)

// 接口定义函数
interface Add{
    (a: number, b: number):number
}
const add: Add = (a, b) => {
    return a + b
}

console.log(add(1, 2))

interface User{
    name: string,
    age: number,
    sex: string
    pwd: string
}
function getUserInfo(user:User):User{
    return user;
}


//定义剩余参数
const fn5 = (array:number[],...items:any[]):any[] => {
    console.log(array,items)
    return items
}

let a:number[] = [1,2,3]

fn5(a,'4','5','6')

// 函数重载
function fn(params: number): void
function fn(params: string, params2: number): void
function fn(params: any, params2?: any): void {
    console.log(params)
    console.log(params2)
 
}

fn(123)
fn('123',456)
// 重载是方法名字相同，而参数不同，返回类型可以相同也可以不同。
// 如果参数类型不同，则参数类型应设置为 any。
// 参数数量不同你可以将不同的参数设置为可选。
```

### 联合类型 | 交叉类型 | 类型断言   

#### 联合类型

```
//例如我们的手机号通常是13XXXXXXX 为数字类型 这时候产品说需要支持座机
//所以我们就可以使用联合类型支持座机字符串
let phone1: number | string  = '010-820'
 
 
//这样写是会报错的应为我们的联合类型只有数字和字符串并没有布尔值
// let phone2: number | string  = true //error

// 函数使用联合类型
const fn1=(sth:number|boolean):boolean=>{
    return !!sth
}

console.log(fn1(true)); // 输出: true
console.log(fn1(false)); // 输出: false
console.log(fn1(1)); // 输出: true
console.log(fn1(0)); // 输出: false
```



#### 交叉类型

多种类型的集合，联合对象将具有所联合类型的所有成员

```
interface People {
    age: number,
    height: number
  }
  interface Man{
    sex: string
  }
const ehzyil= (people: People & Man)=>{
  console.log(people)
}
ehzyil({age: 18,height: 180,sex: 'male'});
```

#### 类型断言 

```
语法：　　值 as 类型　　或　　<类型>值  value as string  <string>value
```

```
interface A {
    run: string
}

interface B {
    build: string
}

// const fn = (type: A | B): string => {
//     return type.run //error
// }
//这样写是有警告的应为B的接口上面是没有定义run这个属性的


interface A1 {
    run: string
}

interface B1 {
    build: string
}

const fn = (type: A1 | B1): string => {
    return (type as A1).run
}
//可以使用类型断言来推断他传入的是A接口的值

// 使用any临时断言
// window.abc = 123 //error
//这样写会报错因为window没有abc这个东西
(window as any).abc = 123
//可以使用any临时断言在 any 类型的变量上，访问任何属性都是允许的。

//as const

const names = 'abc'
names = 'aa' //无法修改 error
 
let names2 = 'abc' as const
names2 = 'aa' //无法修改 error

// 数组
let a1 = [10, 20] as const;
const a2 = [10, 20];

a1.unshift(30); // 错误，此时已经断言字面量为[10, 20],数据无法做任何修改
a2.unshift(30); // 通过，没有修改指针
```

需要注意的是，类型断言只能够「欺骗」TypeScript 编译器，无法避免运行时的错误，反而滥用类型断言可能会导致运行时错误；

**类型断言是不具影响力的**

 在下面的例子中，将 something 断言为 boolean 虽然可以通过编译，但是并没有什么用 并不会影响结果, 因为编译过程中会删除类型断言

```
function toBoolean(something: any): boolean {
    return something as boolean;
}
 
toBoolean(1);
// 返回值为 1
//
```



### 内置对象

#### ECMAScript 的内置对象

**`Boolean`、Number、`string`、`RegExp`、`Date`、`Error`**

```tsx
let b:Boolean=new Boolean(true);
console.log(b);

let n:Number=new Number(10);
console.log(n);

let s:String=new String("Hello");
console.log(s);

let d:Date=new Date();
console.log(d);

let a:Array<number>=[1,2,3,4,5];
console.log(a);

let r:RegExp=new RegExp("a","g");
console.log(r);

let e:Error=new Error("Error");
console.log(e);
```

#### DOM 和 BOM 的内置对象

**`Document`、`HTMLElement`、`Event`、`NodeList` 等**

```tsx
let body: HTMLElement=document.body;

let allDivs: NodeList =document.querySelectorAll("div");

//读取div 这种需要类型断言 或者加个判断应为读不到返回null
let div: HTMLElement =document.querySelector("div") as HTMLDivElement;
document.addEventListener("click",function(e:MouseEvent){
    console.log(e.target);
    });


let div:NodeListOf<HTMLDivElement| HTMLElement> = document.querySelectorAll("div");
let lo:Location=location;
let local:Storage=localStorage;

```



**dom元素的映射表**

* `HTMLElement`：所有 HTML 元素的基类。
* `HTMLAnchorElement`：`<a>` 元素（超链接）。
* `HTMLAreaElement`：`<area>` 元素（图像地图中的区域）。
* `HTMLAudioElement`：`<audio>` 元素（音频）。
* `HTMLBaseElement`：`<base>` 元素（指定文档的基 URL）。
* `HTMLBodyElement`：`<body>` 元素（文档的主体）。
* `HTMLBRElement`：`<br>` 元素（换行）。
* `HTMLButtonElement`：`<button>` 元素（按钮）。
* `HTMLCanvasElement`：`<canvas>` 元素（用于绘制图形和图像）。
* `HTMLDataElement`：`<data>` 元素（定义可机读的数据）。
* `HTMLDataListElement`：`<datalist>` 元素（提供预定义选项列表）。
* `HTMLDetailsElement`：`<details>` 元素（可折叠的详细信息）。
* `HTMLDialogElement`：`<dialog>` 元素（模态对话框）。
* `HTMLDivElement`：`<div>` 元素（通用容器）。
* `HTMLEmbedElement`：`<embed>` 元素（嵌入外部内容）。
* `HTMLFieldSetElement`：`<fieldset>` 元素（一组相关控件）。
* `HTMLFontElement`：`<font>` 元素（已弃用，用于设置字体）。
* `HTMLFormElement`：`<form>` 元素（用于收集用户输入）。
* `HTMLFrameElement`：`<frame>` 元素（内联框架）。
* `HTMLFrameSetElement`：`<frameset>` 元素（一组内联框架）。
* `HTMLHeadElement`：`<head>` 元素（文档的头部）。
* `HTMLHeadingElement`：`<h1>` 到 `<h6>` 元素（标题）。
* `HTMLHRElement`：`<hr>` 元素（水平线）。
* `HTMLHtmlElement`：`<html>` 元素（文档的根元素）。
* `HTMLIFrameElement`：`<iframe>` 元素（内联框架）。
* `HTMLImageElement`：`<img>` 元素（图像）。
* `HTMLInputElement`：`<input>` 元素（用于收集用户输入）。
* `HTMLKeygenElement`：`<keygen>` 元素（生成密钥对）。
* `HTMLLabelElement`：`<label>` 元素（用于关联控件和标签）。
* `HTMLLegendElement`：`<legend>` 元素（`<fieldset>` 元素的标题）。
* `HTMLLIElement`：`<li>` 元素（列表项）。
* `HTMLLinkElement`：`<link>` 元素（用于链接外部资源）。
* `HTMLMapElement`：`<map>` 元素（定义图像地图）。
* `HTMLMarqueeElement`：`<marquee>` 元素（已弃用，用于滚动文本）。
* `HTMLMediaElement`：所有媒体元素（`<audio>`, `<video>`, `<track>`, `<source>`) 的基类。
* `HTMLMenuElement`：`<menu>` 元素（上下文菜单）。
* `HTMLMetaElement`：`<meta>` 元素（用于提供文档元数据）。
* `HTMLMeterElement`：`<meter>` 元素（测量值指示器）。
* `HTMLModElement`：`<mod>` 元素（已弃用，用于指示文本的修改）。
* `HTMLOListElement`：`<ol>` 元素（有序列表）。
* `HTMLObjectElement`：`<object>` 元素（嵌入外部内容）。
* `HTMLOptGroupElement`：`<optgroup>` 元素（`<select>` 元素中的选项组）。
* `HTMLOptionElement`：`<option>` 元素（`<select>` 元素中的选项）。
* `HTMLOutputElement`：`<output>` 元素（用于显示结果）。
* `HTMLParagraphElement`：`<p>` 元素（段落）。
* `HTMLParamElement`：`<param>` 元素（用于 `<object>` 元素指定参数）。
* `HTMLPictureElement`：`<picture>` 元素（用于响应式图像）。
* `HTMLPreElement`：`<pre>` 元素（预格式化文本）。
* `HTMLProgressElement`：`<progress>` 元素（进度条）。
* `HTMLQuoteElement`：`<q>` 和 `<blockquote>` 元素（引用）。
* `HTMLScriptElement`：`<script>` 元素（用于执行脚本）。
* `HTMLSelectElement`：`<select>` 元素（下拉列表）。
* `HTMLSlotElement`：`<slot>` 元素（用于 Web 组件的插槽）。
* `HTMLSourceElement`：`<source>` 元素（用于 `<video>` 和 `<audio>` 元素指定媒体源）。
* `HTMLSpanElement`：`<span>` 元素（内联容器）。
* `HTMLStyleElement`：`<style>` 元素（用于定义样式）。
* `HTMLTableCaptionElement`：`<caption>` 元素（表格标题）。
* `HTMLTableCellElement`：`<td>` 和 `<th>` 元素（表格单元格）。
* `HTMLTableColElement`：`<col>` 和 `<colgroup>` 元素（表格列）。
* `HTMLTableHeadElement`：`<thead>` 元素（表格头部）。
* `HTMLTableRowElement`：`<tr>` 元素（表格行）。
* `HTMLTableSectionElement`：`<tbody>`, `<tfoot>`, `<thead>` 元素（表格部分）。
* `HTMLTemplateElement`：`<template>` 元素（用于定义可重用的内容模板）。
* `HTMLTextAreaElement`：`<textarea>` 元素（多行文本输入）。
* `HTMLTimeElement`：`<time>` 元素（表示日期和时间）。
* `HTMLTitleElement`：`<title>` 元素（文档的标题）。
* `HTMLTrackElement`：`<track>` 元素（用于 `<video>` 和 `<audio>` 元素指定字幕或音轨）。
* `HTMLUListElement`：`<ul>` 元素（无序列表）。
* `HTMLUnknownElement`：用于未知或自定义元素。
* `HTMLVideoElement`：`<video>` 元素（视频）。

**SVG 元素**

* `SVGElement`：所有 SVG 元素的基类。
* `SVGAElement`：`<a>` 元素（SVG 超链接）。
* `SVGAltGlyphElement`：`<altGlyph>` 元素（备用字形）。
* `SVGAltGlyphDefElement`：`<altGlyphDef>` 元素（备用字形定义）。
* `SVGAltGlyphItemElement`：`<altGlyphItem>` 元素（备用字形项）。
* `SVGCircleElement`：`<circle>` 元素（圆形）。
* `SVGClipPathElement`：`<clipPath>` 元素（剪切路径）。
* `SVGCursorElement`：`<cursor>` 元素（光标）。
* `SVGDefsElement`：`<defs>` 元素（定义）。
* `SVGDescElement`：`<desc>` 元素（描述）。
* `SVGEllipseElement`：`<ellipse>` 元素（椭圆）。
* `SVGFEBlendElement`：`<feBlend>` 元素（混合滤镜）。
* `SVGFEColorMatrixElement`：`<feColorMatrix>` 元素（颜色矩阵滤镜）。
* `SVGFEComponentTransferElement`：`<feComponentTransfer>` 元素（分量传输滤镜）。
* `SVGFECompositeElement`：`<feComposite>` 元素（复合滤镜）。
* `SVGFEConvolveMatrixElement`：`<feConvolveMatrix>` 元素（卷积矩阵滤镜）。
* `SVGFEDiffuseLightingElement`：`<feDiffuseLighting>` 元素（漫射光照滤镜）。
* `SVGFEDisplacementMapElement`：`<feDisplacementMap>` 元素（位移贴图滤镜）。
* `SVGFEDistantLightElement`：`<feDistantLight>` 元素（远光源）。
* `SVGFEDropShadowElement`：`<feDropShadow>` 元素（阴影滤镜）。
* `SVGFEFloodElement`：`<feFlood>` 元素（泛光滤镜）。
* `SVGFEFuncAElement`：`<feFuncA>` 元素（Alpha 分量函数）。
* `SVGFEFuncBElement`：`<feFuncB>` 元素（B 分量函数）。
* `SVGFEFuncGElement`：`<feFuncG>` 元素（G 分量函数）。
* `SVGFEFuncRElement`：`<feFuncR>` 元素（R 分量函数）。
* `SVGFEGaussianBlurElement`：`<feGaussianBlur>` 元素（高斯模糊滤镜）。
* `SVGFEImageElement`：`<feImage>` 元素（图像滤镜）。
* `SVGFEMergeElement`：`<feMerge>` 元素（合并滤镜）。
* `SVGFEMergeNodeElement`：`<feMergeNode>` 元素（合并滤镜节点）。
* `SVGFEMorphologyElement`：`<feMorphology>` 元素（形态学滤镜）。
* `SVGFEOffsetElement`：`<feOffset>` 元素（偏移滤镜）。
* `SVGFEPointLightElement`：`<fePointLight>` 元素（点光源）。
* `SVGFESpecularLightingElement`：`<feSpecularLighting>` 元素（镜面光照滤镜）。
* `SVGFESpotLightElement`：`<feSpotLight>` 元素（聚光灯）。
* `SVGFETileElement`：`<feTile>` 元素（平铺滤镜）。
* `SVGFETurbulenceElement`：`<feTurbulence>` 元素（湍流滤镜）。
* `SVGFilterElement`：`<filter>` 元素（滤镜）。
* `SVGForeignObjectElement`：`<foreignObject>` 元素（外部对象）。
* `SVGGElement`：`<g>` 元素（组）。
* `SVGImageElement`：`<image>` 元素（图像）。
* `SVGLineElement`：`<line>` 元素（线）。
* `SVGLinearGradientElement`：`<linearGradient>` 元素（线性渐变）。
* `SVGMarkerElement`：`<marker>` 元素（标记）。
* `SVGMaskElement`：`<mask>` 元素（遮罩）。
* `SVGMetadataElement`：`<metadata>` 元素（元数据）。
* `SVGPathElement`：`<path>` 元素（路径）。
* `SVGPolygonElement`：`<polygon>` 元素（多边形）。
* `SVGPolylineElement`：`<polyline>` 元素（折线）。
* `SVGRadialGradientElement`：`<radialGradient>` 元素（径向渐变）。
* `SVGRectElement`：`<rect>` 元素（矩形）。
* `SVGSVGElement`：`<svg>` 元素（SVG 根元素）。
* `SVGScriptElement`：`<script>` 元素（SVG 脚本）。
* `SVGStopElement`：`<stop>` 元素（渐变停止）。
* `SVGStyleElement`：`<style>` 元素（SVG 样式）。
* `SVGSwitchElement`：`<switch>` 元素（开关）。
* `SVGSymbolElement`：`<symbol>` 元素（符号）。
* `SVGTextElement`：`<text>` 元素（文本）。
* `SVGTextPathElement`：`<textPath>` 元素（文本路径）。
* `SVGTitleElement`：`<title>` 元素（标题）。
* `SVGTspanElement`：`<tspan>` 元素（文本跨度）。
* `SVGUseElement`：`<use>` 元素（使用）。
* `SVGViewElement`：`<view>` 元素（视图）。

**其他元素**

* `Document`：文档对象。
* `DocumentFragment`：文档片段。
* `DocumentType`：文档类型声明。
* `Element`：所有 HTML 和 SVG 元素的基类。
* `Node`：所有 DOM 节点的基类。
* `ShadowRoot`：Web 组件的影子 DOM。
* `Window`：浏览器窗口。
* `XMLDocument`：XML 文档。

#### 定义Promise

```tsx
//如果我们不指定返回的类型TS是推断不出来返回的是什么类型
function promise():Promise<string>{

  return new Promise<string>((resolve,reject)=>{
    resolve("hello world")
  })
}

promise().then((data)=>{
  console.log(data)
})
```



#### 代码雨案例

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>代码雨案例</title>
        <meta charset="UTF-8">
        <style>
            * {
                padding: 0;
                margin: 0;
                overflow: hidden;
            }
        </style>
    </head>
    <body>
        <canvas id="canvas"></canvas>
        <script src="./index.js"></script>
    </body>
</html>
```

```tsx
// 获取页面上id为"canvas"的HTMLCanvasElement元素，并将其类型断言为HTMLCanvasElement
let canvas: HTMLCanvasElement = document.querySelector("#canvas") as HTMLCanvasElement;

// 从canvas元素获取2D渲染上下文，并将其类型断言为CanvasRenderingContext2D
let ctx: CanvasRenderingContext2D = canvas.getContext("2d") as CanvasRenderingContext2D;

// 设置canvas元素的高度和宽度分别为屏幕可用高度和宽度
canvas.height = screen.availHeight;
canvas.width = screen.availWidth;

// 定义一个字符串，包含要随机显示的字符
let str: string = "abcdefghigklmn";

// 创建一个数组，长度为canvas宽度除以10后的向上取整数，所有元素初始化为0
let arr: number[] = Array(Math.ceil(canvas.width / 10)).fill(0);

// 输出初始的arr数组内容和2D渲染上下文对象
console.log(arr);
console.log(ctx);

// 定义绘制函数
const draw = (): void => {
    // 设置背景填充颜色为半透明黑色
    ctx.fillStyle = 'rgba(0,0,0,0.05)';
    // 填充整个canvas区域作为背景
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // 设置文字颜色为绿色
    ctx.fillStyle = "#0f0";

    // 遍历数组，根据数组中的每个值绘制随机字符
    arr.forEach((item, index) => {
        // 从str中随机选择一个字符
        let randomCharIndex = Math.floor(Math.random() * str.length);
        // 在canvas上指定位置绘制字符
        ctx.fillText(str[randomCharIndex], index * 10, item + 10);

        // 更新数组元素值，使其在canvas高度范围内随机分布（增加一定的随机性）
        arr[index] = item >= canvas.height || item > 20000 * Math.random() ? 0 : item + 10;
    });
}

// 每隔50毫秒调用一次draw函数，实现动态绘制效果
setInterval(draw, 50);


```



### Class类

ES6提供了更接近传统语言的写法，引入了Class（类）这个概念，作为对象的模板。通过class关键字，可以定义类。基本上，ES6的class可以看作只是一个语法糖，它的绝大部分功能，ES5都可以做到，新的class写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。





**TypeScript 类**

TypeScript 类是一种创建对象的蓝图。它定义了对象的属性和方法。

**语法**

```typescript
class ClassName {
  // 成员变量
  private name: string;

  // 构造函数
  constructor(name: string) {
    this.name = name;
  }

  // 成员方法
  public greet(): void {
    console.log(`Hello, ${this.name}!`);
  }
}
```

**成员变量**

成员变量是类的属性。它们存储有关对象状态的信息。在上面的示例中，`name` 是一个私有成员变量，这意味着它只能在类内部访问。

**构造函数**

构造函数是类的一个特殊方法，它在创建类的新实例时被调用。它用于初始化成员变量。在上面的示例中，构造函数接受一个字符串参数，并将该参数分配给 `name` 成员变量。

**成员方法**

成员方法是类的函数。它们用于操作对象的状态或执行其他任务。在上面的示例中，`greet` 方法是一个公共成员方法，这意味着它可以在类外部访问。它打印一条包含对象 `name` 属性的消息。

**类实例**

要创建类的实例，请使用 `new` 关键字：

```typescript
const person = new ClassName("John");
```

这将创建一个 `ClassName` 类的实例，并将 `name` 成员变量初始化为 "John"。

**访问成员**

可以使用点语法访问类的成员：

* 访问成员变量：`person.name`
* 调用成员方法：`person.greet()`

#### **继承**

TypeScript 类支持继承，这允许你创建从现有类派生的新类。派生类继承基类的所有成员，并可以添加自己的成员。

**语法**

```typescript
class DerivedClass extends BaseClass {
  // 派生类特有的成员
}
```

**示例**

```typescript
class Animal {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  speak(): void {
    console.log("Animal is speaking");
  }
}

class Dog extends Animal {
  breed: string;

  constructor(name: string, breed: string) {
    super(name); // 调用基类构造函数
    this.breed = breed;
  }

  bark(): void {
    console.log("Dog is barking");
  }
}

const dog = new Dog("Buddy", "Golden Retriever");
dog.speak(); // 继承自基类
dog.bark(); // 派生类特有的方法
```

####  **抽象类**

抽象类是不能被实例化的类。它们用于定义接口，子类必须实现这些接口。

**语法**

```typescript
abstract class AbstractClass {
  abstract method(): void;
}
```

**示例**

```typescript
abstract class Shape {
  abstract area(): number;
}

class Circle extends Shape {
  radius: number;

  constructor(radius: number) {
    super();
    this.radius = radius;
  }

  area(): number {
    return Math.PI * this.radius ** 2;
  }
}
```

#### **接口定义类**

接口是定义一组方法签名的契约。类必须实现接口中定义的所有方法才能被认为实现了该接口。

**语法**

```typescript
interface InterfaceName {
  method(): void;
}
```

**示例**

```typescript
interface Drawable {
  draw(): void;
}

class Rectangle implements Drawable {
  width: number;
  height: number;

  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }

  draw(): void {
    console.log("Drawing a rectangle");
  }
}
```



#### **修饰符**

修饰符用于控制类、接口、方法和属性的访问权限。它们可以指定成员是公有的、私有的、受保护的或只读的。

**访问权限级别**

TypeScript 中有四种访问权限级别：

* **public：**成员可以在任何地方访问。
* **protected：**成员只能在类本身及其派生类中访问。
* **private：**成员只能在类本身内部访问。
* **readonly：**成员只能在声明时初始化，之后不能修改。

**修饰符语法**

修饰符放在成员声明之前：

* **public：**无修饰符
* **protected：**`protected`
* **private：**`private`
* **readonly：**`readonly`

**示例**

```typescript
class Person {
  public name: string; // 公有属性
  protected age: number; // 受保护属性
  private address: string; // 私有属性
  readonly id: number; // 只读属性
}
```

**注意：**

* 修饰符不能用于接口方法，因为接口方法始终是公有的。
* 修饰符不能用于枚举成员，因为枚举成员始终是公有、静态和常量的。



#### **`static` 关键字**

`static` 关键字用于声明类中的静态成员。静态成员属于类本身，而不是类的实例。

**静态成员类型**

静态成员可以是：

* **属性：**存储与类本身相关的数据。
* **方法：**执行与类本身相关的操作。

**静态成员的优点**

* **类级作用域：**静态成员可以在不创建类实例的情况下访问和使用。
* **代码重用：**静态方法可以实现类级的功能，而无需创建实例。
* **内存效率：**静态成员只存储在类本身中，而不是在每个实例中，从而节省内存。

**静态成员的语法**

在成员声明之前使用 `static` 关键字：

```typescript
class MyClass {
  static property: number; // 静态属性
  static method(): void; // 静态方法
}
```

**访问静态成员**

可以使用类名直接访问静态成员：

```typescript
MyClass.property; // 访问静态属性
MyClass.method(); // 调用静态方法
```

**示例**

```typescript
class MathUtils {
  static PI: number = 3.14; // 静态属性

  static calculateArea(radius: number): number {
    return Math.PI * radius ** 2; // 静态方法
  }
}

console.log(MathUtils.PI); // 3.14
const area = MathUtils.calculateArea(5); // 78.5
```



### 元组类型

元组类型是一种有序集合类型，其中每个元素可以具有不同的类型。元组的长度和元素类型在编译时是固定的。

**语法**

```typescript
let tuple: [type1, type2, ..., typeN];
```

**示例**

```typescript
let employee: [string, number, boolean] = ["John Doe", 30, true];
```

在这个示例中，`employee` 元组包含三个元素：

* 第一个元素是一个字符串（姓名）。
* 第二个元素是一个数字（年龄）。
* 第三个元素是一个布尔值（是否已婚）。

**访问元组元素**

可以使用索引访问元组元素：

```typescript
console.log(employee[0]); // "John Doe"
console.log(employee[1]); // 30
console.log(employee[2]); // true
```

**解构元组**

也可以使用解构来提取元组元素：

```typescript
const [name, age, isMarried] = employee;

console.log(name); // "John Doe"
console.log(age); // 30
console.log(isMarried); // true
```

**示例**

元组的一个常见用例是表示一个二维点：

```typescript
type Point = [number, number];

const point: Point = [10, 20];

console.log(point[0]); // 10
console.log(point[1]); // 20
```

### 枚举类型

枚举类型是一种特殊的数据类型，它表示一组命名常量。枚举成员的值在编译时是固定的，并且通常是从 0 开始的整数。

**语法**

```typescript
enum EnumName {
  member1 = value1,
  member2 = value2,
  // ...
  memberN = valueN
}
```

**枚举类型**

TypeScript 支持以下类型的枚举：

* **数字枚举：**枚举成员是数字值。
* **字符串枚举：**枚举成员是字符串值。
* **异构枚举：**枚举成员可以是数字或字符串值。

**数字枚举**

数字枚举的成员值从 0 开始自动增长。但是，也可以显式指定成员值：

```typescript
enum Colors {
  Red = 1,
  Green = 2,
  Blue = 3
}
```

**字符串枚举**

字符串枚举的成员值必须是字符串字面量或其他字符串枚举成员：

```typescript
enum Colors {
  Red = 'red',
  Green = 'green',
  Blue = 'blue'
}
```

**异构枚举**

异构枚举可以混合数字和字符串成员：

```typescript
enum Types {
  No = "No",
  Yes = 1
}
```

**接口枚举**

枚举类型可以与接口一起使用来定义具有特定属性的对象：

```typescript
enum Types {
  yyds
}

interface A {
  red: Types.yyds
}

const obj: A = {
  red: Types.yyds
}
```

**常量枚举**

常量枚举使用 `const` 修饰符定义，并且编译为常量值：

```typescript
const enum Types {
  No = "No",
  Yes = 1
}
```

**反向映射**

数字枚举和异构枚举具有反向映射，它允许你根据值获取枚举成员的名称：

```typescript
enum Enum {
  fall
}

const a = Enum.fall; // 0
const nameOfA = Enum[a]; // "fall"
```

### 类型推论 | 类型别名

#### 类型推论

- 声明了一个变量,但是没有定义类型 TypeScript 会在没有明确的指定类型的时候推测出一个类型，这就是类型推论

  ```
  let str = "Hello World"; //TS帮我推断出来这是一个string类型
  
  str=135 //error 不能将类型“number”分配给类型“string”。
  ```

- 如果声明变量没有定义类型也没有赋值这时候TS会推断成any类型可以进行任何操作

```
let ehzyil;

ehzyil = 123;
ehzyil = "123";
ehzyil = true;
```

#### 类型别名

type 关键字（可以给一个类型定义一个名字）多用于复合类型

```
// 定义类型别名
type str = string
let s:str = "123"
console.log(s);

//  定义函数别名
type fn = () => string
let f: fn = () => "123"
console.log(s);

//  定义联合类型别名
type c = string | number
let c1: c = 123
let c2: c = '123'
console.log(c1 ,c2);

// 定义值的别名
type value = boolean | 0 | '213'
let v:value = true
//变量v的值  只能是上面value定义的值
```



type 和 interface 还是一些**区别**的 虽然都可以定义类型

1.interface可以继承  type 只能通过 & 交叉类型合并

2.type 可以定义 联合类型 和 可以使用一些操作符 interface不行

3.interface 遇到重名的会合并 type 不行



**type高级用法**

左边的值会作为右边值的子类型.

```
type a = 1 extends number ? 1 : 0 //1

type a = 1 extends Number ? 1 : 0 //1

type a = 1 extends Object ? 1 : 0 //1

type a = 1 extends any ? 1 : 0 //1

type a = 1 extends unknow ? 1 : 0 //1

type a = 1 extends never ? 1 : 0 //0
```



### never类型 	

`never` 类型表示一个永远不会发生的值。它用于表示永远不会返回或终止的函数或操作。

**语法**

```typescript
type never = never;
```

**特点**

* `never` 类型的值永远不会存在。
* 任何类型都可以赋值给 `never` 类型，但 `never` 类型的值不能赋值给任何其他类型。
* `never` 类型是任何其他类型的子类型。

**用法**

`never` 类型通常用于以下情况：

* 表示永远不会返回的函数，例如无限循环或抛出异常的函数。
* 表示永远不会终止的操作，例如死锁或未处理的 Promise。
* 作为类型系统中的哨兵值，表示无效或不可能的状态。

**示例**

**永远不会返回的函数：**

```typescript
const infiniteLoop = (): never => {
  while (true) {
    // 无限循环
  }
};
```

**永远不会终止的操作：**

```typescript
const deadLock = (): never => {
  const lock1 = new Lock();
  const lock2 = new Lock();

  lock1.acquire();
  lock2.acquire();

  // 死锁，因为无法同时获取两个锁
};
```

**作为哨兵值：**

```typescript
type State = "active" | "inactive" | "error";

const getState = (): State => {
  // ... 获取状态的逻辑

  if (// 发生错误) {
    return "error";
  } else {
    return "active"; // 永远不会返回 "inactive"
  }
};
```

**never 类型与其他类型的比较**

| 类型      | `never` 类型                                                 |
| --------- | ------------------------------------------------------------ |
| `void`    | `void` 是一个特殊情况，表示没有返回值的函数。`never` 类型表示永远不会返回或终止的函数。 |
| `unknown` | `never` 类型是 `unknown` 类型的子类型，因为任何值都可以赋值给 `never` 类型。 |
| `any`     | `never` 类型不是 `any` 类型的子类型，因为 `never` 类型的值永远不会存在。 |

**never 与 void 的差异**

```tsx

//差异1 void类型只是没有返回值 但本身不会出错
function Void():void {
    console.log();
}
    
//只会抛出异常没有返回值
function Never():never {
throw new Error('aaa')
}
//差异2 当我们鼠标移上去的时候会发现 只有void和number    never在联合类型中会被直接移除

type A = void | number | never
```
**注意事项**

* `never` 类型不能用于表示可选值。使用 `undefined` 或 `null` 来表示可选值。
* `never` 类型不能用于表示异步函数。使用 `Promise<never>` 来表示永远不会解析或拒绝的异步函数。

#### never 类型的一个应用场景

`kun` 函数使用 `never` 类型来表示一个兜底逻辑，即当输入值 `value` 不属于枚举类型 `bb` 中的任何一个值时执行的逻辑。

**具体来说，`never` 类型在这里的作用是：**

- **类型检查：**确保 `value` 变量只能取 `bb` 枚举类型中的值，否则会产生类型错误。
- **错误处理：**当 `value` 不是有效的枚举值时，函数会抛出一个类型为 `never` 的错误。这可以帮助我们检测和处理无效输入。
- **哨兵值：**`never` 类型作为一个哨兵值，表示一种不可能或无效的状态，在这种情况下，表示一个无效的输入值。

```tsx
type  bb = "唱" | "跳" | "rap" | "篮球"

function kun(value: bb): void {
  switch (value) {
    case "唱":
      console.log("唱歌");
      break;
    case "跳":
      console.log("跳舞");
      break;
    case "rap":
      console.log("饶舌");
      break;
    default:
      //兜底逻辑
      const error:never = value;
      return error       
      }
}
```



### Symbol类型

自ECMAScript 2015起，symbol成为了一种新的原生类型，就像number和string一样。
symbol类型的值是通过Symbol构造函数创建的。

**可以传递参做为唯一标识 只支持 string 和 number类型的参数**

```
let sym1 = Symbol();
let sym2 = Symbol("key"); // 可选的字符串key
```

**Symbol的值是唯一的**

```
const s1 = Symbol()
const s2 = Symbol()
// s1 === s2 =>false
console.log(s1 === Symbol()) // false
```

**用作对象属性的键**

```
let sym = Symbol();

let obj = {
    [sym]: "value"
};

console.log(obj[sym]); // "value"
```

**使用symbol定义的属性，是不能通过如下方式遍历拿到的**

```
const symbol1 = Symbol('666')
const symbol2 = Symbol('777')
const obj1= {
   [symbol1]: '小满',
   [symbol2]: '二蛋',
   age: 19,
   sex: '女'
}

// 1 for in 遍历
for (const key in obj1) {
   // 注意在console看key,是不是没有遍历到symbol1
   console.log(key)
//    console.log(obj1[key]) //error
}

// 2 Object.keys 遍历
Object.keys(obj1)
console.log(Object.keys(obj1))
// 3 getOwnPropertyNames
console.log(Object.getOwnPropertyNames(obj1))
// 4 JSON.stringfy
console.log(JSON.stringify(obj1))
```

**如何拿到**

```
// 1 拿到具体的symbol 属性,对象中有几个就会拿到几个
Object.getOwnPropertySymbols(obj1)
console.log(Object.getOwnPropertySymbols(obj1))
// 2 es6 的 Reflect 拿到对象的所有属性
Reflect.ownKeys(obj1)
console.log(Reflect.ownKeys(obj1))
```





```
interface Item {
    name: string
    age: number
}

const array: Array<Item> = [
    {
        name: "张三",
        age: 18
    },
    {
        name: "李四",
        age: 20
    }
]

type mapType = string | number
const map: Map<mapType, mapType> = new Map()
map.set("0", "张三")
map.set(1, "李四")
map.set(2, "王五")


const obj = {
    aaa: 123,
    bbb: 456
}


const gen = (args: any): void => {
    let it: Iterator<any> = args[Symbol.iterator]();

    let next: any = { done: false }

    while (!next.done) {
        next = it.next();
        if (!next.done) {
            console.log(next.value)
        }

    }
}

gen(array)
gen(map)
// gen(obj) //error obj 并不是一个可迭代对象
```

我们平时开发中不会手动调用iterator 应为 他是有语法糖的就是for of  记住 for of 是不能循环对象的因为对象没有 iterator

```
for (let value of array) {
    console.log(value)
}
```

我们可以自己实现一个迭代器让对象支持for of

```

const obj = {
    max: 5,
    current: 0,
    [Symbol.iterator]() {
        return {
            max: this.max,
            current: this.current,
            next() {
                if (this.current == this.max) {
                    return {
                        value: undefined,
                        done: true
                    }
                } else {
                    return {
                        value: this.current++,
                        done: false
                    }
                }
            }
        }
    }
}
console.log([...obj])

for (let val of obj) {
    console.log(val);

}
```

**众所周知的Symbols**

Symbol.hasInstance
方法，会被instanceof运算符调用。构造器对象用来识别一个对象是否是其实例。

Symbol.isConcatSpreadable
布尔值，表示当在一个对象上调用Array.prototype.concat时，这个对象的数组元素是否可展开。

Symbol.iterator
方法，被for-of语句调用。返回对象的默认迭代器。

Symbol.match
方法，被String.prototype.match调用。正则表达式用来匹配字符串。

Symbol.replace
方法，被String.prototype.replace调用。正则表达式用来替换字符串中匹配的子串。

Symbol.search
方法，被String.prototype.search调用。正则表达式返回被匹配部分在字符串中的索引。

Symbol.species
函数值，为一个构造函数。用来创建派生对象。

Symbol.split
方法，被String.prototype.split调用。正则表达式来用分割字符串。

Symbol.toPrimitive
方法，被ToPrimitive抽象操作调用。把对象转换为相应的原始值。

Symbol.toStringTag
方法，被内置方法Object.prototype.toString调用。返回创建对象时默认的字符串描述。

Symbol.unscopables
对象，它自己拥有的属性会被with作用域排除在外。



### 泛型



泛型允许你创建可重用的组件，这些组件可以在不修改源代码的情况下使用不同类型的数据。它们使代码更灵活、更可维护。

**语法**

```
function myFunction<T>(arg: T): T {
  // ...
}
```

在这个示例中，`T` 是一个类型变量，它可以被任何类型替换。这意味着 `myFunction` 可以接受和返回任何类型的数据

#### 函数泛型

我写了两个函数一个是数字类型的函数，另一个是字符串类型的函数,其实就是类型不同，

实现的功能是一样的，这时候我们就可以使用泛型来优化

```
function num (a:number,b:number) : Array<number> {
    return [a ,b];
}
num(1,2)
function str (a:string,b:string) : Array<string> {
    return [a ,b];
}
str('独孤','求败')
```

**泛型优化**

语法为函数名字后面跟一个<参数名> 参数名可以随便写 例如我这儿写了T

当我们使用这个函数的时候把参数的类型传进去就可以了 （也就是动态类型）

```
function Add<T>(a: T, b: T): Array<T>  {
    return [a,b]
}

Add<number>(1,2)
Add<string>('1','2')
```

我们也可以使用不同的泛型参数名，只要在数量上和使用方式上能对应上就可以。

```
function Sub<T,U>(a:T,b:U):Array<T|U> {
    const params:Array<T|U> = [a,b]
    return params
}


Sub<Boolean,number>(false,1)
```

#### 定义泛型接口

声明接口的时候 在名字后面加一个<参数>

使用的时候传递类型

```
interface MyInter<T> {
   (arg: T): T
}

function fn<T>(arg: T): T {
   return arg
}

let result: MyInter<number> = fn

result(123)
```

#### 对象字面量泛型

```
let foo: { <T>(arg: T): T }

foo = function <T>(arg:T):T {
   return arg
}

foo(123)
```



#### 泛型约束

我们期望在一个泛型的变量上面，获取其length参数，但是，有的数据类型是没有length属性的

```
function getLegnth<T>(arg:T) {
  return arg.length // 类型“T”上不存在属性“length”。
}
```

于是，我们就得对使用的泛型进行约束，我们约束其为具有length属性的类型，这里我们会用到interface,代码如下

```
interface Len {
   length:number
}

function getLegnth<T extends Len>(arg:T) {
  return arg.length
}

getLegnth<string>('123')
```

#### 使用keyof 约束对象

其中使用了TS泛型和泛型约束。首先定义了T类型并使用extends关键字继承object类型的子类型，然后使用keyof操作符获取T类型的所有键，它的返回 类型是联合 类型，最后利用extends关键字约束 K类型必须为keyof T联合类型的子类型

```
function prop<T, K extends keyof T>(obj: T, key: K) {
   return obj[key]
}


let o = { a: 1, b: 2, c: 3 }

prop(o, 'a') 
prop(o, 'd') //此时就会报错发现找不到
```



#### 泛型类

声明方法跟函数类似名称后面定义<类型>

使用的时候确定类型new Sub<number>()

```
class Sub<T>{
   attr: T[] = [];
   add (a:T):T[] {
      return [a]
   }
}

let s = new Sub<number>()
s.attr = [1,2,3]
s.add(123)

let str = new Sub<string>()
str.attr = ['1','2','3']
str.add('123')
```

### tsconfig.json配置文件

生成tsconfig.json 文件
这个文件是通过tsc --init命令生成的

配置详解

```
"compilerOptions": {
  "incremental": true, // TS编译器在第一次编译之后会生成一个存储编译信息的文件，第二次编译会在第一次的基础上进行增量编译，可以提高编译的速度
  "tsBuildInfoFile": "./buildFile", // 增量编译文件的存储位置
  "diagnostics": true, // 打印诊断信息 
  "target": "ES5", // 目标语言的版本
  "module": "CommonJS", // 生成代码的模板标准
  "outFile": "./app.js", // 将多个相互依赖的文件生成一个文件，可以用在AMD模块中，即开启时应设置"module": "AMD",
  "lib": ["DOM", "ES2015", "ScriptHost", "ES2019.Array"], // TS需要引用的库，即声明文件，es5 默认引用dom、es5、scripthost,如需要使用es的高级版本特性，通常都需要配置，如es8的数组新特性需要引入"ES2019.Array",
  "allowJS": true, // 允许编译器编译JS，JSX文件
  "checkJs": true, // 允许在JS文件中报错，通常与allowJS一起使用
  "outDir": "./dist", // 指定输出目录
  "rootDir": "./", // 指定输出文件目录(用于输出)，用于控制输出目录结构
  "declaration": true, // 生成声明文件，开启后会自动生成声明文件
  "declarationDir": "./file", // 指定生成声明文件存放目录
  "emitDeclarationOnly": true, // 只生成声明文件，而不会生成js文件
  "sourceMap": true, // 生成目标文件的sourceMap文件
  "inlineSourceMap": true, // 生成目标文件的inline SourceMap，inline SourceMap会包含在生成的js文件中
  "declarationMap": true, // 为声明文件生成sourceMap
  "typeRoots": [], // 声明文件目录，默认时node_modules/@types
  "types": [], // 加载的声明文件包
  "removeComments":true, // 删除注释 
  "noEmit": true, // 不输出文件,即编译后不会生成任何js文件
  "noEmitOnError": true, // 发送错误时不输出任何文件
  "noEmitHelpers": true, // 不生成helper函数，减小体积，需要额外安装，常配合importHelpers一起使用
  "importHelpers": true, // 通过tslib引入helper函数，文件必须是模块
  "downlevelIteration": true, // 降级遍历器实现，如果目标源是es3/5，那么遍历器会有降级的实现
  "strict": true, // 开启所有严格的类型检查
  "alwaysStrict": true, // 在代码中注入'use strict'
  "noImplicitAny": true, // 不允许隐式的any类型
  "strictNullChecks": true, // 不允许把null、undefined赋值给其他类型的变量
  "strictFunctionTypes": true, // 不允许函数参数双向协变
  "strictPropertyInitialization": true, // 类的实例属性必须初始化
  "strictBindCallApply": true, // 严格的bind/call/apply检查
  "noImplicitThis": true, // 不允许this有隐式的any类型
  "noUnusedLocals": true, // 检查只声明、未使用的局部变量(只提示不报错)
  "noUnusedParameters": true, // 检查未使用的函数参数(只提示不报错)
  "noFallthroughCasesInSwitch": true, // 防止switch语句贯穿(即如果没有break语句后面不会执行)
  "noImplicitReturns": true, //每个分支都会有返回值
  "esModuleInterop": true, // 允许export=导出，由import from 导入
  "allowUmdGlobalAccess": true, // 允许在模块中全局变量的方式访问umd模块
  "moduleResolution": "node", // 模块解析策略，ts默认用node的解析策略，即相对的方式导入
  "baseUrl": "./", // 解析非相对模块的基地址，默认是当前目录
  "paths": { // 路径映射，相对于baseUrl
    // 如使用jq时不想使用默认版本，而需要手动指定版本，可进行如下配置
    "jquery": ["node_modules/jquery/dist/jquery.min.js"]
  },
  "rootDirs": ["src","out"], // 将多个目录放在一个虚拟目录下，用于运行时，即编译后引入文件的位置可能发生变化，这也设置可以虚拟src和out在同一个目录下，不用再去改变路径也不会报错
  "listEmittedFiles": true, // 打印输出文件
  "listFiles": true// 打印编译的文件(包括引用的声明文件)
}

// 指定一个匹配列表（属于自动指定该路径下的所有ts相关文件）
"include": [
   "src/**/*"
],
// 指定一个排除列表（include的反向操作）
 "exclude": [
   "demo.ts"
],
// 指定哪些文件使用该配置（属于手动一个个指定文件）
 "files": [
   "demo.ts"
]
```

 **介绍几个常用的**

- 1.include
  指定编译文件默认是编译当前目录下所有的ts文件

- 2.exclude
  指定排除的文件
- 3.target
  指定编译js 的版本例如es5  es6

- 4.allowJS
  是否允许编译js文件

- 5.removeComments
  是否在编译过程中删除文件中的注释

- 6.rootDir
  编译文件的目录
- 7.outDir
  输出的目录

- 8.sourceMap
  代码源文件

- 9.strict
  严格模式

- 10.module
  默认common.js  可选es6模式 amd  umd 等





### 命名空间

命名空间是一种组织和封装相关代码的机制，它可以防止名称冲突并提高代码的可读性和可维护性。

TypeScript与ECMAScript 2015一样，任何包含顶级`import`或者`export`的文件都被当成一个模块。相反地，如果一个文件不带有顶级的`import`或者`export`声明，那么它的内容被视为全局可见的（因此对模块也是可见的）

**创建命名空间**

要创建命名空间，可以使用 `namespace` 关键字，后跟命名空间的名称：

```typescript
namespace MyNamespace {
  // 命名空间代码
}
```

**访问命名空间成员**

要访问命名空间中的成员，可以使用点运算符（`.`）：

```typescript
MyNamespace.myFunction();
```

命名空间中通过`export`将想要暴露的部分导出

如果不用export 导出是无法读取其值的

```tsx
namespace a {
    export const Time: number = 1000
    export const fn = <T>(arg: T): T => {
        return arg
    }
    fn(Time)
}
 
 
namespace b {
     export const Time: number = 1000
     export const fn = <T>(arg: T): T => {
        return arg
    }
    fn(Time)
}
 
a.Time
b.Time
```



**嵌套命名空间**

命名空间可以嵌套，这意味着一个命名空间可以包含另一个命名空间：

```typescript
namespace a {
    export namespace b {
        export class Vue {
            parameters: string
            constructor(parameters: string) {
                this.parameters = parameters
            }
        }
    }
}
 
let v = a.b.Vue
console.log(v) //[class Vue]
new v('1')
```

**默认命名空间**

TypeScript 中存在一个默认命名空间，它包含没有明确指定命名空间的代码。默认命名空间的名称是全局对象（通常是 `window`）。



**抽离命名空间**

```
a.ts
export namespace V {
    export const a = 1
}

b.ts
import {V} from '../observer/index' 
console.log(V);

 //{a:1}
```

**简化命名空间**

```
namespace A {
    export namespace B {
        export const C = 1;
    }
}
import x = A.B.C;
console.log(x); //1
```

**合并命名空间**

重名的命名空间会合并

```
namespace A {
    export namespace B {
        export const C = 1;
    }
}
namespace A {
    export namespace B {
        export const D = 1;
    }
}

console.log(A); //{ B: { C: 1, D: 1 } }
```

**命名空间和模块**

命名空间与模块类似，但它们之间存在一些关键区别：

* **作用域：** 命名空间是全局作用域的，这意味着它们可以在代码中的任何地方访问。模块是块作用域的，这意味着它们只能在定义它们的块内访问。
* **导入/导出：** 命名空间不需要导入或导出，而模块必须使用 `import` 和 `export` 语句进行导入或导出。
* **封装：** 命名空间不提供封装，而模块通过模块系统提供封装。

**使用命名空间的好处**

使用命名空间的主要好处包括：

* **防止名称冲突：** 命名空间有助于防止来自不同模块或库的名称冲突。
* **提高可读性：** 命名空间可以将相关代码组织到有意义的组中，从而提高代码的可读性。
* **可维护性：** 通过将代码组织到命名空间中，可以更容易地维护和更新代码。

### 前端模块化

在 ES6 模块化规范之前，前端模块化有以下规范：

* **CommonJS：**一种用于 Node.js 的模块化规范，使用 `require()` 和 `module.exports` 来导出和导入模块。

  ```js
  // 导入
  require("xxx");
  require("../xxx.js");
  // 导出
  exports.xxxxxx= function() {};
  module.exports = xxxxx;
  ```

* **AMD (Asynchronous Module Definition)：**一种用于在浏览器中加载异步模块的规范，使用 `define()` 和 `require()` 函数来定义和加载模块。

  ```js
  // 定义
  define("module", ["dep1", "dep2"], function(d1, d2) {...});
  // 加载模块
  require(["module", "../app"], function(module, app) {...});
  ```

* **CMD (Common Module Definition)：**类似于 AMD，但更适合于在服务器端加载模块。

  ```js
  define(function(require, exports, module) {
    var a = require('./a');
    a.doSomething();
    
    var b = require('./b');
    b.doSomething();
  });
  ```

* **UMD (Universal Module Definition)：**一种通用模块化规范，可以在浏览器和 Node.js 中使用。

  ```js
  (function (window, factory) {
      // 检测是不是 Nodejs 环境
  	if (typeof module === 'object' && typeof module.exports === "objects") {
          module.exports = factory();
      } 
  	// 检测是不是 AMD 规范
  	else if (typeof define === 'function' && define.amd) {
          define(factory);
      } 
  	// 使用浏览器环境
  	else {
          window.eventUtil = factory();
      }
  })(this, function () {
      //module ...
  });
  ```

* **SystemJS：**一种现代模块加载器，支持多种模块化规范，包括 ES6 模块。

这些规范允许前端开发者将代码分解为可重用的模块，从而提高代码的可维护性和可扩展性。然而，它们在语法和加载机制上存在差异，这给开发人员带来了额外的复杂性。

ES6 模块化规范（也称为 ECMAScript 模块）于 2015 年引入，它提供了一种更简单、更统一的方式来定义和加载模块。它使用 `import` 和 `export` 关键字，并且内置于现代浏览器和 Node.js 中，消除了对外部模块加载器的需要。

ES6 模块化规范出来之后上面这些模块化规范就用的比较少了,现在主要使用 import export .

**es6模块化规范用法**

1.默认导出和引入

默认导出可以导出任意类型，这儿举例导出一个对象，并且默认导出只能有一个

引入的时候名字可以随便起

```
test.ts
//导出
export default {
    a:1,
}
//引入
import test from "./test";
```

 2.分别导出

```
//导出
index.ts
export default {
    a: 1,
    b: 2,
}

export function add<T extends number>(a: T, b: T) {
    return a + b
}

export let xxx = "xxx"

//引入

import obj, { xxx, add } from './index'

var b = add(1, 2);

console.log(obj);
console.log(xxx);
console.log(b);
```

3.重名问题 如果 导入的时候叫add但是已经有变量占用了可以用as重命名

```
import obj, { xxx as str, add } from './index'

console.log(str);
```

 4.动态引入

import只能写在顶层，不能掺杂到逻辑里面，这时候就需要动态引入了

```
if(true){
    import('./test').then(res => {
        console.log(res)
    })
}
```

### 声明文件d.ts

声明文件（也称为类型声明文件）是 TypeScript 中以 `.d.ts` 为扩展名的文件。它们包含有关 JavaScript 库、模块或 API 的类型信息，而无需实际实现这些库、模块或 API。

声明文件允许 TypeScript 编译器检查使用这些库、模块或 API 的 TypeScript 代码的类型正确性。它们提供有关函数、类、接口和变量的类型信息，从而使编译器可以确保代码中使用的类型与声明文件中定义的类型兼容。

**当使用第三方库时，我们需要引用它的声明文件，才能获得对应的代码补全、接口提示等功能。**

```tsx
declare var 声明全局变量
declare function 声明全局方法
declare class 声明全局类
declare enum 声明全局枚举类型
declare namespace 声明（含有子属性的）全局对象
interface 和 type 声明全局类型
/// <reference /> 三斜线指令
```



**用途**

声明文件用于以下用途：

- **类型检查：**允许 TypeScript 编译器检查代码中使用的类型的正确性，即使这些类型来自外部库或 API。
- **自动完成：**在使用外部库或 API 时提供代码自动完成和 IntelliSense。
- **重构：**允许重构工具（例如 Visual Studio Code）重构使用外部库或 API 的代码。
- **文档：**提供有关外部库或 API 的类型信息的文档。

**创建声明文件**

可以使用以下方法创建声明文件：

- **手动创建：**手动编写声明文件，并确保它们与外部库或 API 的实际实现兼容。
- **使用工具：**使用诸如 DefinitelyTyped 之类的工具自动生成声明文件。
- **从现有代码生成：**使用诸如 dtsgenerator 之类的工具从现有 JavaScript 代码生成声明文件。

**案例**

初始化项目后，安装axios和express

```bash
PS D:\Study\TS> npm i express
added 64 packages in 2s
PS D:\Study\TS> npm i axios
added 7 packages in 1s
```

在Index.ts中引入两个库

```tsx
import axios from "axios";
import express from "express"; //error
```

>  报错显示：
>
> 已声明“express”，但从未读取其值。ts(6133)
>  无法找到模块“express”的声明文件。“d:/Study/TS/node_modules/express/index.js”隐式拥有 "any" 类型。
>  尝试使用 `` (如果存在)，或者添加一个包含 `declare module 'express';` 的新声明(.d.ts)文件ts(7016)



让我们去下载他的声明文件`npm i --save-dev @types/express`

**那为什么axios 没有报错?**

我们可以去node_modules 下面去找axios 的package json

发现axios已经指定了声明文件 所以没有报错可以直接用

通过语法declare 暴露我们声明的axios 对象`declare const axios: AxiosStatic;`

如果有一些第三方包确实没有声明文件我们可以自己去定义名称.d.ts 创建一个文件去声明

**案例手写声明文件**

index.ts

```tsx
import axios from "axios";
import express from "express";

const app = express();
const router = express.Router();

app.use("/", router);

router.get("/", (req, res) => {
    res.json({
        code: 200,
        message: "Hello Worlld"
    });
});

app.listen(3000, () => {
    console.log("Server is running on port 3000");
});

fn("abc")

C.a
```

express.d.ts

```tsx
declare module "express" {
    interface Express {
        (): App;
        Router(): Router;
    }
    interface App {
        use: (path: string, router: any) => void;
        listen(port: number, cb?: () => void): void
    }
    interface Router {
        get: (path: string, cb: (req: any, res: any) => void) => void;
    }
    const express: Express;
    export default express;
}

declare var a: number;


declare function fn(param: type) {

}
declare class Vue { }

declare enum C {
    a,
    b
}
```


### Mixins混入（不熟悉）

Mixins 是 TypeScript 中的一种设计模式，它允许将多个类的功能组合到一个类中，而无需使用继承。这使得可以轻松地向现有类添加新功能，而无需修改其源代码。

**1.对象混入**
可以使用es6的Object.assign 合并多个对象

此时 people 会被推断成一个交差类型 Name & Age & sex;

```
interface Name {
    name: string
}
interface Age {
    age: number
}
interface Sex {
    sex: number
}

let people1: Name = { name: "小满" }
let people2: Age = { age: 20 }
let people3: Sex = { sex: 1 }

const people = Object.assign(people1,people2,people3)
```

**2.类的混入**
首先声明两个mixins类 （严格模式要关闭不然编译不过）=> ` "strict": false,  `

```
class A {
    type: boolean = false;
    changeType() {
        this.type = !this.type
    }
}


class B {
    name: string = '张三';
    getName(): string {
        return this.name;
    }
}
```


下面创建一个类，结合了这两个mixins

首先应该注意到的是，**没使用extends而是使用implements**。 把类当成了接口

我们可以这么做来达到目的，为将要mixin进来的属性方法创建出占位属性。 这告诉编译器这些成员在运行时是可用的。 这样就能使用mixin带来的便利，虽说需要提前定义一些占位属性

```
class C implements A,B{
    type:boolean
    changeType:()=>void;
    name: string;
    getName:()=> string
}
```

最后，创建这个帮助函数，帮我们做混入操作。 它会遍历mixins上的所有属性，并复制到目标上去，把之前的占位属性替换成真正的实现代码

Object.getOwnPropertyNames()可以获取对象自身的属性，除去他继承来的属性，
对它所有的属性遍历，它是一个数组，遍历一下它所有的属性名

```
Mixins(C, [A, B])
function Mixins(curCls: any, itemCls: any[]) {
    itemCls.forEach(item => {
        Object.getOwnPropertyNames(item.prototype).forEach(name => {
            curCls.prototype[name] = item.prototype[name]
        })
    })
}
```

### 装饰器Decorator

Decorator 装饰器是一项实验性特性，在未来的版本中可能会发生改变

它们不仅增加了代码的可读性，清晰地表达了意图，而且提供一种方便的手段，增加或修改类的功能

若要启用实验性的装饰器特性，你必须在命令行或`tsconfig.json`里启用编译器选项

```
    "experimentalDecorators": true,                  
    "emitDecoratorMetadata": true,  
```

#### 所有装饰器

**类装饰器：**

- `@ClassDecorator`：应用于类本身。

**方法装饰器：**

- `@MethodDecorator`：应用于类方法。
- `@Get`：应用于类访问器（getter）方法。
- `@Set`：应用于类访问器（setter）方法。

**属性装饰器：**

- `@PropertyDecorator`：应用于类属性。
- `@Field`：应用于类字段（仅限于使用 `--experimentalDecorators` 标志）。

**参数装饰器：**

- `@ParameterDecorator`：应用于类方法的参数。

**其他装饰器：**

- `@DecoratorFactory`：用于创建自定义装饰器的工厂装饰器。

**注意：**

- `@ClassDecorator`、`@MethodDecorator`、`@PropertyDecorator` 和 `@ParameterDecorator` 是 TypeScript 中内置的通用装饰器类型。
- `@Get`、`@Set` 和 `@Field` 是特定于特定库或框架的装饰器（例如 Angular 和 MobX）。
- `@DecoratorFactory` 是一个元装饰器，用于创建自定义装饰器。



#### 类装饰器 ClassDecorator

1. 首先定义一个类

   ```
   class A {
       constructor() {
           console.log('A');
       }
   }
   ```

   

2. 定义一个类装饰器函数 

   ```
   const Base: ClassDecorator = (target) => {
       //target是类名
       console.log("target:" + target);
       target.prototype.name = '测试类装饰器';
   }
   ```

   

3. 使用的时候 直接通过@函数名使用

   ```
   @Base
   class A {
     constructor() {
      console.log('A');
     }
   }
   ```

   

4. 验证

   ```
   const a = new A() as any;
   console.log(a.name); //测试类装饰器
   
   // target:class A {
   //     constructor() {
   //         console.log('A');
   //     }
   // }
   // A
   // 测试类装饰器
   ```

   

#### 装饰器工厂

其实也就是一个高阶函数 外层的函数接受值 里层的函数最终接受类的构造函数

```
const Base = (args: any) => {
    const fn: ClassDecorator = (target: any) => {
        console.log("target:" + target);
        console.log("args:" + args);
    }
    return fn;
}

@Base("装饰器工厂")
class A {

}

const a = new A() as any;
```



#### 属性装饰器 PropertyDecorator

**返回两个参数**

1. 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
2. 属性的名字。

```
const PropDecorator = (args: any) => {
    const fn: PropertyDecorator = (target, propertyKey) => {
        console.log(target);
        console.log(propertyKey); //属性名
        console.log("args:" + args); //参数
    }
    return fn;
}

class A {
    @PropDecorator("属性装饰器")
    name: string = "张三";
    constructor() {
        this.name = "张三";
    }
}

const a = new A() as any;
console.log(a);

// {}
// name
// args:属性装饰器
// A { name: '张三' }

```

#### 参数装饰器 ParameterDecorator

**返回三个参数**

1. 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
2. 成员的名字。
3. 参数在函数参数列表中的索引。

```

class A {
    constructor() {
    }
    setParasm(@ParamDecorator method: string = '参数装饰器') {
    }
}

const a = new A() as any;

function ParamDecorator(target: A, propertyKey: any, parameterIndex: number): void {
    console.log(target);
    console.log(propertyKey); //方法名
    console.log(parameterIndex); //方法的索引位置
}

```



#### 方法装饰器

返回三个参数

1. 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。

2. 成员的名字。

3. 成员的*属性描述符*。

   ```
   class A {
       constructor() {
       }
       @MethDecorator
       setParasm(method: string = '参数装饰器') {
       }
   }
   
   const a = new A() as any;
   
   
   function MethDecorator(target: A, propertyKey: 'setParasm', descriptor?: PropertyDescriptor): void {
       console.log(target);
       console.log(propertyKey); // 方法名
       console.log(descriptor); // 成员的属性描述符
   }
   
   // {}
   // setParasm
   // {
   //   value: [Function: setParasm],
   //   writable: true,
   //   enumerable: false,
   //   configurable: true
   // }
   ```

   #### 案例

   ```
   import axios from 'axios';
   import 'reflect-metadata';
   
   class Http {
       constructor() {
       }
       @Get('https://api.apiopen.top/api/getHaoKanVideo?page=0&size=5')
       getList(@Result data: any) {
           console.log(data);
       }
   }
   
   const http = new Http() as any;
   
   function Get(url: string) {
       const fn: MethodDecorator = function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
           axios.get(url).then(res => {
               // console.log(res);
               const getedKey = Reflect.getMetadata('key', target);
               descriptor.value(getedKey ? res.data[getedKey] : res.data);
           })
       }
       return fn;
   }
   function Result(target: Object, propertyKey: string | symbol, parameterIndex: number) {
       console.log(target, propertyKey, parameterIndex);
       Reflect.defineMetadata('key', 'result', target);
   }
   ```

   

### webpack构建ts+vue3项目

[webpack构建ts+vue3项目](https://blog.csdn.net/qq1195566313/article/details/122708348?spm=1001.2014.3001.5501)



### 实战TS编写发布订阅模式

on订阅/监听

emit 发布/注册

once 只执行一次

off解除绑定

```tsx
interface EventFace {
    on(event: string, callback: Function): void;
    emit(event: string, ...args: any): void;
    off(event: string, callback: Function): void;
    once(event: string, callback: Function): void;
}

class Emitter implements EventFace {
    events: Map<string, Function[]>;
    constructor() {
        this.events = new Map();
    }

    on(event: string, callback: Function) {
        if (this.events.has(event)) {
            const callbackList = this.events.get(event);
            callbackList && callbackList.push(callback);
        } else {
            this.events.set(event, [callback]);
        }
    }
    emit(event: string, ...args: any) {
        const callbackList = this.events.get(event);
        if (callbackList) {
            callbackList.forEach(fn => {
                fn(...args);
            });
        }

    };
    off(event: string, callback: Function) {
        const callbackList = this.events.get(event);
        // console.log("callbackList-off:" + callbackList);
        if (callbackList) {
            let index = callbackList.indexOf(callback);
            callbackList.splice(index, 1);
        }
    };
    once(event: string, callback: Function) {
        const fn = (...args: any) => {
            callback(...args);
            this.off(event, callback);
        };

        this.on(event, fn);
    };
}

const bus = new Emitter();
const fn = (b: boolean, n: number) => {
    console.log(1, b, n);
};


bus.once('once', fn)
bus.emit('once', false, Math.random());
bus.emit('once', false, Math.random());
bus.emit('once', false, Math.random());

console.log(bus);
```

### weakMap，weakSet，set，map

在es5的时候常用的Array object ，在es6又新增了两个类型，Set和Map，类似于数组和对象。

#### **WeakMap 和 WeakSet**

WeakMap 和 WeakSet 是 JavaScript 中的集合类型，其元素是弱引用的对象。这意味着：

* 当对象不再被任何其他变量引用时，它们会被垃圾回收。
* WeakMap 和 WeakSet 自身不会阻止对象被垃圾回收。

**用法：**

* WeakMap 可用于将对象作为键存储值。
* WeakSet 可用于存储对象，而无需关联任何值。

**示例：**

```typescript
// WeakMap
const weakMap = new WeakMap();
const object = {};
weakMap.set(object, "value");

// WeakSet
const weakSet = new WeakSet();
weakSet.add(object);
```

#### **Map 和 Set**

Map 和 Set 是 JavaScript 中的集合类型，其元素是强引用的值。这意味着：

* 只要 Map 或 Set 中包含元素，它们就不会被垃圾回收。
* Map 和 Set 自身可以阻止元素被垃圾回收。

**用法：**

* Map 可用于将键值对存储在集合中。
* Set 可用于存储唯一值。

**示例：**

```typescript
// Map
const map = new Map();
map.set("key", "value");

// Set
const set = new Set();
set.add("value");
```

#### **比较**

| 特征     | WeakMap/WeakSet        | Map/Set            |
| -------- | ---------------------- | ------------------ |
| 元素类型 | 弱引用对象             | 强引用值           |
| 垃圾回收 | 不会阻止               | 会阻止             |
| 键       | 只能是对象             | 任意值             |
| 用途     | 存储对象，避免循环引用 | 存储键值对或唯一值 |

**何时使用**

* **使用 WeakMap/WeakSet：**当需要存储对象且不希望阻止它们被垃圾回收时。例如，在缓存中存储对象。
* **使用 Map/Set：**当需要存储强引用值时。例如，在对象中存储数据或跟踪唯一元素。

#### 案例

首先obj引用了这个对象 + 1，aahph也引用了 + 1，wmap也引用了，但是不会  + 1，应为他是弱引用，不会计入垃圾回收，因此 obj 和 aahph 释放了该引用 weakMap 也会随着消失的，但是有个问题你会发现控制台能输出，值是取不到的，应为V8的GC回收是需要一定时间的，你可以延长到500ms看一看，并且为了避免这个问题不允许读取键值，也不允许遍历，同理weakSet 也一样

```
let obj:any = {name:'小满zs'} //1
let aahph:any = obj //2
let wmap:WeakMap<object,string> = new WeakMap()
 
wmap.set(obj,'爱安徽潘慧') //2 他的键是弱引用不会计数的
 
obj = null // -1
aahph = null;//-1
//v8 GC 不稳定 最少200ms
 
setTimeout(()=>{
    console.log(wmap)
},500)
```

### proxy & Reflect

#### **Proxy**

 Proxy对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）

**target**

要使用 Proxy 包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）。

**handler**

一个通常以函数作为属性的对象，各属性中的函数分别定义了在执行各种操作时代理 p 的行为。

handler.get() 本次使用的get

属性读取操作的捕捉器。

handler.set() 本次使用的set

属性设置操作的捕捉器。

#### **Reflect**

与大多数全局对象不同Reflect并非一个构造函数，所以不能通过new运算符对其进行调用，或者将Reflect对象作为一个函数来调用。Reflect的所有属性和方法都是静态的（就像Math对象）

Reflect.get(target, name, receiver) 
Reflect.get方法查找并返回target对象的name属性，如果没有该属性返回undefined

Reflect.set(target, name,value, receiver) 
Reflect.set方法设置target对象的name属性等于value。

```
type Person = {
    name: string,
    age: number,
    text: string
}

const proxy = (object: any, key: any) => {
    return new Proxy(object, {
        get: (target, prop, receiver) => {
            console.log(`get key======>${key}`);
            return Reflect.get(target, prop, receiver)
        },
        set: (target, prop, value, receiver) => {
            console.log(target)
            console.log(prop)
            console.log(value)

            console.log(`set ${key}======>${value}`);
            return Reflect.set(target, prop, value, receiver)
        }

    })
}

// const logAccess = (object: Person, key: 'name' | 'age' | 'text') => {
//     return proxy(object, key)
// }
// 使用泛型+keyof优化
const logAccess = <T>(object: T, key: keyof T): T => {
    return proxy(object, key)
}

let man: Person = logAccess({ name: 'zhangsan', age: 18, text: 'hello' }, "text")

man.text = "hello world"
```

**案例简单实现一个mobx观察者模式**

```
const list: Set<Function> = new Set<Function>();

const autoRun = (cb: Function) => {
    if (cb) {
        list.add(cb);
    }
}

const obserbver = <T extends object>(params: T) => {
    return new Proxy(params, {
        set: (target: T, key: string, value: string, receiver: any) => {
            const result = Reflect.set(target, key, value, receiver);
            list.forEach(cb => cb());
            return result;
        }
    });
}


const person = obserbver({ name: "小满", attr: "威猛先生" })

autoRun(() => {
    console.log('我变化了')
    console.log(person);
});

person.name = 'zhangsan';

```

### 类型兼容

所谓的类型兼容性，就是用于确定一个类型是否能赋值给其他的类型。typeScript中的类型兼容性是基于结构类型的（也就是形状），如果A要兼容B 那么A至少具有B相同的属性。

**1.协变 也可以叫鸭子类型**
什么是鸭子类型？

一只鸟 走路像鸭子 ，游泳也像，做什么都像，那么这只鸟就可以成为鸭子类型。

举例说明

```
interface A {
    name:string
    age:number
}

interface B {
    name:string
    age:number
    sex:string
}

let a:A = {
    name:"老墨我想吃鱼了",
    age:33,
}

let b:B = {
    name:"老墨我不想吃鱼",
    age:33,
    sex:"女"
}

a = b
```

A B 两个类型完全不同但是竟然可以赋值并无报错B类型充当A类型的子类型，当子类型里面的属性满足A类型就可以进行赋值，也就是说不能少可以多，这就是协变。

**2.逆变**
逆变一般发生于函数的参数上面

举例说明

```
interface A {
    name:string
    age:number
}

interface B {
    name:string
    age:number
    sex:string
}

let a:A = {
    name:"老墨我想吃鱼了",
    age:33,
}

let b:B = {
    name:"老墨我不想吃鱼",
    age:33,
    sex:"女"
}

a = b

let fna = (params:A) => {

}
let fnb = (params:B) => {
    
}

fna = fnb //错误

fnb = fna //正确
```

这里比较绕，注意看fna 赋值 给 fnb 其实最后执行的还是fna 而 fnb的类型能够完全覆盖fna 所以这一定是安全的，相反fna的类型不能完全覆盖fnb少一个sex所以是不安全的。

**3.双向协变**
tsconfig strictFunctionTypes 设置为false 支持双向协变 fna fnb 随便可以来回赋值



### 类型守卫


在 TypeScript 中，类型守卫（Type Guards）是一种用于在运行时检查类型的机制。它们允许你在代码中执行特定的检查，以确定变量的类型，并在需要时执行相应的操作。

**typeof 类型收缩**

```
const isString = (str:any) => {
   return typeof str === 'string';
}
```

在这个例子里面我们声明一个函数可以接受任意类型，只筛选出字符串类型，进行类型收缩。

```
instanceof
const isArr = (value:unknown) => {
    if(value instanceof Array){
        value.length
    }
}
```

使用 instanceof 类型守卫可以检查一个对象是否是特定类的实例

typeof 和 instanceof 区别
typeof 和 instanceof 是 TypeScript 中用于类型检查的两个不同的操作符，它们有不同的作用和使用场景。

typeof 和 instanceof 是 TypeScript 中用于类型检查的两个不同的操作符，它们有不同的作用和使用场景。

```

const str = "Hello";
console.log(typeof str); // 输出: "string"

const num = 42;
console.log(typeof num); // 输出: "number"

const bool = true;
console.log(typeof bool); // 输出: "boolean"
```

注意事项：typeof 只能返回有限的字符串类型，包括 “string”、“number”、“boolean”、“symbol”、“undefined” 和 “object”。对于函数、数组、null 等类型，typeof 也会返回 “object”。因此，typeof 对于复杂类型和自定义类型的判断是有限的。

instanceof
作用：instanceof 操作符用于检查一个对象是否是某个类的实例。它通过检查对象的原型链来确定对象是否由指定的类创建。

```
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

const person = new Person("Alice");
console.log(person instanceof Person); // 输出: true

const obj = {};
console.log(obj instanceof Person); // 输出: false

```

注意事项：instanceof 操作符主要用于检查对象是否是特定类的实例，它无法检查基本类型。此外，它也无法检查对象是通过字面量创建的，因为字面量对象没有显式的构造函数。

注意事项：instanceof 操作符主要用于检查对象是否是特定类的实例，它无法检查基本类型。此外，它也无法检查对象是通过字面量创建的，因为字面量对象没有显式的构造函数。

**自定义守卫**
结合题目实现

实现一个函数支持任意类型
如果是对象，就检查里面的属性，
如果里面的属性是number就取两位，如果是string就去除左右空格
如果是函数就执行

```
const isString = (str:any)=> typeof str === 'string'

const isNumber = (num:any)=> typeof num === 'number'

const isFn = (fn:any)=> typeof fn === 'function'

const isObj = (obj:any)=> ({}).toString.call(obj) === '[object Object]'

const fn = (data:any) => {
    let value;
    if(isObj(data)){
        Object.keys(data).forEach(key=>{
            value = data[key]
            if(isString(value)){
                data[key] = value.trim()
            }
            if(isNumber(value)){
                data[key] = value.toFixed(2)
            }
            if(isFn(value)){
                value()
            }
        })
    }
}
const obj = {
    a: 100.22222,
    b: ' test  ',
    c: function () {
        console.log(this.a);
        return this.a;
    }
}

fn(obj)
```


乍一看没啥问题 一运行就报错

乍一看没啥问题 一运行就报错

他说找不到a

当函数被单独调用时（例如 value()），函数内部的 this 会指向全局对象（在浏览器环境下是 window）

修改如下

```
const isString = (str:any)=> typeof str === 'string'

const isNumber = (num:any)=> typeof num === 'number'

const isFn = (fn:any)=> typeof fn === 'function'

const isObj = (obj:any)=> ({}).toString.call(obj) === '[object Object]'

const fn = (data:any) => {
    let value;
    if(isObj(data)){
        Object.keys(data).forEach(key=>{
            value = data[key]
            if(isString(value)){
                data[key] = value.trim()
            }
            if(isNumber(value)){
                data[key] = value.toFixed(2)
            }
            if(isFn(value)){
                data[key]() //修改这儿
            }
        })
    }
}
const obj = {
    a: 100.22222,
    b: ' test  ',
    c: function () {
        console.log(this);
        return this.a;
    }
}

fn(obj)
```

第一个问题解决了

第二个问题是我们编写的时候没有代码提示很烦

这时候就需要自定义守卫了

类型谓词的语法形式。它表示当 isString 返回 true 时，str 的类型被细化为 string 类型

```
const isString = (str:any):str is string => typeof str === 'string'

const isNumber = (num:any):num is number => typeof num === 'number'

const isFn = (fn:any) => typeof fn === 'function'

const isObj = (obj:any) => ({}).toString.call(obj) === '[object Object]'

const fn = (data:any) => {
    let value;
    if(isObj(data)){
        Object.keys(data).forEach(key=>{
            value = data[key]
            if(isString(value)){
                data[key] = value.trim()
            }
            if(isNumber(value)){
                data[key] = value.toFixed(2)
            }
            if(isFn(value)){
                data[key]()
            }
        })
    }
}
const obj = {
    a: 100.22222,
    b: ' test  ',
    c: function () {
        console.log(this);
        return this.a;
    }
}

fn(obj)
```



### 泛型工具



### infer关键字

#### 提取元素的妙用



#### 用法infer 递归





### 实战插件编写

---
layout: post
title: 从Java学Js(一)JS简介
description: Interceptor介绍
keywords: java, js, 学习, JS, JS简介
categories: js
---
最近由于工作需要，开始接触一些JS的东西。作者想要将自己的学习路程记录下来方便大家一起分享学习。作为一个Android程序原来，在学习Js中难免会和Java进行对比，所以本系列的Blogs是从一个熟练掌握java的开发者的角度出发的。其中一些基础的内容部分比如 if for等等，就不会进行专门的介绍，如果你是一个0基础的人选，推荐可以先看下 [菜鸟教程](https://www.runoob.com/js/js-tutorial.html) .如果你已经熟练掌握了java，同样建议先过一遍 [菜鸟教程](https://www.runoob.com/js/js-tutorial.html) ,你可能只需要2 3个小时就可以浏览完相关内容。

进过深思之后，我决定分成以下内容进行Js的介绍：
* JS的基础介绍
* 动态类型和类型转换
* == 和 ===
* 全局变量和作用域
* 变量提升
* 对象
* 原型链与继承
* eval
* 严格模式
* IDE的选择
* css 和 html
* Js压缩处理
* 其它js库  
* es6
* es7

此目录只作为初步设想，在后续编写过程中可能会做出局部调整。下面我们现在简单看一下JS的基础介绍

内容大纲: 
<pre>
1. JS历史介绍
2. JS版本介绍
3. JS的兼容问题
</pre>
早期的浏览器是由网景公司率先推出的Netspace，最开始是没有js的。网景预见到网络需要变得更动态。公司的创始人马克·安德森认为HTML需要一种胶水语言，让网页设计师和兼职程序员可以很容易地使用它来组装图片和插件之类的组件，且代码可以直接编写在网页标记中。之后便有了JavaScript, JavaScript和java没有任何关系，之所以叫JavaScript是因为当时java很火，网景为了热度最后才临时该名为JavaScript.之后微软在ie上就推出了类似的JScript.然后两家竞争，没有统一的标准导致当时很多的兼容性问题。之后在1996年11月，网景想ECMA(欧洲计算机制造商协会）提交了语言版标准。JavaScript成为了ECMAScript最著名的实现之一。除此之外，ActionScript和JScript也都是ECMAScript规范的实现语言。 

ECMAScript后续发布了若干版本，目前最新版本是2016年发布的ECMAScript 7，其版本时间表如下（来自网络）：  

年份 | 名称 | 描述 
---- | ---- | ---
1997 | ECMAScript 1 | 第一个版本
1998 | ECMAScript 2	| 版本变更
1999 | ECMAScript 3 | 添加正则表达<br/>添加 try/catch
无| ECMAScript 4	| 没有发布
2009 | ECMAScript 5	|添加"strict mode"，严格模式<br>添加 JSON 支持
2011 | ECMAScript 5.1 | 版本变更
2015 | ECMAScript 6(ECMAScript 2015)	| 添加类和模块
2016 | ECMAScript 7(ECMAScript 2016)	| 增加指数运算符 (**)<br/>增加 Array.prototype.includes

各浏览器和手机版本对es的支持可以参照 [http://kangax.github.io/compat-table/es6/](http://kangax.github.io/compat-table/es6/)

从一个java开发的角度出发，个人认为应该重点关注了解JavaScript的以下特性：  
JS单线程模型，JS里是没有多线程操作的  
没有IO操作，IO操作依赖它的载体  
解释执行，不需要预先进行编译，而java需要先编译成字节码  
动态类型，使用var声明变量，而变量类型是在运行时才确认的，而java的变量在编译时已确定  
Eval(), JavaScript可以通过Eval()函数直接执行一段JavaScript脚本  

从这些特性我们可以看到，javaScript是一门非常轻量简单的编程语言，之后我们将按照文章开头的目录一次对JavaScript的使用和特性进行介绍。  

----
参考资料：  
1. [JavaScript教程](https://www.runoob.com/js/js-tutorial.html)
2. 《JavaScript权威指南第五版》
3. [维基百科-JavaScript](https://zh.wikipedia.org/wiki/JavaScript)



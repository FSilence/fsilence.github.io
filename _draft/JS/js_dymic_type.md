---
layout: post
title: 从Java学Js(二)动态类型
description: Js动态类型
keywords: java, js, 学习, JS, 动态类型, Var
categories: js
---

内容大纲：  
<pre>
1. 变量的动态性  
2. typeof 和 instanceof  
3. null undefined 和 NaN
4. 变量之间转换  
</pre>

上一篇我们说到js是解释执行的, 本篇我们将介绍js的动态类型。js并没有java中的 int float String等声明变量的关键字，而是只有一个var用来声明变量。js的变量类型无法提前确认，只有在程序解释执行时才会去动态判断变量的类型，所以给一个变量赋值不同的类型是合法的，如下:  

```javascript
var a = 1
a = "123"
console.log(a)
```  

### typeof 
js 下的类型分为6种：number string object function boolean undefined, 使用typeof关键字只可能返回这6中类型中的一种 以下为示例：  

```javascript
typeof “test”  // 返回string
typeof 12      // 返回number
typeof 12.12   // 返回number
typeof NaN     // 返回number
typeof new Date() // 返回object
typeof [1, 2, 3] // 返回object
typeof {a: "", b: ""} // 返回object
typeof false // 返回boolean
typeof function() {} // 返回function
typeof undefined // 返回undefined
```
### Instanceof
除了typeof外，js还提供了一个instanceOf关键字，可以用来判断变量的具体对象类型, instanceof 只能用于判断对象，不能用于判断基础变量，例如:  

```javascript
"test" instanceof String  // 返回false
12 instanceof Number // 返回false
var a = new String()
a instanceof String // 返回true
```

### 区分整数和浮点数
通过上面介绍 我们发现 通过typeof 和 instanceof 都无法区分出整数和浮点数，那么应该如何区分整数和浮点数呢？  
方式一: 取1的余数，判断是否为0  
```javascript
 2%1 == 0 // 返回true
 2.2%1 == 0 // 返回false，实际2.2%1 的结果为0.20000000000000018, 浮点数的是不精确的
```

方式二: 利用Math的方法  
```javascript
function isInteger(obj) {
    return Math.floor(obj) == obj;
}
 isInteger(1) // 返回true
 isInteger(1.2) // 返回false  
```

方式三: 利用parseInt方法   
```javascript
function isInteger(obj) {
    return parseInt(obj, 10) == obj
}
 isInteger(1) // 返回true
 isInteger(1.2) // 返回false  
```
### null undefined 和 NaN
js中海还有几个特殊的值 null undefined 和 NaN 我们来看下它们在typeof下的表现  
```javascript
typeof null //返回object
typeof undefined //返回undefined
typeof NaN //返回number
```  
从结果可以看出 null是一个特殊的object类型, undefined 是一个特殊关键字，表示尚未定义，NaN是number类型，表示不合法的数字  

### 变量之间的相互转换  
转换成String:  
```javascript
//Number
String(12) // 返回 “12”
(12).toString()
12 + ""

//Date
String(new Date())
new Date() + ""
new Date().toString()

//boolean
String(false)
false + ""
false.toString()

```

转换成Number:  
```javascript
parseInt("12")
Number("12")
+ "12" //一元运算符 尝试转换成Number,不能转换则返回NaN
```

转换成boolean:  
```javascript
Boolean("") // false
```

自动类型转换:  
```javascript
"12" - 1 // 11
"12" + 1 // "121"
5 + null // 5
"5" + null // "5null"
```

---
参考资料:  
1. [JavaScript教程](https://www.runoob.com/js/js-tutorial.html)
2. 《JavaScript权威指南第五版》
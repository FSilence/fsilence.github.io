---
layout: wiki
title: UML类图
description: UML类图
categories: java
keywords: uml, java
---

## 继承

```java
class A {}

class B extends A {}
```
![](/images/wiki/uml/extends.png)

## 实现

```java
interface A {}

class B implements A{}
```

![](/images/wiki/uml/implement.png)

## 关联

```java
class A {
  List<B> bList;
}

class B {
}
```
一对多 多对多 多对一
![](/images/wiki/uml/related.png)

## 聚合关系
  关联关系的一种，表示整体部分，例如 汽车中包含引擎 轮胎

```java
   class 引擎 
   { 
   }; 
   class 轮胎 
   { 
   }; 
   class 汽车 
   { 
   protected: 
   引擎 engine; 
   轮胎 tyre[4]; 
   }; 
```
![](/images/wiki/uml/aggregation.png)

## 合成关系
  关联关系的一种，比聚合关系强，部分和整体具有相同的生命周期

```java
   class 肢 
   { 
   }; 
   class 人 
   { 
   protected: 
   肢 limb[4]; 
   }; 
```
![](/images/wiki/uml/composition.png)

## 依赖关系

   1、依赖关系也是类与类之间的联结  
   2、依赖总是单向的。  
   3、依赖关系在 Java 或 C++ 语言中体现为局部变量、方法的参数或者对静态方法的调用。  

![](/images/wiki/uml/dependency.png)

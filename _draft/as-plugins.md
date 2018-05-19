---
layout: post
title: AndroidStudio插件推荐
description: AndroidStudio插件推荐
categories: java 
keywords: java, android, AndroidStudio, plugin, builder, bean
---
最近开发了2个AndroidStuio插件，像大家推荐一下,也欢迎给我提issues。    

## POJOGenerator
在我们日常开发中我们经常需要和后台打交道，将后台定义的接口文档转化成我们的java对象，这个工作有些繁琐，而后台也不一定给出的是完整的json格式的文档说明，现在插件中心也有很多将json转换为java对象的插件，我这里主要解决的是 将按一定格式排列的数据转换为java对象，这样我们可以将文档中的定义复制到工具中直接生成对应的java类, 具体请移步
[PojoGenerator](https://github.com/FSilence/PojoGenerator)  
![](/images/pojo-use1.png)  
![](/images/pojo-use2.png)  

## BuilderPojoGenerator
顾名思义，这个是Builder模式的生成工具。我们在一个java类中（定义好了field）然后通过工具可以为当前类生成Builder的内部类用于builder模式，具体可见
[BuilderPojoGenerator](https://github.com/FSilence/BuilderPojoGenerator)  
![](/images/builder-generator1.png)  
![](/images/builder-generator2.png)

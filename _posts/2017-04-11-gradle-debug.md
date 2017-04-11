---
layout: post
title: gradle调试断点
description: gradle调试断点
categories: gradle
keywords: debug, gradle
---

调试断点我们的gradle插件  


我们免不了需要调试gradle脚本。但是要特殊说明的是， 现在还没有办法调试gradle的脚本文件， 我们只能通过pringln 来输出message， 然后在Gradle Console中查看。  
我们这里要说的是利用远程调试，去断点调试自己编写的gradle插件。其实gradle的脚本使用groovy编写的而groovy也是运行在jvm上的，所以这里的调试方法就是远程调试jvm的方法，此方法同样适用于调试编译时注解（编译时注解 要注意clean项目）。  

## 配置remote
首先我们先点击Edit Config 点击+号选择remote:   
![](/images/gradle-debug1.png)  

然后复制下面代码，将suspend=n改为suspend=y，表示一直阻塞等待:  
![](/images/gradle-debug2.png)  

你也可以修改name为你喜欢的名字，然后点击ok。  

## 配置运行时候的参数
然后我们有几种方法来运行我们的gradle命令，，

### 方法一
 我们可以在命令行执行: 
 <pre>
 export GRADLE_OPTS="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005"
 </pre>  
 然后直接运行你的gradle命令，你会方法会一直等待5005的端口，知识后依旧可以debug运行刚才的remote，进行断点调试了。  
 你可以通过 unset GRADLE_OPTS 来取消这个配置  
 
### 方法二  
 在gradle中找到你要断点的命令，点击create config:  
 ![](/images/gradle-debug3.png)  
 
 然后在VM options中输入你刚才复制并修改的参数：  
  
   ![](/images/gradle-debug4.png) 
   
   然后点击运行，效果同上 会等待设置的5005端口，然后再切到remote 执行debug即可。  
   
### 方法三  
  
  直接在gradle.properties中设置jvm参数（刚才复制的内容）:  
  
  <pre>
  org.gradle.jvmargs=-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005
  </pre>
 

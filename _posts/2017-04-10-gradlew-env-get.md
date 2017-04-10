---
layout: post
categories: gradle
title: gradle在不同系统下适配
description: gradle在不同环境下获取环境变量
keywords: env, gradle, mac, windows
---
有时我们需要自己定制一些gradle的执行task，可能会用到一些系统中的其它环境，这时需要我们针对不同的OS做一些适配。


##判断当前系统环境
方法一：  
获取os的name判断其中是否包含windows字段  

<pre>
<code>
def isWindows() {
    return System.properties['os.name'].contains('windows');
}
</code>
</pre>

方法二： 
通过OperatingSystem类中的方法来判断当前环境    

<pre>
<code>
def isWindows() {
 return org.gradle.internal.os.OperatingSystem.current().isWindows()
}
</code>
</pre>


##获取系统环境变量  
在windows环境下可以通过  $System.env来获取 比如获取ANDROID_HOME 可以调用$System.env.ANDROID_HOME（注意在path下配置ANDROID_HOME变量）  

在linux和mac上可以通过which指令来获取环境变量，比如我要获取adb的环境变量可以通过以下代码获取：  

<pre>
<code>
"which adb".execute().inputStream.readLines()[0]
</code>
</pre>
mark：  
TODO此方法可以在命令行中调用gradlew命令的时候获取到adb的环境变量，但是如果在Android Studio上直接运行时获取不到的，暂时不知道为什么先mark以下

---
layout: post
title: multidex找不到的问题
description: android Gradle插件升级到2.3.1后multidex找不到的问
categories: android
keywords: android, android studio, 2.3, 2.3.1, multidex
---
最近将AndroidStudio升级到了2.3， 然后发现instan run不好使了，要求Gradle插件版本必须是2.3以上，所以讲gradle插件版本修改为2.3.1，然后发现点击运行会报错：程序包：multidex 找不到。  

在之前的版本中只要配置好multidex = true 插件会自动引用multidex的包，升级到2.3.1后不能自动引用 只能手动添加了, 在dependencies中添加：  
 compile "com.android.support:multidex:1.0.1" 即可。

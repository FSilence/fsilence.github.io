---
layout: post
title: 用gradle构建java项目
description: 用gradle构建java项目
categories: gradle 
keywords: gradle, project, java
---
一般像后台的项目都是用maven构建的，我们来看一下使用gradle来构建java项目，本文不涉及到具体的gradle配置

IDE: idea

1. 首先安装gradle 并配置环境变量  
2. 在项目目录下执行 gradle wrapper 生成wrapper文件（之后再idea中配置使用wrapper构建，使用wrapper的好处在于使用项目配置的gradle版本 不会因为版本冲突引发问题)可以通过--verison 来制定特定的版本  
3. 在idea 设置中搜索gradle 选择使用gradle wrapper 并勾选use-auto-import然后即可  
4. 根据需求自己定义setting.gradle 和 build.gradle中的内容  

在新建完项目执行单元测试的时候你可能会遇到Error:gradle-resources-test:di_test: java.lang.NoClassDefFoundError: org/apache/tools/ant/util/ReaderInputStream 的错误这是你可以执行以下操作：  
File → Invalidate Caches / Restart 然后重启idea即可  

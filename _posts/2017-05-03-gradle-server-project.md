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
  
2. 打开idea 新建一个gradle 项目 如图所示：  
   ![](/images/gradle-server1.png)

3. 在项目目录下执行 gradle wrapper 生成wrapper文件（之后再idea中配置使用wrapper构建，使用wrapper的好处在于使用项目配置的gradle版本 不会因为版本冲突引发问题)可以通过--verison 来制定特定的版本  
同时生成了gradlew文件 通过gradlew 执行gradle命令能够保证多人协作时版本统一。生成的目录结构如下图所示：  
   ![](/images/gradle-server2.png)

4. 在idea 设置中搜索gradle 选择使用gradle wrapper 并勾选use-auto-import然后即可，使用 gradle wrapper可以保证多人协作时gradle版本统一，user-auto-import保证修改gradle配置后自动同步导入  
  
5. 新建一个gradle的web应用  

6. 修改根目录下的setting.gradle文件， 修改为:  
   
```gradle
   include ':web'
```

7. 修改根目录下的build.gradle文件:  
   
```gradle
subprojects { //配置所有子工程都应用以下配置
    buildscript {
        repositories {
            mavenCentral()
        }
    } //build插件的仓库地址
    repositories { //仓库地址
        mavenCentral()
    }

}

ext { //统一配置常量，方便我们管理用到的第三方库的版本
  testVersion = "1.0.1"
}
```   

8. 重启工程

在新建完项目执行单元测试的时候你可能会遇到Error:gradle-resources-test:di_test: java.lang.NoClassDefFoundError: org/apache/tools/ant/util/ReaderInputStream 的错误这是你可以执行以下操作：  
File → Invalidate Caches / Restart 然后重启idea即可  

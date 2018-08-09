---
layout: post
title: Gradle发布项目到Maven(New) 
categories: gradle
description: 发布项目到Maven
keywords: gradle, maven, publish
---  

之前写过一篇Gradle发布到maven的Blog，具体可见 [Android Gradle上传Maven仓库](https://fsilence.github.io/2017/04/26/gradle-maven/)。最近由于项目需要需要一个publishToLocal的功能，发现基于之前的maven插件不太好配置，而且Gradle官方也有了新的发布插件，所以就产生了本文，基于maven-publish插件实现maven上传。  

首先我们明确一下我们的需求：  
1.我们需要发布到maven  
2.我们需要publishToLocal功能  
3.我们需要SnapShot版控制  

我们首先看下项目下的gradle.properties文件：  

```gradle
# 是否支持maven
supportMaven = true
# 仓库地址
mavenUrl = http://maven.test.com
# Snap 的仓库地址
snapUrl = http://maven.test.snapshot.com

# maven name
mavenUserName = name 

# maven password
mavenUserPwd = password 

snapShot = true
```  
我们定义了几个控制项，用来控制当前工程是否支持maven， 各个仓库地址，maven的用户名和密码，以及是否打包成snapShot版   

然后我们完成一份maven-common.gradle文件，用来作为maven模块的复用：  

```gradle
apply plugin: 'maven-publish'

def getMavenUrl() {
    if (isSnapShot()) {
        return project.properties.get("snapUrl")
    } else {
        return project.properties.get("mavenUrl")
    }
}

def getMavenUserName() {
    return project.properties.get("mavenUserName")
}

def getMavenUserPwd() {
    return project.properties.get("mavenUserPwd")
}

def isSnapShot() {
    return project.properties.containsKey("snapShot") &&
            project.properties.get("snapShot").toBoolean()
}

// 获取系统时间
def static releaseTime() {
    return new Date().format("yyyyMMdd.HHmm", TimeZone.getTimeZone("GMT+08"))
}

def getVersionNameStr() {
    if (project.properties.containsKey("versionName")) {
        return "${versionName}"
    } else {
        return "${android.defaultConfig.versionName}"
    }
}

def getTimeStr() {
    return new Date().format('yyyyMMddHHmmss')
}


publishing {
    repositories {
        maven {
            url getMavenUrl()
            credentials {
                username getMavenUserName()
                password getMavenUserPwd()

            }
        }
    }
    publications {
        sdk(MavenPublication) {
            def versionStr = getVersionNameStr()
            if (isSnapShot()) {
                versionStr = "${versionName}.${getTimeStr()}"
            }
            groupId "groupId"
            artifactId "$project.name"
            version versionStr
            artifact(generateJar)
        }
    }
}

```  

在项目下配置是否支持maven发布 写入到root的build.gradle中:  

```gradle
subprojects {
    if (project.getProperties().containsKey("supportMaven") &&
            project.getProperties().get("supportMaven").toBoolean()) {
        apply from: rootProject.file("maven-common.gradle")
    }
}
```

将maven地址加入到repositories中，注意如果需要发布到本地需要加入mavenLocal():
  
```gradle
allprojects {
    repositories {
        google()
        jcenter()
	mavenLocal()
        maven {
            url = "test"
        }
    }
}
```

以上配置结束，可以通过 ./gradlew publishToLocal发布到本地地址，通过./gradlew publish 发布到远程仓库。  

## 其它  

### publishToLocal无法覆盖生效  
发布到local前需要移除gradle的缓存，gradle的缓存目录在 .gradle下的cache中可以写shell脚本方便控制：  

```sh
publishToLocal () {
    echo begin publishToLocal : $1
    # 编译
    ./gradlew $1:build

    # 发布
    ./gradlew $1:publishToMavenLocal

    # 清空缓存
    cachePath=$GRADLE_HOME/caches/modules-2/files-2.1/{Your Group Id}/$1
    echo $cachePath

    if [ ! -f "$cachePath" ];then
        echo "not find cache file"
    else
        rm -rf $cachePath
    fi
}
publishToLocal {Your Project Name}
```
将示例的GroupId 和 project Name换成自己的即可  


### 0.0.1.+ 无法获取最新版本  
系统缓存默认时间为24小时，可以将缓存时间改为0:  

```gradle
 configurations.all {
        resolutionStrategy.cacheDynamicVersionsFor 0, 'seconds'
    }
```

### 0.0.1-SNAPSHOT 无法获取最新版本  
将项目标记为 changing=true 然后配置:  

```
    configurations.all {
        resolutionStrategy.cacheChangingModulesFor 0, 'seconds'
    }
```
具体描述可以参照 之前的老文章见篇头  




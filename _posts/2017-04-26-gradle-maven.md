---
layout: post
title: Android Gradle上传Maven仓库
description: Android Gradle配置上传Maven仓库
categories: gradle
keywords: gradle, maven
---

在Android的gradle配置上追加上传Maven的相关操作  

## maven的gradle插件
Android里内置了maven插件 只需要apply plugin: “maven” 即可使用maven插件

## 配置maven
### 配置maven仓库地址
在root 下的build.gradle 的subprojects下的repositories中追加一下maven仓库地址，表示给子项目添加仓库地址
<pre>
  maven {
            url "http://maven.release.test" 
        }
  maven {
            url "http://maven.test"
        }
</pre>
### 统一配置maven的账号信息  
 我们现在setting.gradle中统一配置账号信息，留待下面使用

<pre>
gradle.ext {
    // maven config
    mavenUrl = "http://maven.test.release"
    mavenDevUrl = "http://maven.test.snapshot"
    mavenUserName = "deployer"
    mavenPwd = "test"
}
</pre>
其中：  
mavenUrl表示正式仓库地址  
mavenDevUrl表示snapshot的仓库（这里我们分了2个仓库 你也可以使用一个）  
mavenUserName maven账号的用户名  
mavenPwd macen账号密码

### 配置maven上传
这里用到的gradle.mavenUrl等 都是刚才在setting.gradle中配置好的
<pre>
uploadArchives{
    repositories {
        mavenDeployer {
            repository(url: gradle.mavenUrl) {
                authentication(userName: gradle.mavenUserName, password: gradle.mavenPwd)
            }
            project.afterEvaluate {
                def versionName = "${android.defaultConfig.versionName}"
                def version = versionName;
                //修改pom文件（maven的配置文件）
                pom('aar').version = version;
                pom('aar').artifactId = "$project.name"
                pom('aar').groupId = "com.fsilence"  
            }
        }
    }
}
</pre>
然后我们调用gradle 的uploadArchives 的task就可以了，之后可以在你的maven仓库中查看代码。在这里我们的artifactid直接使用了当前项目的名称，version使用了当前项目android中配置的版本号，你也可以手动设置不过不建议。   
我们可以直接在build中调用 compile "com.fsilence:{$project.name}:{$version}"来应用了 例如 compile "com.fsilence:test:1.0.0"

## 配置SNAPSHOT 和 sources版本
我们在其它的开源软件上经常能看到snapshot版本 和 带源码的版本，那么我们要怎么配置呢？ 我们先来看snapshot版本的配置, snapshot和release的区分主要是在版本号的结尾 如果版本号是以SNAPSHOT结尾的就会本认为是快照版本(其实合理可以用其它的字符串只要和仓库的开头能匹配上即可)，我们可以给snapshot版本配置特殊的仓库:  
<pre>
   snapshotRepository(url: gradle.mavenDevUrl) {
          authentication(userName: gradle.mavenUserName, password: gradle.mavenPwd)
       }
</pre>
为了标识打包的是snapshot还是其它版本，我会在项目下的gradle.properties文件中定义个isSnapshot的变量如下:  
<pre>
isSnapshot = true
</pre>
然后我们就可以在项目中通过project.isSnapshot来划分版本了，然后我们将上面的配置代码加入snapshot版本的控制，  
<pre>
uploadArchives{
    repositories {
        mavenDeployer {
            repository(url: gradle.mavenUrl) {
                authentication(userName: gradle.mavenUserName, password: gradle.mavenPwd)
            }
            project.afterEvaluate {
                def versionName = "${android.defaultConfig.versionName}"
                def version = versionName;
                if (project.isSnapshot) {
                    version = "$versionName-SNAPSHOT"
                }
                //修改pom文件（maven的配置文件）
                pom('aar').version = version;
                pom('aar').artifactId = "$project.name"
                pom('aar').groupId = "com.fsilence"  
            }
        }
    }
}
</pre>

然后我们再追加一个sources的版本控制，我们在每个版本都添加一个-sources的版本（包含源码的），首先我们得有一个打包源码的task：  
<pre>
task generateSourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier 'sources'
}
</pre>
然后我们再次修改上面的配置，也就是我们最终的配置：  
<pre>

uploadArchives{
    repositories {
        mavenDeployer {
            repository(url: gradle.mavenUrl) {
                authentication(userName: gradle.mavenUserName, password: gradle.mavenPwd)
            }
            snapshotRepository(url: gradle.mavenDevUrl) {
                authentication(userName: gradle.mavenUserName, password: gradle.mavenPwd)
            }
            addFilter("aar-src"){artifact, file ->
                artifact.name == 'aar-src' || artifact.name == 'aar'
            }
            addFilter("aar"){artifact, file ->
                artifact.name == 'aar'
            }
            project.afterEvaluate {
                print project.isSnapshot
                def versionName = "${android.defaultConfig.versionName}"
                def version = versionName;
                def aarSrcVersion = "$versionName-sources";
                if (project.isSnapshot) {
                    version = "$versionName-SNAPSHOT"
                    aarSrcVersion += "-SNAPSHOT"
                }
                pom('aar').version = version;
                pom('aar').artifactId = "$project.name"
                pom('aar').groupId = "com.fsilence"
                pom('aar-src').artifactId = "$project.name"
                pom('aar-src').groupId = "com.fsilence"
                pom('aar-src').version = aarSrcVersion
            }
        }
    }
}
</pre>
我们现在达到了这个效果：  
1. 我们可以手动通过配置 isSnapshot来决定打包的是SnapSHot还是正式版本  
2. 每个版本都有一个-sources的包是包含项目源码的  
举个例子我们的版本号可以是: 1.0.0 1.0.0-sources 1.0.0-SNAPSHOT 或者 1.0.0-sources-SNAPSHOT是不是很清晰。  

## 解决gradle上使用maven版本不同步的问题
你以为这样就ok了吗,在实际中我还遇到一个问题，当你在开发发布snap版本的时候，其它引用的地方总是不能及时更新，这是为什么呢？最后我发现是因为gradle对jar包的缓存问题，我们可以才用户目录的.android下的caches/modules-2/files中查找到缓存的文件，这给实际开发造成了很大影响，解决方案有2个：  
1.每次发的版本号都不相同，我们可以在版本号中间再插入一个git的版本号（或者你使用的是svn），git获取版本号的代码如下:  
<pre>
def gitVersion = 'git rev-parse --short HEAD'.execute().text.trim()
</pre>
这样可以解决我们的问题但是有一个麻烦支出在于 ，引用包的地方必须知道你的git版本号，这可是很麻烦的，如果你又不希望对方知道你的git地址。所以这种方式个人不推荐  
2.让gradle每次都能去下载最新版本的maven仓库,我们有一个dsl的配置如下:  
<pre>
configurations.all {
    resolutionStrategy.cacheChangingModulesFor 0, 'seconds'
}
</pre>
然后在引用的地方我们改为：  
<pre>
compile ("com.fsilence:test:1.0.0-SNAPSHOT"){changing = true}
</pre>
这样我们每次调用的时候都会更新到最新版本，如果改成正式版本后曲调后面的changing配置即可。  

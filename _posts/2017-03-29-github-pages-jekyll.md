---
layout: post
title: 用GithubPages搭建个人Blog
description: 搭建个人Blog
categories: Blog
keywords: githubpages, Blog, Jekyll 
---

用GithubPages + jekyll搭建你的个人Blog, 专注于内容。  


<pre>
内容大纲：
1.配置githubpages  
2.选择你的jekyll模板    
3.开始写Blog  
</pre>
以前总是羡慕别人有自己的Blog，一直想有有个属于自己的Blog网站。之前断断续续写过一些，用过csdn 用过 eoe的blog（现在已经没有这个功能了）。 也尝试过wordpress等，买过云服务器。最后总是不了了之。前段时间偶然看到了githubpages, 眼前一亮
，这么nb的功能以前竟然不知道。然后花了几天时间fork了别人的Blog 修改后搭建了自己的Blog。

## 配置githubs 
官网上有很详细的介绍，基本就是你新建一个以你的github名字开始+github.io的仓库，具体的配置过程可以移步官网 [github.io](http://github.io) 我就不做重复的搬运了   
之后可以在工程的Settings中做一些配置，比如指定你的个人域名， 


## 配置jekyll  
关于jekyll 可以移步 [Using jekyll with GithubPages](https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/)  
基本是说jekyll是一个静态html的生成框架。  
关于jekyll的配置模板网上有很多 你可以选择你喜欢的主题加以修改[jekyll themes](http://jekyllthemes.org/)  
如果你像我一样对前端不了解的话，别着急你可以fork别人的blog 然后修改其中的内容，告诉你个小技巧 你可以在github上搜索github,io 然后选择你喜欢的blog fork后修改其中的关键内容就行，我就是fork的 [https://github.com/mzlogin/mzlogin.github.io](https://github.com/mzlogin/mzlogin.github.io)  
一般你只需要修改_config.yml 和CNAME中的东西即可，是不是很容易啊。如果项目中设计到其它第三方的插件，你需要注册你个人的插件并替换相关内容，比如用到的评论模块,你需要换成你个人的。  

## 开始Blog
将项目push到你新建的github仓库中，之后你就可以开始写Blog了。

本文只是一个阐述，详细过程可以参照其他人的Blog [http://www.jianshu.com/p/3f355c7872d5](http://www.jianshu.com/p/3f355c7872d5)

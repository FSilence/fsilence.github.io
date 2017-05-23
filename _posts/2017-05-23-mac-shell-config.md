---
layout: post
title: Mac Shell的配置
description: Mac配置shell
categories: 其它
keywords: mac, shell, config
---

要想方便的使用mac，shell的使用个人认为是必不可少的，在我们拿到mac的时候，我们应该先配置好我们的shell，这样可以极大的提高我们的效率。本文不会描述具体的配置过程，只是对相关配置资料的一个索引。

## zsh
为什么我要使用zsh 而不是bash，可以参照[为什么说 zsh 是 shell 中的极品？
](https://www.zhihu.com/question/21418449)  

mac安装zsh:   
zsh的配置能力非常强，如果你要自己完全配置zsh你会发现非常非常复杂，还好有人给我我们提供了不错的配置方案 [on-my-zsh](https://github.com/robbyrussell/oh-my-zsh)  

配置zsh:  
zsh的配置在文件 ~/.zshrc中，我们可以在其中使用别名，配置插件等提高shell的使用效率。  
zsh的具体配置可以参照  
[在osx中配置和使用zsh](http://www.jianshu.com/p/24a0ded2e3ba)  
[ zsh安装和配置](http://blog.csdn.net/ii1245712564/article/details/45843657)
  

Terminal自动开启zsh:  
安装好on-my-zsh后，我们可以在 ~/.bash_profile中 添加一行 zsh，这样每次启动terminal的时候就都是zsh

## 安装autojump
通过 brew 安装autojump:  
<pre>
brew install autojump
</pre>

在 ~/.zshrc中加入以下内容  
<pre>
[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh
</pre>

然后重新启动zsh，就可以通过j + 目录名跳转到任何你cd过的目录中，支持模糊匹配。

## 配置vi
vi的插件管理 参考 [VBundle的介绍及安装](http://blog.csdn.net/zhangpower1993/article/details/52184581)  
vi的插件统建 参考 [vi配置及插件管理](http://blog.csdn.net/namecyf/article/details/7787479)
## 其它
[osx shell快捷键](http://fsilence.com/wiki/mac-bash/)

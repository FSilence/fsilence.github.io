---
layout: post
title: Python环境搭建
description: Python环境搭建
categories: python
keywords: python, Env, 环境搭建
---
上一篇和大家分享了Python是什么，为什么要开始学习Python。从本篇开始，我们将正式开始一起学Python。工欲善其事必先利其器，让我们先来准备一下我们的“器”.  

我现在换成了Mac，所以我主要说明的还是mac上的配置搭建，而关于windows系统我会给大家一些链接，或者大家可以各自google 或 百度。

## 管理Python的版本
Python比较蛋疼的一点在于python有2个系列的版本，2系列 和 3系列，这两个系列之间的api 和 语法都有一些差别，而市面上的python脚本针对的版本也是分庭抗礼，所以这就导致了我们必须要有2个版本才可以。（之前和同事用的python版本不一致非常蛋疼的体验，为了要能运行同事的python 和 自己的python得频繁的切换2 和3 的python版本，请原谅我当时没有管理好python的多版本）. mac/linux python下载可以使用之后要提到的pyenv管理，windows可以去 [Python 官网](https://www.python.org/)下载。  

Python的多版本管理基本上有2种方式:  
1.修改python3的命令，将使python 代表python2 python3代表python3  
2.使用python的环境管理工具pyenv(我不太清楚windows上的python环境管理工具，如果你有好用的windows上的python环境管理工具可以告知一下我)  

因为考虑到如果修改python3的命令为python3后，使用其它脚本时内部的python可能需要换成python3，而且此方法不适用太多的版本，比如我有一个2.7 一个 2.6的版本。我们重点来看一下pyenv的安装和使用;  
### 安装pyenv
使用brew命令来安装pyenv: 
<pre>
brew install pyenv
</pre>
安装成功后，使用pyenv来查看是否安装成功
[!](/images/study/python/env/1.png)

### pyenv的命令
可以通过pyenv help来查看pyenv支持的命令，我们在这里说几个常用的命令  

命令 | 描述 | 例子
-----|------|------
pyenv install $version | 安装制定版本的python | pyenv install 3.6.1
pyenv uninstall $version | 卸载指定版本 | pyenv uninstall 3.6.1
pyenv local $version | 切换本地python版本，使用后要调用rehash命令才能生效 | pyenv local 3.6.1
pyenv shell $version | 修改python版本，只在当前shell生效，同上需要rehash | pyenv shell 3.6.1
pyenv version | 查看当前python版本 | pyenv version
pyenv versions | 查看全部python版本, 其中带 * 的是当前的版本 | pyenv versions

## IDE
因为java开发的时候用惯了idea所以ide我选择了同一个系列的 [Pychram](http://www.jetbrains.com/pycharm/)

## 包管理(pip)


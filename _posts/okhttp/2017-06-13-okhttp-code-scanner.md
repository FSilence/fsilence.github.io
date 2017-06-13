---
layout: post
title: okHttp各个module介绍
description: okHttp各个module介绍 
categories: java
keywords: okhttp, module
---
上一篇我们简单介绍了okHttp，了解了OkHttp的基本用法，并下载了okhttp的源码。本篇我们将一起来看看okhttp源码的目录结构，来大概了解下okHttp源码下的各个module的作用。  

okHttp的构建工具使用的是maven，关于[maven](https://www.google.com.hk/#safe=strict&q=maven)可自行google,本文不对此做过多阐述。我们来重点看一下okHttp的项目目录。在这里以及之后的源码都是okHttp的最新版本 3.8.0。我们可以选择git工具checkout出 3.8.0的tag，本人使用的是idea自带的git管理工具，在右下角git分支出点击 选测checkout tag, 输入3.8.0的分支名字：parent-3.8.0 确认即可, 然后我们来看一下项目的目录结构截图:  
![](/images/okhttp/okhttp-modules.png)  


我们来看看各个module的功能作用，主要的借阅资料是module下的README.MD文件，读者也可自行查阅。  

## benchmarks
README.MD中的说明:  
<pre>
This module allows you to test the performance of HTTP clients.
Running
  1. If you made modifications to `Benchmark` run `mvn compile`.
  2. Run `mvn exec:exec` to launch a new JVM, which will execute the benchmark.
</pre>

用google caliper来测试HTTP客户端性能的，具体的使用我们先不加关注。  

## mockwebserver
README.MD中的说明，我们这里只截取了介绍部分，具体的使用说明不做说明。  
<pre>
A scriptable web server for testing HTTP clients    
**Motivation**  
This library makes it easy to test that your app Does The Right Thing when it
makes HTTP and HTTPS calls. It lets you specify which responses to return and
then verify that requests were made as expected.  
Because it exercises your full HTTP stack, you can be confident that you're
testing everything. You can even copy & paste HTTP responses from your real web
server to create representative test cases. Or test that your code survives in
awkward-to-reproduce situations like 500 errors or slow-loading responses.  
**Example**   
Use MockWebServer the same way that you use mocking frameworks like
[Mockito](https://github.com/mockito/mockito):
1. Script the mocks.
2. Run application code.
3. Verify that the expected requests were made.
</pre>
此处省略了具体的demo部分，大致是说MockWebServer是用来在单元测试中来模拟网络请求用的，我们可以模拟一个网络返回来验证我们的网络调用。  

## okcurl
来看README文件:  
<pre>
_A curl for the next-generation web._  
OkCurl is an OkHttp-backed curl clone which allows you to test OkHttp's HTTP engine (including
HTTP/2) against web servers.
</pre>
用来离线测试okhttp引擎的。

## okhttp
我们将要研究的项目主体。 

## okhttp-android-support okhttp-urlconnection okhttp-apache
对其它http引擎的适配。 

## okhttp-hpacktests
<pre>
These tests use the [hpack-test-case][1] project to validate OkHttp's HPACK
implementation.  The HPACK test cases are in a separate git submodule, so to
initialize them, you must run:

    git submodule init
    git submodule update

TODO
----

 * Add maven goal to avoid manual call to git submodule init.
 * Make hpack-test-case update itself from git, and run new tests.
 * Add maven goal to generate stories and a pull request to hpack-test-case
   to have others validate our output.

[1]: https://github.com/http2jp/hpack-test-case 
</pre>
这个是用来测试hpack的。HPACK是http 2.0的头部压缩算法。

## okhttp-loggin-interceptor
<pre>
An [OkHttp interceptor][1] which logs HTTP request and response data.
</pre>
日志输出的拦截器，关于interceptor之后再分析OkHttp的代码中我们会看到它的具体含义和使用。  

## okhttp-testing-support
对okhttp测试的一些基础能力支持。

## okhttp-tests
okhttp自己的测试 

## samples
demo

## website 
okhttp的介绍说明的静态网页。  

我们已大致了解了okhttp下的各个module，在后面的文章中我们将开始逐步进入okhttp的具体代码，来一步一步学习okhttp。  

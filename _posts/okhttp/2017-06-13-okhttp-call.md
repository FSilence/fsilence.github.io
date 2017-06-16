---
layout: post
title: okHttp之请求Call
description: 介绍okHttp中的Call的体系结构
categories: java
keywords: java, okhttp, call
---

前一篇我们简单介绍了OkHttp中的几个常见的概念，我们知道OkHttpClient是请求的门户（外观模式），通过OkHttpClient创建了一个Call对象，然后通过Call对象来实现网络的请求，本篇我们来看一下Call做同步和异步时候都做了些什么。  

首先，我们先来看一下Call Dispatcher OkHttpClient 等的类关系图:  
![](/images/okhttp/call-class.png)  

我们可以看到OkHttpClient实现了Call.Factory方法负责生产Call对象，查看OkHttpClient中的newCall方法可以发现，创建的是一个RealCall对象，RealCall对象持有OkHttpClient来方便使用OkHttpCLient中的配置项等。 在RealCall中有一个内部类AsyncCall是对RealCall的一次封装，在做异步请求的时候会用到这个对象。  

## 同步请求  
现在我们来看一下同步请求的调用过程， 我们先来看一下调用的序列图:  
![](/images/okhttp/sync-call-seq.png)  
同步请求的调用过程可以总结为:  
1.调用Dispatcher的execute方法  
2.调用getResponseWithInterceptorChain方法获取结果Respone(具体的过程，我们之后再来做研究)  
3.调用Dispatcher的finish方法，通知请求结束  

## 异步请求  
异步请求和同步调用表现基本一致，不过在异步中，RealCall中是通过封装了一个AsyncCall（一个Runnable）然后再交给Dispatcher的，然后再AsyncCall的execute方法中，依然是通过getResponseWithInterceptorChain来获取结果，然后再调用Dispatcher的finish方法来通知Dispatcher任务结束。我们来看下异步请求的序列图:  
![](/images/okhttp/async-call-seq.png)  

## getResponseWithInterceptorChain  
之前我们介绍了，不管是同步还是异步获取Response都是通过RealCall中的getResponseWithInterceptorChain方法。我们来看下方法的实现;  

```java
  Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(
        interceptors, null, null, null, 0, originalRequest);
    return chain.proceed(originalRequest);
  }

```
可以看到 首先由一个interceptors的列表，然后调用Interceptor.Chain的proceed方法，可以理解会依次触发Interceptor并最终返回Response, 至于各个Interceptor的作用，我们在下一篇中再来做详细的讨论。  



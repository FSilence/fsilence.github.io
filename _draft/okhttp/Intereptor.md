---
layout: post
title: okhttp之Interceptor
description: Interceptor介绍
keywords: okhttp, sources, java, interceptor, chain
categories: java
---
之前我们介绍了OkHttp的请求分发过程，其中的Response都是通过RealCall中的getResonseWithInterceptorChain方法获取的。之前在讲解OkHttpClient中我们也遗留了关于networkInterceptor 和 interceptor的疑问。本节我们就来看一下okhttp之中的Interceptor。 

首先，我们先来看下2个概念， Interceptor 和 Interceptor.Chain ;  

```java
/**
 * Observes, modifies, and potentially short-circuits requests going out and the corresponding
 * responses coming back in. Typically interceptors add, remove, or transform headers on the request
 * or response.
 */
public interface Interceptor {
  Response intercept(Chain chain) throws IOException;

  interface Chain {
    Request request();

    Response proceed(Request request) throws IOException;

    /**
     * Returns the connection the request will be executed on. This is only available in the chains
     * of network interceptors; for application interceptors this is always null.
     */
    @Nullable Connection connection();
  }
}
```

Intercepor中只有一个方法 intercept 参数传入Chain 结果返回Response，   
Chain中主要的方法是proceed方法，传入Request 返回 Response. 

Interceptor可以理解为拦截器， Chain可以理解为各个拦截器的中建产物，用来交付给下一个Interceptor去处理的。Interceptor.Chain只有一个实现类，那就是RealInterceptorChain, 我们先来看下其中的一些参数:  

```java
 //全部的拦截器
 private final List<Interceptor> interceptors;
  
 private final StreamAllocation streamAllocation;
  
  //建立http连接之后的流处理器
  private final HttpCodec httpCodec;
  
  //连接，用来建立http连接，获取HttpCodec对象
  private final RealConnection connection;
 
  //当前Chain对应的interceptors中的下标
  private final int index;
  
  //请求Request
  private final Request request;
  
  //记录当前proceed方法调用次数
  private int calls;
```  
我们先来看下 

```java

```

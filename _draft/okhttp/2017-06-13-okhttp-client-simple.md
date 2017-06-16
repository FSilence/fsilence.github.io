---
layout: post
title: OkHttp源码初探
description: OkHttp源码初探
categories: java
keywords: java, okhttp, okhttpclient, call, realcall, asynccall
---
在之前的文章我中我们介绍了OkHttp的基本使用方法并简单说明了源码下各个module的功能作用，从这篇开始我们将要开始分析okHttp的源码。  

首先，我们先来回忆一下OkHttp的使用过程: 
1.创建一个OkHttpClient对象  
2.创建一个Request对象  
3.调用OkHttpClient的newCall方法创建request的Call实例  
4.调用Call的execute 或 enqeue方法分别启动同步异步请求获取Response  

上述我们涉及到几个概念OkHttpClient, Request, Call, Response 我们将分别来分析一下这些内容。  

## Request
一个Request是一次请求的抽象，我们先简单看下里面的参数:  

```java
  final HttpUrl url;
  final String method;
  final Headers headers;
  final @Nullable RequestBody body;
  final Object tag;
```
可以看到一个Request 基本包含一个请求的url 请求的方法， Headers 和 请求的body，还有一个tag。  
关于Request的具体内容，我们会在后面的系列文章中来进行分析。   

## OkHttpClient  
我们可以看到OkHttpClient基本上是和okHttp打交道的门户，我们先来看一下OkHttpClient的定义:  

```java
public class OkHttpClient implements Cloneable, Call.Factory, WebSocket.Factory
```
可以看到OkHttpClient实现了接口Call.Factory 和 WebSocket,Factory顾名思义，OkHttpClient将负责创建Call 和 WebSocket的实例，在这里我们先对WebSocket不做考虑，我们讲桌布分析Http的请求。  

OkHttpClient使用了创建者模式，使用一个Builder对象来控制构建一个OkHttpClient所需的各项配置，我们来看一下Builder中的参数:  

```java
  final Dispatcher dispatcher;
  final @Nullable Proxy proxy;
  final List<Protocol> protocols;
  final List<ConnectionSpec> connectionSpecs;
  final List<Interceptor> interceptors;
  final List<Interceptor> networkInterceptors;
  final EventListener.Factory eventListenerFactory;
  final ProxySelector proxySelector;
  final CookieJar cookieJar;
  final @Nullable Cache cache;
  final @Nullable InternalCache internalCache;
  final SocketFactory socketFactory;
  final @Nullable SSLSocketFactory sslSocketFactory;
  final @Nullable CertificateChainCleaner certificateChainCleaner;
  final HostnameVerifier hostnameVerifier;
  final CertificatePinner certificatePinner;
  final Authenticator proxyAuthenticator;
  final Authenticator authenticator;
  final ConnectionPool connectionPool;
  final Dns dns;
  final boolean followSslRedirects;
  final boolean followRedirects;
  final boolean retryOnConnectionFailure;
  final int connectTimeout;
  final int readTimeout;
  final int writeTimeout;
  final int pingInterval;
```
除了一些Http的配置外，我们可以看到3个特殊的概念配置，分别是EventListener.Factory Dispatcher 和 Interceptor  
我们先来看一下EventListener 至于Dispatcher 和 interceptor我们会在系列文章的后续部分看到详细的分析，其中Dispatcher是来管理分发请求任务的，Interceptor是拦截器。  

```java
abstract class EventListener {
  public static final EventListener NONE = new EventListener() {
  };

  static EventListener.Factory factory(final EventListener listener) {
    return new EventListener.Factory() {
      public EventListener create(Call call) {
        return listener;
      }
    };
  }

  public void fetchStart(Call call) {
  }

  public void dnsStart(Call call, String domainName) {
  }

  public void dnsEnd(Call call, String domainName, List<InetAddress> inetAddressList,
      Throwable throwable) {
  }

  public void connectStart(Call call, InetAddress address, int port) {
  }

  public void secureConnectStart(Call call) {
  }

  public void secureConnectEnd(Call call, Handshake handshake,
      Throwable throwable) {
  }

  public void connectEnd(Call call,  InetAddress address, int port, String protocol,
      Throwable throwable) {
  }

  public void requestHeadersStart(Call call) {
  }

  public void requestHeadersEnd(Call call, Throwable throwable) {
  }

  ...

  public interface Factory {
    EventListener create(Call call);
```
我们可以看到EventListener是对请求过程中各个步骤的事件监听.  

## Call
一次具体的请求, OkHttpClient通过newCall创建了call对象，通过call对象的execute 或 enqeue来发起请求  

## Response
网络返回的结果，我们将在后续文章中进行详细的解析。  

现在我们已经了解了在okHttp启动中的几个概念 其中有入口 OkHttpClient, 请求的实例Request 请求的抽象Call 拦截器Interceptor 分发管理Call的Dispatcher http的返回响应 Response, 我们还阅读了OkHttpClient的源码, 在下一篇中我们将要详细分析Call的体系结构。  


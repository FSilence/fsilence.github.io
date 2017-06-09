---
layout: post
title: okhttp-简介
description: okhttp简介
keywords: okhttp, 源码, java
categories: java

---
An HTTP & HTTP/2 client for Android and Java applications

[okhttp github](https://github.com/square/okhttp)

## okhttp简介
okhttp是[square公司](https://zh.wikipedia.org/wiki/Square%E5%85%AC%E5%8F%B8)以Apache License, Version 2.0开源的一个网络框架。  
在现在的网络应用中，我们一般都是用HTTP来做数据交换的，一个高效的HTTP可以让你应用加载更快并节省带宽。  
OkHttp有以下的优势:  
* 支持Http2(支持相同host的请求共享socket通道)  
* 如果Http2不能使用，会有连接池来降低请求等待时间  
* 通过gzip压缩减少sizes  
* responese缓存, 减少重复请求 
* OKHttp默认支持失败重连，如果你有多个IP地址，在服务失败后OKHttp将尝试切换请求的IP  
* OKhttp默认支持TLS的新版特性(SNI, ALPN),如果握手失败将自动降级到TLS 1.0.OkHttp引用了OkIo的项目  

OkHttp引用了OkIo的项目,[OkIo](https://github.com/square/okio)是对javaIO的扩展和简化。    
OkHttp支持同步和异步的请求，最低支持Android 2.3  java 1.7的版本。  
我们可以把okhttp看成是Appach Http的替代品。
## okhttp基本使用
### okhttp引用
直接下载jar包（需要引入okIo的jar包）  

MAVEN:  

```xml
<dependency>
  <groupId>com.squareup.okhttp3</groupId>
  <artifactId>okhttp</artifactId>
  <version>3.8.0</version>
</dependency>
```

GRADLE:  
```gradle
compile 'com.squareup.okhttp3:okhttp:3.8.0'
```

### 基本使用
GET A URL:  

```java
kHttpClient client = new OkHttpClient(); 
String run(String url) throws IOException {
  Request request = new Request.Builder()
      .url(url)
      .build();
  try (Response response = client.newCall(request).execute()) {
    return response.body().string();
  }
}
```

POST TO A SERVER:  

```java
public static final MediaType JSON
    = MediaType.parse("application/json; charset=utf-8");

OkHttpClient client = new OkHttpClient();

String post(String url, String json) throws IOException {
  RequestBody body = RequestBody.create(JSON, json);
  Request request = new Request.Builder()
      .url(url)
      .post(body)
      .build();
  try (Response response = client.newCall(request).execute()) {
    return response.body().string();
  }
}
```

## 源码下载
github : [okhttp github](https://github.com/square/okhttp)   
IDE: idea   
我们用git clone项目到本地即可, 项目是maven项目我使用的是idea编译器，我们用idea导入项目，然后同步maven即可。  

下一篇我们来介绍okhttp项目下的各个module。  


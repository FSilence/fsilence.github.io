---
layout: post
title: EventBus Q&A
description: EventBus Q&A
categories: android
keywords: android, eventbus
---
阅读EventBus源码过程中总结的一些问题和答案

### EventBus 中如果一个subscriber中的父类也注册了相同的监听事件，以哪个为准
___
以子类中的事件为准，在SubscriberMethodFinder方法中，在添加一个subscribeMethod的时候  会调用FindState的checkAdd方法，其中有这么一段逻辑：
<pre>
<code>
   if (methodClassOld == null || methodClassOld.isAssignableFrom(methodClass)) {
                // Only add if not already found in a sub class
                return true;
            } else {
                // Revert the put, old class is further down the class hierarchy
                subscriberClassByMethodKey.put(methodKey, methodClassOld);
                return false;
            }
</code>
</pre>

如果原先注册过的methodClass 是当前想要注册的methodClass的父类的话，返回true，（isAssignableFrom 判断类是否是另一个类的父类）
在结束后
<pre>
<code>
 if (findState.checkAdd(method, eventType)) {
ThreadMode threadMode = subscribeAnnotation.threadMode();
findState.subscriberMethods.add(new SubscriberMethod(method, eventType, threadMode,
                                    subscribeAnnotation.priority(), subscribeAnnotation.sticky()));
                        }
</code>
</pre>
如果checkAdd返回了true的话 表示可以添加当前method到列表中  

###SubscribeMethodFinder中的FindStat有什么用
FindStat用来记录一次查找的中间变量和结果，其中的superClass用来帮助查找器进行递归查找。  

### EventBus中的event 对象是如何管理的
当用户通过post来发送一个消息的时候，EventBus会首先获取一个PostThreadStat对象（通过ThreadLocal存放的 每个线程都会有一份，来记录当前线程的一些发送状态），然后将消息放入到PostThreadStat中的event队列，然后会依次取队列中的消息，进行发送操作。  
单次的发送操作： 
如果eventInheritance为true(默认为true)，就先查找出所有的父类 和 接口，都记录为event，发送，如果为false 则只发送当前消息。如果没有找到对应的订阅者，则抛出一个NoSubscriberEvent的消息。如果有订阅者，则根据订阅者的类型来选择发送的方式。  
EventBus中有3个变量来保存消息，1.subscriptionsByEventType 一个Map,key是eventType value是subscribtions，主要用来post消息的时候可以找到对应的subscribtion来换起方法。2.typesBySubscriber 一个Map,key是订阅者 value是订阅的事件列表，主要用来在unregister订阅者的时候方便去清空对应的数据。3.stickyEvents 主要是用来存放sticky消息的。

### EventBus的register的过程
register一个订阅者的时候，会进行以下几步：  
1.通过subscriberMethodFinder查找到所有注释的订阅方法  
2.循环依次注册方法  
3.每一次注册的时候，根据优先级加入到列表中（subscriptionsByEventType的value），优先级高的在前面  
4.更新typesBySubscriber  
5.如果注册的方法是sticky的，则检查当前的stickyEvents列表，触发sticky消息的相应  

### SubscriberMethodFinder的查找过程
Finder中每次查找结果都有缓存（不会有泄露 因为缓存的应用都是Class Method这些）,可以通过EventBus的clearCache来清空。（我们可以考虑在onLowMemory的时候调用此方法）
SubscirberMethodFinder中有一个FindStat类用来帮助记录一次查找的中间状态和结果 以及帮助递归查找（里面有一个superClass的参数），每次遍历类和父类找到注册方法。  
FindStat中有一个checkAdd方法 来判断当前方法是否应该添加到subscribMethod中。在checkAdd方法中，会根据当前类是否注册过来判断是否可以添加。  


### EventBus的postSticky 和 post消息有什么区别
post过程见上。  
postSticky 先将消息放入到stickyEvents的map中 然后再调用post消息  

### EventBus的threadMode类型
POSTING:在发送事件的线程中通知订阅者  
MAIN:如果当前是主线程直接通知订阅者，否则调用HandlePoster的enque方法在主线程中通知订阅者  
BACKGROUD:如果当前不是主线程中则直接通知订阅者，否则启动一个后台线程来通知订阅者（不同的消息共用一个线程）  
ASYN:启动不同的线程来通知订阅者  

HandlerPoster:帮助主线程中的调度，其中维护一个PendingPost的队列，在handleMessage中依次弹出消息并通知。  

BackgroundPoster:帮助实现BACKGROUND模式，BackgroundPoster本身是一个runable，通过enque一个event对象，如果当前有线程正在处理post消息则直接在此线程中处理，否则会通过eventBus调用
builder中设置的线程池来执行线程（在run方法中循环取出消息队列中的消息来处理）  

AsyncPoster：帮助实现ASYN模式，本事是一个Runnable,enque一个event对象后会直接换起线程池来执行线程，在run中poll一个消息来处理  


### EventBus中的eventTypesCache是干什么用的
在eventInheritance为true的时候 会通过发送的event对象 查找event对象的所有父类和接口，这个cache就是用来缓存这个的。  
通过clearCaches来清空缓存

### 如何理解PendingPost
PendingPost是一次post的意图，其中定义了要发送的事件event 和 订阅方法 以及next 意图（用来和pendingPostQueue配合 实现队列）

### EventBus中用到了哪些设计模式
创建者模式： Builder  
订阅者模式： 整体就是订阅模式  

### *SubscriberMethodFinder中的FindStat是如何确定一个方法是否应该被记录的
（重点是FindStat中的2个临时变量 之后再做补充说明）


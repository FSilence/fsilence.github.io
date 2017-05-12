---
layout: post 
title: ReactNative调研报告  
description: ReactNative调研报告  
categories: ReactNative  
keywords: ReactNative, RN, reactnative, 跨平台  
---
2015年9月15日Facebook发布了ReactNative for Android，引发了学习使用ReactNative开发跨平台引用的热潮。目前React Native发布到了0.44的版本。最近项目中考虑是否要接入ReactNative, 所以对ReactNative进行了一些调研性的工作。  

## ReactNative简介
在reactnative中文网上有以下简介:  
<pre>
React Native使你能够在Javascript和React的基础上获得完全一致的开发体验，构建世界一流的原生APP。  
React Native着力于提高多平台开发的开发效率 —— 仅需学习一次，编写任何平台。(Learn once, write anywhere)  
Facebook已经在多项产品中使用了React Native，并且将持续地投入建设React Native。
</pre>
React Native是使用javascript和react来实现跨平台开发的, React Native提倡组件化开发: 即提供一个个封装好的组件，组件相互嵌套形成新的组件.你可以完全将App使用ReactNative编写，也可以使用原生和react native混合和的方式。由于目前React Native才发布到0.44版本，还没有发布正式的1.0, 其变动相对比较频繁，在react native升级后经常导致项目不能运行，你必须去做额外的工作来保证项目正常的运行。不过最近React Native的发布已经慢慢趋于稳定了，现在基本是1-2个月发一次版本更新，更新内容也没有之前变化那么大了。  

众所周知Android的碎片化和厂商的定制化是影响app稳定的很重要的原因之一,尤其是在国内，各个手机厂商都对Android系统进行了深度定制。这样的问题在ReactNative上也有很明显的体现，ReactNative在兼容Android各个系统版本和厂商版本的时候会暴露出不少的问题，目前React Native只支持Android 4.1以上，IOS 7.0以上的版本.附上最新的Android版本分布，[Android3月份分版本分布](https://zhuanlan.zhihu.com/p/25737474)可以看到Android 4.0的应用占比基本在%1左右，所以影响不大。  

在2017年3月爆发了一股苹果商店禁止热更新的事件，很多人都担心苹果是否会封掉Reactnative，其实苹果想要禁止的是热更新的功能。在苹果的拒绝邮件上也只是说明拒绝热更新的能力。从目前来看，这2个月苹果ReactNative依然可以过审。在各个论坛上也并未发现此次禁止热更新对ReactNative有较为明显的影响。

## ReactNative VS 原生 VS Hybrid
### 实现
(hybrid和 phoneGap APICloud AppCan等思想相同)
hybrid是一款h5的跨终端的解决方案，思想是 编写一次，就可以跨平台。
本质上hybrid还是web界面，只不过针对各个平台提供了一些调用native方法的api，通过js 和 native交互进行通信。  
用web渲染界面是比较耗时的，为了解决界面的渲染耗时，react-native将界面用原生实现，而js部分只负责描述界面并将描述后的界面通过js和native的交互机制通知给终端，让终端根据js传递过来的界面描述信息来选择本地的控件进行界面的拼装。react-native的思想是 学习一次，可以跨平台开发（仍需要各个终端的人配合）。

### 性能 
原生 > ReactNative > Hybrid  
具体的性能对比参数可以查看[H5、React Native、Native应用对比分析](http://blog.csdn.net/yczz/article/details/50468181)
### 开发&维护
以下成本和更新能力模块参考了[H5、React Native、Native应用对比分析](http://blog.csdn.net/yczz/article/details/50468181)
### 维护成本  
Hybrid < ReactNative < 原生  
Hybrid是H5的夸平台解决方案，ReactNative可以达到百分之90左右的代码共用，但仍需要终端支持。  

H5/Hybird： Web代码 ＋ iOS/Android平台支持  

React Native：可以一个开发团队 ＋ iOS/Android工程师；业务组件颗粒度小，不用把握全局即可修改业务代码。  

Native：iOS/Android开发周期长，两个开发团队。

总结：React Native 统一了开发人员技术栈，代码维护相对容易。 
#### 更新能力
H5/Hybird： 随时更新，适合做营销页面，目前携程一些BU全部都是H5页面；但是重要的部分还是Native。  

React Native：React Native部分可以热更新，bug及时修复。  

Native：随版本更新，尤其iOS审核严格，需要测试过关，否则影响用户。

## React-Canvas
为了解决webapp渲染dom的效率问题，Flipboard推出了一款react-canvas，它的思想是用做游戏的方式来做app。界面渲染针对到canvas上，通过重新定义绘制过程来减少web界面渲染时候的开销问题。  
react-canvas 也是使用的css-layout和react-native一致，也集成了react，只是最终界面是通过canvas展示的。
react-canvas因为是针对webapp的，所以它的灵活性要比react-native强，渲染的卡顿问题比较接近原生,由于react-canvas还是一个webapp所以有一些原生的界面效果不太好处理。react-canvas如果要和native交互需要单独提供本地的基础库和js和native的通信机制。
其实react-native可以理解成重写了一个简版的web浏览器，它只支持一些特殊的定制化界面，用native实现界面渲染来达到原生的体验效果。

## 案例
目前国内使用ReactNative的还不是很多（有些小的应用是完全用React Native来实现的）, 比较出名的是 携程的火车票模块， QQ空间，销售易的包中也发现了React Native的痕迹。要想看到更多的国内案例可以通过react native中文网收集的案例上查看: [ReactNative案例](http://reactnative.cn/cases.html)。  






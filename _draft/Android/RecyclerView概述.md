---
layout: post
title: RecyclerView 概述 
description: RecyclerView中关于ChildHelper的实现
keywords: RecyclerView 源码 
categories: Android
--- 
<pre>
大纲:
1. RecyclerView 的 onMeaure onLayout onDraw
2. RecyclerView.Adapter 的nofityDataChanged
</pre>
RecyclerView 是非常常见的一个控件，我准备做一个系列的RecyclerView 源码分析，旨在理解RecyclerView的内部工作原理，然后进一步学习借鉴相关的设计思想。  
本文我们将从RecyclerView.Adapter的nofity 和 onMeaure onLayout onDraw 分别入手, 来了解RecyclerView的一个工作的基本原理。  

onLayout 分为三步
第一步: 
计算动画信息, 进行记录

第二步骤:  
真正的onLayout 

第三步:
进行动画

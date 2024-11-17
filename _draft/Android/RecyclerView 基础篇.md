---
layout: post
title: RecyclerView 基础篇 
description: RecyclerView 基础使用
keywords: RecyclerView 基础 Basic LayoutManager LinearLayoutManager ItemDecoration ItemAnimator
categories: Android
--- 
<pre>
大纲:
1. RecyclerView 的使用
2. RecyclerView 常用LayoutManager
3. RecyclerView Adapter的实现
4. RecyclerView 添加Item动画
5. RecyclerView 添加分割线
</pre>

RecyclerView是为了大量数据下的显示提供各种复用和布局机制的View, 常用作对之前ListView 和 GrideView的替代。
## RecyclerView的使用
RecyclerView使用的时候除了要指明Adapter外 另外需要指明LayoutManager, 在代码中可以通过  
setLayoutManager 和  setAdapter指定， 另外layoutManager可以在xml布局文件中通过layoutManager字段配置。  

## LayoutManager
LayoutManager 是用来管理View的布局的, 系统提供了LinearLayoutManager GridLayoutManager  和 StaggeredGridLayoutManager, 分别表示线性布局，网格布局 和瀑布流布局。  

### LinearLayoutManager
线性布局 可以通过setOritation 设置是横向还是纵向。
oritaton == Vertical:  

oritation == HORIZONAL:  


### GridLayoutManager
网格布局, 通过setSpanCount 设置每一行要显示几个数据

### StaggeredGridLayoutManager
瀑布流，不固定高度，可以通过setSpanCount 控制一行显示个数, 可以通过StaggeredGridLayoutManager.LayoutParams#setFullSpan()方法，来强制当前元素填充一行，而非使用设置的spanCount.


## RecyclerView Adapter的实现
onCreateViewHolder 和 onBindViewHolder， 在onBindViewHolder中处理数据

## ItemAnimator
通过setItemAnimator 可以给RecyclerView的Item的增删改查设置动画, 系统提供了DefaultItemAnimator， 我们也可以通过自定义来实现动画
### 自定义ItemAnimator

## ItemDecoration 
可以通过addItemDecoration 给RecyclerView添加分割线,系统提供了SpaceItemDecoration, 我们也可以自定实现。
### 自定义ItemDecoration


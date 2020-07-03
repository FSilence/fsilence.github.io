---
layout: post
title: 扩大View的点击区域 
description: Android扩大View的点击区域
keywords: TouchDelegate View点击区域 扩充
categories: Android
---

## TouchDelegate

可以通过设置TouchDelegate 给View的父类来实现点击事件的区域扩充（拦截父View的Touch事件）

```java
   View child;
   ViewGroup parent;
   // 上下左右各扩充10px的点击范围
   int sizeDifference =  10;
   Rect delegateArea = new Rect();
   delegateArea.right += sizeDifference;
   delegateArea.bottom += sizeDifference;
   delegateArea.left -= sizeDifference;
   delegateArea.top -= sizeDifference;
   TouchDelegate delegate = new TouchDelegate(bounds, child);
   parent.setTouchDelegate(delegate);
```

系统提供的默认TouchDelegate有一定的局限性:  
* 一个ViewGroup下只能添加一个View的点击区域扩充
* 点击区域扩充, 不能超出父View的显示区间  

查看TouchDelegate的实现, 可以看到TouchDelegate的处理方式就是拦截Touch事件，然后判断事件范围是否是在扩充区域内，然后根据是否是点击，来决定事件坐标系是否转换成View内部的 并直接通过dispatchTouchEvent方法来触发delegateView的Click事件，来看下onTouchEvent中的实现：

```java
    public boolean onTouchEvent(@NonNull MotionEvent event) {
        int x = (int)event.getX();
        int y = (int)event.getY();
        boolean sendToDelegate = false;
        boolean hit = true;
        boolean handled = false;

        switch (event.getActionMasked()) {
            case MotionEvent.ACTION_DOWN:
               // 判断当前是否命中范围
                mDelegateTargeted = mBounds.contains(x, y);
                sendToDelegate = mDelegateTargeted;
                break;
            case MotionEvent.ACTION_POINTER_DOWN:
            case MotionEvent.ACTION_POINTER_UP:
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_MOVE:
                sendToDelegate = mDelegateTargeted;
                if (sendToDelegate) {
                    Rect slopBounds = mSlopBounds;
                    // 判断是否是hit
                    if (!slopBounds.contains(x, y)) {
                        hit = false;
                    }
                }
                break;
            case MotionEvent.ACTION_CANCEL:
                sendToDelegate = mDelegateTargeted;
                mDelegateTargeted = false;
                break;
        }
        if (sendToDelegate) {
            if (hit) {
               // 将Event坐标转换为View的中心位置并
                // Offset event coordinates to be inside the target view
                event.setLocation(mDelegateView.getWidth() / 2, mDelegateView.getHeight() / 2);
            } else {
                // 将Event坐标转换到View的触控范围以外, 避免触发按下态
                // Offset event coordinates to be outside the target view (in case it does
                // something like tracking pressed state)
                int slop = mSlop;
                event.setLocation(-(slop * 2), -(slop * 2));
            }
            // 分发事件
            handled = mDelegateView.dispatchTouchEvent(event);
        }
        return handled;
    }
```

为了解决多个View的问题我们可以将传入的bounds 和 delegateView 换成一个列表, 为了解决点击范围限制的问题, 我们可以将TouchDelegate 设置到RootView上，然后命中区间转换都转换成相对于RootView的相对坐标，而不是使用相对于父View的相对坐标。  
首先我们先定义一个类用来存储delegateView 和 bounds信息:  

```java
    class TouchDelegateItem {
        // 扩展点击范围
        Rect appendRect;

        // 扩展后的点击范围，相对于root
        Rect clickBounds = new Rect();

        // 扩展后的点击溢出范围, 相对于root
        Rect slopBounds = new Rect();

        // delegateView的弱引用
        WeakReference<View> viewRef;

        // 相对于Window的location 用于计算和root的相对坐标
        int[] location = new int[2];
    }
```

然后我们给TouchDelegateItem补充更新这些数据对应的方法：  

```java
        void updateClickBoundsAndSlop(int[] rootLocation, int slop) {
            if (viewRef == null || viewRef.get() == null) {
                return;
            }
            View view = viewRef.get();
            view.getLocationInWindow(location);
            clickBounds.set(view.getLeft(), view.getTop(), view.getRight(), view.getBottom());
            // 坐标转换 计算相对位置
            clickBounds.offsetTo(location[0] - rootLocation[0], location[1] - rootLocation[1]);
            // 追加扩展区域
            clickBounds.left -= appendRect.left;
            clickBounds.right += appendRect.right;
            clickBounds.top -= appendRect.top;
            clickBounds.bottom += appendRect.bottom;
            // 设置溢出区域 
            slopBounds.set(clickBounds);
            slopBounds.inset(-slop, -slop);
        }

        boolean contains(int x, int y) {
            return viewRef != null && viewRef.get() != null && clickBounds.contains(x, y);
        }

        boolean slopContains(int x, int y) {
            return viewRef != null && viewRef.get() != null && slopBounds.contains(x, y);
        }

```

然后回头来看TouchDelegate的构造方法，我们改造一下, 只接受一个ViewGroup用作root 然后通过方法可以追加代理的View：  

```java
    public IVOSTouchDelegate(ViewGroup viewGroup) {
        super(new Rect(), viewGroup);
        mSlop = ViewConfiguration.get(viewGroup.getContext()).getScaledTouchSlop();
        mRootView = viewGroup;
        viewGroup.setTouchDelegate(this);
    }
```

然后我们添加对TouchDelegateItem的增,删，查方法，因为涉及到View 所以我们要尽可能避免内存泄漏使用弱引用，并检查无效的数据及时清除:  

```java
    public void addDelegateItem(@NonNull View view, int left, int top, int right, int bottom) {
        if (mDelegateItems == null) {
            mDelegateItems = new ArrayList<>();
        }
        // 避免重复添加
        TouchDelegateItem item = removeDelegateItem(view);
        if (item != null) {
            item.appendRect.set(left, top, right, bottom);
        } else {
            item = new TouchDelegateItem(view, new Rect(left, top, right, bottom));
        }
        mDelegateItems.add(item);
    }

    public @Nullable
    TouchDelegateItem removeDelegateItem(View view) {
        if (mDelegateItems == null || mDelegateItems.isEmpty()) {
            return null;
        }
        for (int i = 0, n = mDelegateItems.size(); i < n; ) {
            TouchDelegateItem item = mDelegateItems.get(i);
            if (item.viewRef.get() == null) { // 清除无效数据
                mDelegateItems.remove(i);
            } else if (item.viewRef.get() == view) {
                return mDelegateItems.remove(i);
            } else {
                i++;
            }
        }
        return null;
    }

    private TouchDelegateItem getDelegateTarget(int x, int y, boolean needUpdate) {
        if (mCurTouchDelegateItem != null && mCurTouchDelegateItem.contains(x, y)) {
            return mCurTouchDelegateItem;
        }
        if (mDelegateItems == null || mDelegateItems.isEmpty()) {
            return null;
        }
        mRootView.getLocationInWindow(mRootLocation);
        for (int i = 0, n = mDelegateItems.size(); i < n; i++) {
            TouchDelegateItem item = mDelegateItems.get(i);
            if (needUpdate) {
                item.updateClickBoundsAndSlop(mRootLocation, mSlop);
            }
            if (item.slopContains(x, y)) {
                return item;
            }
        }
        return null;
    }
```

然后我们再添加一个总的开关: 

```java

    public void doDelegate(boolean doDelegate) {
        this.mDoDelegate = doDelegate;
    }

```

然后我们复写onTouchEvent方法，将父类中的事件处理copy出来进行改造，将坐标范围判断，改为从添加的TouchDelegateItem的列表中查找：  

```java
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        if (!mDoDelegate) {
            return false;
        }
        int x = (int) event.getX();
        int y = (int) event.getY();
        boolean hit = true;
        boolean handled = false;
        TouchDelegateItem cur = null;

        switch (event.getActionMasked()) {
            case MotionEvent.ACTION_DOWN:
                mCurTouchDelegateItem = getDelegateTarget(x, y, true);
                cur = mCurTouchDelegateItem;
                break;
            case MotionEvent.ACTION_POINTER_DOWN:
            case MotionEvent.ACTION_POINTER_UP:
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_MOVE:
                cur = mCurTouchDelegateItem;
                if (cur != null && cur.contains(x, y)) {
                    if (!cur.slopContains(x, y)) {
                        hit = false;
                    }
                }
                break;
            case MotionEvent.ACTION_CANCEL:
                cur = mCurTouchDelegateItem;
                mCurTouchDelegateItem = null;
                break;
            default:
                break;
        }
        if (cur != null) {
            final View delegateView = cur.viewRef.get();

            if (hit) {
                // Offset event coordinates to be inside the target view
                event.setLocation(delegateView.getWidth() / 2, delegateView.getHeight() / 2);
            } else {
                // Offset event coordinates to be outside the target view (in case it does
                // something like tracking pressed state)
                int slop = mSlop;
                event.setLocation(-(slop * 2), -(slop * 2));
            }
            handled = delegateView.dispatchTouchEvent(event);
        }
        return handled;

    }

```


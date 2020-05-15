---
layout: post
title: RecyclerView AdapterHelper源码
description: RecyclerView中关于AdapterHelper的实现
keywords: RecyclerView 源码 AdapterHelper 
categories: Android
--- 

<pre>
大纲：
1. UpdateOp
2. 如何管理和执行UpdateOp
</pre>

AdapterHelper是帮助RecyclerView 管理和执行更新操作的帮助类。RecyclerView将每一次更新操作封装成了一个UpdateOp操作，然后通过AdapterHelper进行管理和执行。我们先来看UpdateOp的数据结构:

```java
    static class UpdateOp {

        static final int ADD = 1;

        static final int REMOVE = 1 << 1;

        static final int UPDATE = 1 << 2;

        static final int MOVE = 1 << 3;

        static final int POOL_SIZE = 30;

        int cmd;

        int positionStart;

        Object payload;

        // holds the target position if this is a MOVE
        int itemCount;
    }
```
可以看到UpdateOp分别定义了ADD REMOVE UPDATE MOVE ，分别对应onItemChanged等等方法，其中itemCount在操作是MOVE时比较特殊表示Move到的具体的位置 而非数量，Move暂时只支持一个一个移动。  

我们先来看下AdapterHelper中的关键参数。  
```java
class AdapterHelper implements OpReorderer.Callback {
    ......
    private Pools.Pool<UpdateOp> mUpdateOpPool = new Pools.SimplePool<UpdateOp>(UpdateOp.POOL_SIZE);

    final ArrayList<UpdateOp> mPendingUpdates = new ArrayList<UpdateOp>();

    final ArrayList<UpdateOp> mPostponedList = new ArrayList<UpdateOp>();

    final Callback mCallback;
    ......
    // 用来对UpdateOp进行重排序，将Move操作的UpdateOp全部向后移动到末尾
    final OpReorderer mOpReorderer;
    // 记录当前存在哪些操作，典型的二进制标记为记录方式 
    private int mExistingUpdateTypes = 0;
    ......
}
```
mUpdateOpPool是UpdateOp的对象池，用来做UpdateOp对象的回收和复用的。然后内部将UpdateOp分成了2部分 mPendingUpdates 和 mPostponedList. 其中mPendingUpdates表示将要执行的操作列表, mPostponedList指的是需要延迟执行的操作。当添加一个新的UpdateOp时 它会首先进入mPendingUpdates，可以在addUpdateOp方法中看到:  
```java
  AdapterHelper addUpdateOp(UpdateOp... ops) {
        Collections.addAll(mPendingUpdates, ops);
        return this;
    }
``` 
然后我们来分别从 reProcess 和 consumePostponedUpdates 开始看起，之前说过当系统判断有动画时会提前执行reProcess 否则会直接执行consumePostponedUpdates,具体的触发逻辑可以翻看之前的Blog. 我们先从简单的看起consumePostponedUpdates:  
```java
   void consumePostponedUpdates() {
        final int count = mPostponedList.size();
        for (int i = 0; i < count; i++) {
            mCallback.onDispatchSecondPass(mPostponedList.get(i));
        }
        recycleUpdateOpsAndClearList(mPostponedList);
        mExistingUpdateTypes = 0;
    }
```
顾名思义此方法目的是消费在PostponedUpdates中的记录的UpdateOp, 从删除方法可以看到主要操作是依次去除mPostponedList中的UpdateOp然后依次调用mCallback.onDispatchSecondPass方法，mCallback是用来完成AdapterHelper和RecyclerView之间的交互的接口，我们先来看下在RecyclerView中onDispatchSecondPass的具体实现:  
```java
          void dispatchUpdate(AdapterHelper.UpdateOp op) {
                switch (op.cmd) {
                    case AdapterHelper.UpdateOp.ADD:
                        mLayout.onItemsAdded(RecyclerView.this, op.positionStart, op.itemCount);
                        break;
                    case AdapterHelper.UpdateOp.REMOVE:
                        mLayout.onItemsRemoved(RecyclerView.this, op.positionStart, op.itemCount);
                        break;
                    case AdapterHelper.UpdateOp.UPDATE:
                        mLayout.onItemsUpdated(RecyclerView.this, op.positionStart, op.itemCount,
                                op.payload);
                        break;
                    case AdapterHelper.UpdateOp.MOVE:
                        mLayout.onItemsMoved(RecyclerView.this, op.positionStart, op.itemCount, 1);
                        break;
                }
            }

            @Override
            public void onDispatchSecondPass(AdapterHelper.UpdateOp op) {
                dispatchUpdate(op);
            }
```
可以看到最终会根据UpdateOp中的类型分别处罚LayoutManager中的onItemsAdded onItemsRemoced onItemsUpdated onItemsMoved方法，查看LinearLayoutManager中的具体实现，可以看到这些都是系统提供的钩子方法，内部都是空实现。  

我们下面来看下重头戏 preProcess方法，前面说过当RecyclerView有item动画的时候，会先走preProcess 保留之前的视图来方便动画处理，我们直接看preProcess方法:  
```java

    void preProcess() {
        mOpReorderer.reorderOps(mPendingUpdates);
        final int count = mPendingUpdates.size();
        for (int i = 0; i < count; i++) {
            UpdateOp op = mPendingUpdates.get(i);
            switch (op.cmd) {
                case UpdateOp.ADD:
                    applyAdd(op);
                    break;
                case UpdateOp.REMOVE:
                    applyRemove(op);
                    break;
                case UpdateOp.UPDATE:
                    applyUpdate(op);
                    break;
                case UpdateOp.MOVE:
                    applyMove(op);
                    break;
            }
            if (mOnItemProcessedCallback != null) {
                mOnItemProcessedCallback.run();
            }
        }
        mPendingUpdates.clear();
    }
```
看到其中主要做了4件事  
1.将mPendingUpdates中的UpdateOp进行重排序，我们后面再具体看怎么进行的重排序，我们先说明这里的Reorder是将Move类型的操作移动到列表的末尾。  
2.根据类型一次调用apply方法，马上我们按UpdateOp类型来看相关操作  
3.如果从在mOnItemProcessedCallback 就触发一下mOnItemProcessedCallback是一个Runnable对象，也是用作钩子  
4.清空mPendingUpdates  
至于为什么要将Move操作向后移动延迟呢，个人理解是因为move会打乱数据链表的顺序， 而move的可能是add后的数据 所以move  和 add走延迟加载。而remove 和 update 尽量直接执行。
### applyAdd & applyMove
看源码看到都是走了 postponeAndUpdateViewHolders方法:  
```java
    private void postponeAndUpdateViewHolders(UpdateOp op) {
        if (DEBUG) {
            Log.d(TAG, "postponing " + op);
        }
        mPostponedList.add(op);
        switch (op.cmd) {
            case UpdateOp.ADD:
                mCallback.offsetPositionsForAdd(op.positionStart, op.itemCount);
                break;
            case UpdateOp.MOVE:
                mCallback.offsetPositionsForMove(op.positionStart, op.itemCount);
                break;
            case UpdateOp.REMOVE:
                mCallback.offsetPositionsForRemovingLaidOutOrNewView(op.positionStart,
                        op.itemCount);
                break;
            case UpdateOp.UPDATE:
                mCallback.markViewHoldersUpdated(op.positionStart, op.itemCount, op.payload);
                break;
            default:
                throw new IllegalArgumentException("Unknown update op type for " + op);
        }
    }
``` 
此方法主要完成2件事:  
1. 将UpdateOp加入到mPostponedList中   
2. 更新各个部分（包括缓存)的ViewHolder中的position记录的位置  
第一件事很简单，我们来看下offsetPositionsForAdd 方法，其它方法可以自行查看:  
```java
  @Override
  public void offsetPositionsForAdd(int positionStart, int itemCount) {
       offsetPositionRecordsForInsert(positionStart, itemCount);
       mItemsAddedOrRemoved = true;
  }

    void offsetPositionRecordsForInsert(int positionStart, int itemCount) {
        final int childCount = mChildHelper.getUnfilteredChildCount();
        for (int i = 0; i < childCount; i++) {
            final ViewHolder holder = getChildViewHolderInt(mChildHelper.getUnfilteredChildAt(i));
            if (holder != null && !holder.shouldIgnore() && holder.mPosition >= positionStart) {
                if (DEBUG) {
                    Log.d(TAG, "offsetPositionRecordsForInsert attached child " + i + " holder "
                            + holder + " now at position " + (holder.mPosition + itemCount));
                }
                holder.offsetPosition(itemCount, false);
                mState.mStructureChanged = true;
            }
        }
        mRecycler.offsetPositionRecordsForInsert(positionStart, itemCount);
        requestLayout();
    }
```
可以看到主要的功能是标记了有相关操作然后最终都是查找到ViewHolder然后调用了holder中的offsetPosition方法， mRecycler中的方法也是类似区别是offsetPosition中的第二个参数是true。 看一下Viewholder的实现：  
```java

        void offsetPosition(int offset, boolean applyToPreLayout) {
            if (mOldPosition == NO_POSITION) {
                mOldPosition = mPosition;
            }
            if (mPreLayoutPosition == NO_POSITION) {
                mPreLayoutPosition = mPosition;
            }
            if (applyToPreLayout) {
                mPreLayoutPosition += offset;
            }
            mPosition += offset;
            if (itemView.getLayoutParams() != null) {
                ((LayoutParams) itemView.getLayoutParams()).mInsetsDirty = true;
            }
        }
```
可以看到是记录了一下位置然后标记itemView的LayoutParams的mInsetsDirty为true，相关参数的具体职责我们之后介绍ViewHolder的时候再来介绍。  


### Remove & Update
看源码2部分的实现基本一致，可以自行查看源码. 方法基本将UpdateOp拆分为2类 一类会受到之前视图影响的 一类是不会的，判断依据如下 :  
```java
  for (int position = op.positionStart; position < tmpEnd; position++) {
            boolean typeChanged = false;
            RecyclerView.ViewHolder vh = mCallback.findViewHolder(position);
            if (vh != null || canFindInPreLayout(position)) {
                ......
            }
  }
  ......
    private boolean canFindInPreLayout(int position) {
        final int count = mPostponedList.size();
        for (int i = 0; i < count; i++) {
            UpdateOp op = mPostponedList.get(i);
            if (op.cmd == UpdateOp.MOVE) {
                if (findPositionOffset(op.itemCount, i + 1) == position) {
                    return true;
                }
            } else if (op.cmd == UpdateOp.ADD) {
                // TODO optimize.
                final int end = op.positionStart + op.itemCount;
                for (int pos = op.positionStart; pos < end; pos++) {
                    if (findPositionOffset(pos, i + 1) == position) {
                        return true;
                    }
                }
            }
        }
        return false;
    }
```
canFindInPreLayout的判断规则是，一次查找mPostponedList中的操作，计算出经过后续操作后的对应position 是不是和当前要判断的操作相等，相等即认为是受到影响的。这类操作最终都走到了postponeAndUpdateViewHolders方法，加入到mPostponedList中，另一类则通过dispatchAndUpdateViewHolders直接更新ViewHolder中对应的位置。
dispatchAndUpdateViewHolders 将每个item的操作逐步处理更新到mPostponedList中，并且归类后 走此方法dispatchFirstPassAndUpdateViewHolders， 更新对应的ViewHolder和Recycler中的ViewHolder的 preLayoutPos 和 pos  并添加对应的UPDATE 或 REMOVE的Flag。  

我们来总结下AdapterHelper中的preProcess方法，preProcess方法会会将操作全部更新到mPostponedList 并更新ViewHolder中的记录的值以保存当前视图，区别是:  
1. 对于Add和Move操作 添加到的mPostponedList 并更新对应的ViewHolder中操作后的position 
2. 对于Update 和 Remove操作，将操作拆分 受到当前mPostponedList中数据影响的部分同Add和Move的操作 追加到末尾，剩余的更新对当前mPostponedList的更新 并且对ViewHolder添加 REMOVE 和 UPDATE的Flag标志 以及更新preLayoutPosition参数  
ViewHolder中的参数的职能我们暂时先不管，之后会看到相关部分的使用。




---
layout: post
title: RecyclerView ChildHelper源码
description: RecyclerView中关于ChildHelper的实现
keywords: RecyclerView 源码 ChildHelper 
categories: Android
--- 
RecyclerView childHelper 源码分析

<pre>
大纲: 
1. ChildHelper的职责
2. hidden的处理方式
2. 关于Bucket的位计算
</pre>
## 概述
ChildHelper帮助处理RecyclerView中的View的辅助类。 RecyclerView中的childern被分成了2类一类是当前正在使用可见的，一类是hidden的。ChildHelper提供了2类方法，一类获取getChildCount等，一类是Unfilter的方法 例如getUnfilterChildCount ， 区别是getChildCount返回当前可见的View的个数，getUnfilterChildCount返回未过滤的View在ViewGroup中的实际个数。

## Unfilter
关注过滤掉的是什么，过滤掉的是不可见的View, 即被缓存起来留着复用的View
```java 
    // index为可见View中的下标，计算出真正的位置并返回
    View getChildAt(int index) {
        final int offset = getOffset(index);
        return mCallback.getChildAt(offset);
    }

    // index为view的实际位置
    View getUnfilteredChildAt(int index) {
        return mCallback.getChildAt(index);
    }

    // 获取可见View中位置index 对应的ViewGroup中的实际位置
    private int getOffset(int index) {
        if (index < 0) {
            return -1; //anything below 0 won't work as diff will be undefined.
        }
        final int limit = mCallback.getChildCount();
        int offset = index;
        while (offset < limit) {
            final int removedBefore = mBucket.countOnesBefore(offset);
            // 改成(index+removedBefore) - offset更好理解 可见的View的个数加上现在计算出的不可见的View的个数最终应该和实际的个数offset相同
            final int diff = index - (offset - removedBefore);
            if (diff == 0) {
                while (mBucket.get(offset)) { // ensure this offset is not hidden
                    offset++;
                }
                return offset;
            } else {
                offset += diff;
            }
        }
        return -1;
    }
```

## Bucket
用来帮助管理哪些View可见 哪些是hidden的，使用位计算来标记View的当前状态。
```java
    static class Bucket {

        static final int BITS_PER_WORD = Long.SIZE;

        static final long LAST_BIT = 1L << (Long.SIZE - 1);

        long mData = 0;

        Bucket mNext;

        ......
    }
```
用一个64位的二进制表示，哪一位为1表示这个位置的View是隐藏状态的 比如 00001010 表示下标为 1 和 3位置的View是隐藏的。当超过64的存储在mNext中
### set
```java
      // 标记某个位置的View 为hidden
      void set(int index) {
            // mData中只能记录64个信息，超过的部分记录到mNext中（方法很巧妙值得借鉴）
            if (index >= BITS_PER_WORD) {
                ensureNext();
                mNext.set(index - BITS_PER_WORD);
            } else {
                // 将对应位置标记为1
                mData |= 1L << index;
            }
        }
```
### insert 
```java
      // 修改某一位的值, value 为true 修改为1 为false 修改为0
      void insert(int index, boolean value) {
            // 超过64的交给mNext处理
            if (index >= BITS_PER_WORD) {
                ensureNext();
                mNext.insert(index - BITS_PER_WORD, value);
            } else {
                // 暂时记录最后一位的值，最终会移动到mNext中
                final boolean lastBit = (mData & LAST_BIT) != 0;
                // 帮助记录插入位置右边的记录，可以得到 00001111111 的标记
                long mask = (1L << index) - 1;
                // 记录插入位置右侧的数据
                final long before = mData & mask;
                // 记录插入位置左侧的数据 并左移一位 给插入的数据流出位置
                final long after = ((mData & ~mask)) << 1;
                mData = before | after;
                if (value) { // 插入数据
                    set(index);
                } else {
                    clear(index);
                }
                if (lastBit || mNext != null) {
                    ensureNext();
                    mNext.insert(0, lastBit);
                }
            }
        }
```

### remove 
```java
 // 移除某一位的数据
  boolean remove(int index) {
            // 超过64的交给mNext处理
            if (index >= BITS_PER_WORD) {
                ensureNext();
                return mNext.remove(index - BITS_PER_WORD);
            } else {
                // 获取到 100000000的mask
                long mask = (1L << index);
                // 获取要移除位置的值是否为1
                final boolean value = (mData & mask) != 0;
                // ~mask 的值大概为 1111111011111111 即清空要移除位置的值 将其记录为0用于之后循环右移
                mData &= ~mask;
                // mask 修改为 00000001111111111
                mask = mask - 1;
                // 获取到右侧(要删除区域前面的值)
                final long before = mData & mask;
                // 循环右移一位 高位补0 即移除既定位置后的上半区域的值
                // cannot use >> because it adds one.
                final long after = Long.rotateRight(mData & ~mask, 1);
                // 合并成最终的值
                mData = before | after;
                // 处理mNext
                if (mNext != null) {
                    if (mNext.get(0)) {
                        set(BITS_PER_WORD - 1);
                    }
                    mNext.remove(0);
                }
                return value;
            }
        }
```
### countOnesBefore

```java
        // 记录对应位置前有多少hidden状态的View
        int countOnesBefore(int index) {
            if (mNext == null) {
                if (index >= BITS_PER_WORD) {
                    return Long.bitCount(mData);
                }
                return Long.bitCount(mData & ((1L << index) - 1));
            }
            if (index < BITS_PER_WORD) {
                return Long.bitCount(mData & ((1L << index) - 1));
            } else {
                return mNext.countOnesBefore(index - BITS_PER_WORD) + Long.bitCount(mData);
            }
        }
```





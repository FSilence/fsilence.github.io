---
layout: post
title: RecyclerView UpdateOp的Reorder
description: RecyclerView中关于UpdateOp的Reorder
keywords: RecyclerView UpdateOp OpReorderer
categories: Android
---  
<pre>
大纲: 
1. 关于UpdateOp的重新排序
2. 对于各类操作移动转换的处理
</pre>
处理方式是将所有的move操作都移动到操作列表的最后，用来合并处理前面有相互交叉的操作。

向前依次找到第一个是Move 且 下一个不是Move操作的，然后尝试将Move操作向后移动一位。直到所有的Move操作都被沉到列表的末尾
## 关于Remove操作的交换
注意 Remove 类型的UpdateOp 的itemCount 表示要移动到的位置而非移动的个数， Remove操作暂时只支持一次移动一个
### 思想
1. 查找移动后move的操作被恢复的情况,即先做move操作后又做了删除操作，和做move前的数据位置不变， 此时直接将move操作丢弃即可。  
   此move 操作应该满足 是删除区间2边边界位置的数据移动，即将 删除的startPos - 1 的位置移动到 删除的区间endPos的位置或将书暗处区间的末尾endPos移动到startPos-1的位置  
2. 假设将要移动数据先移除了，然后再找到合适的位置插入,此时如果:
  a. 如果move操作将数据移动到了删除的区间，则直接删除此move的数据，删除区间-1即可  
  b. 如果移动的数据在删除数据的区间内，则此数据被移走后并未被删除 以此数据位置为边界拆分成2块要移除的区域
  c. 确定删除操作之后，移动数据的新的位置
### 具体实现  
```java
   // list 当前的操作集合，movePos 移动操作的下标 移动的操作对象， 删除操作的下标 删除对象
   void swapMoveRemove(List<UpdateOp> list, int movePos, UpdateOp moveOp,
            int removePos, UpdateOp removeOp) {
        // 扩展的删除操作，有情况会将删除操作拆分成2个删除区域，后面会看到      
        UpdateOp extraRm = null;
        // check if move is nulled out by remove
        // move操作是否可以直接移除
        boolean revertedMove = false;
        // move操作的移动方向是否是反向的，true表示向前移动 false 表示向后移动
        final boolean moveIsBackwards;
        // move操作的itemCount 表示要移动到的位置，而非移动的个数，当前只支持一个操作内移动一条数据
        if (moveOp.positionStart < moveOp.itemCount) {
            // 标记是向后移动
            moveIsBackwards = false;
            if (removeOp.positionStart == moveOp.positionStart
                    && removeOp.itemCount == moveOp.itemCount - moveOp.positionStart) {
                      // 正好从删除区域的前一个移动到删除区域的后一个，即删除后move的数据位置相当于无变化
                revertedMove = true;
            }
        } else {
            moveIsBackwards = true;
            if (removeOp.positionStart == moveOp.itemCount + 1
                    && removeOp.itemCount == moveOp.positionStart - moveOp.itemCount) {
                     // 正好从删除区域的后一个移动到删除区域的前一个，即删除后move的数据位置相当于无变化
                revertedMove = true;
            }
        }

        // going in reverse, first revert the effect of add
        if (moveOp.itemCount < removeOp.positionStart) {
          // move操作的最终位置在删除区间的前面， 如果取数据的位置也在删除区间前面则后面会恢复此数据
            removeOp.positionStart--;
        } else if (moveOp.itemCount < removeOp.positionStart + removeOp.itemCount) {
          // 移动的数据被移动到了删除的区间内，此时删除移动数据，然后相对应删除区间少1 
            // move is removed.
            removeOp.itemCount--;
            moveOp.cmd = REMOVE;
            moveOp.itemCount = 1;
            if (removeOp.itemCount == 0) {
              // 如果删除的操作的个数为0 则直接回收删除操作
                list.remove(removePos);
                mCallback.recycleUpdateOp(removeOp);
            }
            // no need to swap, it is already a remove
            return;
        }

        // now affect of add is consumed. now apply effect of first remove
        if (moveOp.positionStart <= removeOp.positionStart) {
            // 移动的开始位置在删除区间之前，恢复之前移动的startpos - 1的操作
            removeOp.positionStart++;
        } else if (moveOp.positionStart < removeOp.positionStart + removeOp.itemCount) {
            // 移动的数据的开始位置在区间内，删除的数据拆分成两部分
            final int remaining = removeOp.positionStart + removeOp.itemCount
                    - moveOp.positionStart;
            extraRm = mCallback.obtainUpdateOp(REMOVE, moveOp.positionStart + 1, remaining, null);
            removeOp.itemCount = moveOp.positionStart - removeOp.positionStart;
        }

        // if effects of move is reverted by remove, we are done.
        if (revertedMove) { // 删除移除的操作
            list.set(movePos, removeOp);
            list.remove(removePos);
            mCallback.recycleUpdateOp(moveOp);
            return;
        }

        // now find out the new locations for move actions
        if (moveIsBackwards) {
           // 计算新的坐标, 思想是将现在的位置去除掉删除的区域后重新标记
            if (extraRm != null) {
                if (moveOp.positionStart > extraRm.positionStart) {
                    moveOp.positionStart -= extraRm.itemCount;
                }
                if (moveOp.itemCount > extraRm.positionStart) {
                    moveOp.itemCount -= extraRm.itemCount;
                }
            }
            if (moveOp.positionStart > removeOp.positionStart) {
                moveOp.positionStart -= removeOp.itemCount;
            }
            if (moveOp.itemCount > removeOp.positionStart) {
                moveOp.itemCount -= removeOp.itemCount;
            }
        } else {
            if (extraRm != null) {
                if (moveOp.positionStart >= extraRm.positionStart) {
                    moveOp.positionStart -= extraRm.itemCount;
                }
                if (moveOp.itemCount >= extraRm.positionStart) {
                    moveOp.itemCount -= extraRm.itemCount;
                }
            }
            if (moveOp.positionStart >= removeOp.positionStart) {
                moveOp.positionStart -= removeOp.itemCount;
            }
            if (moveOp.itemCount >= removeOp.positionStart) {
                moveOp.itemCount -= removeOp.itemCount;
            }
        }

        list.set(movePos, removeOp);
        if (moveOp.positionStart != moveOp.itemCount) {
            // 数据确实有移动，则交换位置
            list.set(removePos, moveOp);
        } else {
            // 否则移除数据交换Move操作
            list.remove(removePos);
        }
        if (extraRm != null) {
            // 插入之前标记的额外删除区间
            list.add(movePos, extraRm);
        }
    }
```

## 关于Add操作的交换
### 思想
重新计算添加操作提前的位置。
### 具体实现
```java
   private void swapMoveAdd(List<UpdateOp> list, int move, UpdateOp moveOp, int add,
            UpdateOp addOp) {
        int offset = 0;
        // going in reverse, first revert the effect of add
        // 后后面的offset++ 一起看，即分成了3中情形：
        //     1. 数据移动和添加都发生在添加区域的左侧 或者右侧 则addPos位置不发生变化
        //     2. 数据是从添加区域的左侧移动到右侧 则addPos应该 + 1 （移动前添加的）
        //     3. 数据从添加区域的右侧移动到左侧 则addPos应该 - 1
        // move的位置重新计算则比较简单
        //     1. 如果移动的数据的起始位置在add的位置之后则移动位置想右移动添加的个数
        //     2. 如果移动的数据的目标位置在add的位置之后则移动位置想右移动添加的个数
        if (moveOp.itemCount < addOp.positionStart) {
            offset--;
        }
        if (moveOp.positionStart < addOp.positionStart) {
            offset++;
        }
        // 移动的节点在添加节点之后，则将添加提前后移动的位置要向后移动添加的个数
        if (addOp.positionStart <= moveOp.positionStart) {
            moveOp.positionStart += addOp.itemCount;
        }
        if (addOp.positionStart <= moveOp.itemCount) {
            moveOp.itemCount += addOp.itemCount;
        }
        addOp.positionStart += offset;
        list.set(move, addOp);
        list.set(add, moveOp);
    }
```

## 关于Update操作的交换
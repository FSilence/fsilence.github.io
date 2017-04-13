---
layout: post
categories: git
title: git分支恢复
description: git分支恢复
keywords: git, recover, branch
---
今天误删了git分支，还好可以恢复  


今天git push之后 以为merge了 就把本地和远程分支都删掉了，然后悲催的发现并没有执行merge操作。。还好分支可以恢复。。

这个时候我们可以从之前提交的那次merge request中查找到提交的版本号，然后本地执行  
<pre>
 git branch 分支名称 commit号
</pre>
然后重新push即可。。。


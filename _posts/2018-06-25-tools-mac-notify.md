---
layout: post
title: mac定时提醒 
categories: others
description: 用mac定时进行提醒 
keywords: test
---

最近想要保护下眼睛，所以想要让电脑没隔1个小时 提醒让眼睛休息2分钟。   
定时任务我们知道可以使用 crontab 进行，可以参考[http://www.cnblogs.com/peida/archive/2013/01/08/2850483.html](http://www.cnblogs.com/peida/archive/2013/01/08/2850483.html)  
唤起通知可以使用 AppleScript AppleScript是苹果的脚本可以用来帮助和mac交互。例如 display dialog "aaa" 表示弹窗提示 ”aaa“,   display notification "Content !" with title "Title" 表示右上角的提醒。其它命令可自行搜索。  

在bash中可以通过执行 osascript 来执行相应的命令 例如   

```script
osascript 'display notifiction "Content" with title "Title"'
```

---
layout: post
title: C++异常堆栈排查 
description: C++异常堆栈排查 
keywords: C++ 异常 堆栈 排查 
categories: C++
---

偶尔会写一部分C++的内容，碰到堆栈错误堆栈不清楚如何定位，特此记录一下，方便后续查阅。  
C++异常单行的错误信息如：addr  xxx <buildId>  可读性不好，需要进一步分析才能确定最终出错的      

流程概述：
1.  确认BuildId
2. 确认符号表是否被剥离
3. 将符号表还原到共享库
4. 查看堆栈对应的行
5. 分析错误原因

工具准备：
安装 binutils  
brew install binutils  
<pre>
注：brew install成功后需要单独配置PATH, 具体地址查看安装成功的信息。修改 ~/.bash_profile后，使用source ~/.bash_profile生效或重启命令行。
</pre>
安装 HopperDisassembler 或 ida pro  (可选，都是收费软件)

## 确认BuildId
使用 readelf (elf：Executable and Linking Format 可执行可关联的文件)命令查看对应so文件的buildId 和 堆栈buildId对比，确认是同一个。  
```
readelf -n xxx.so
```
或 使用file命令查看
```
file xxx.so
```

## 确认符号表是否被剥离
使用file xxx.so 命令可以查看。观察stripped字段，not stripped意味着未剥离可以直接查看信息，stripped需要进一步关联符号表。

## 还原符号表到共享库
如果符号表已删除，但是有一个未剥离符号表的so文件可以  
提取符号表：  
```
objcopy --only-keep-debug   libexample_debug.so libexample_debug.info
```
使用objcopy来恢复： 
```
objcopy --add-gnu-debuglink=my_program_debug my_program
```

通过
```
nm -n my_program 
```
验证符号表是否已恢复

<pre>
延伸：
确保生成so时保留了符号表：在编译时使用 -g 选项来生成调试信息。这会保留符号表，使调试工具能够正确工作：  
gcc -g -o my_program my_program.c
</pre>
 


## 分析错误原因

使用 addr2line 查看对应地址的代码位置：
```
addr2line -e my_program 0x4005b0 
```

使用 objdump -d 查看反汇编代码

使用 nm -n 查看符号表

还原函数：binutils里面提供了一个叫"c++filt"的工具可以用来解析被修饰过的名称 （可以尝试追加 -n）：
```
c++filt -n _ZN1N1C4funcEi
N::C::func(int)
```

分析汇编指令：
lar bl 等等

## 参考
https://blog.csdn.net/wdjjwb/article/details/86233389

---
layout: post
title: How to analyze the java memory when wired problems occor?
comments: true
author: "Yin Haomin"
tags:
    - java memory
    - analyze
---

本位我们讨论下当异常发生的时候，我们怎么分析内存的相关信息。

#### 讨论的内容适用于如下的场景:
1. 程序莫名的Hang，而log中没有相关的日志说明。
2. 程序中出现了OOM

#### 相关的排查方法

##### 程序Hang住

除了查看日志以外，还可以查看Java的堆栈

1. jps -lvm或者ps -ef | grep java  可以显示出程序，对应的进程号等

```
jps -lvm

ps -ef | grep java
```

2. 查看堆栈的信息

```
jstack -l <pid>
```

这样就可以看堆栈信息了，这个是实时刷的，可以 写入一个文件中进行查看：  jstack -l 219  > jstack.txt

 
jmap 能查看jvm内存中，对象占用内存的情况，还提供非常方便的命令将jvm的内存信息导出的文件。
 





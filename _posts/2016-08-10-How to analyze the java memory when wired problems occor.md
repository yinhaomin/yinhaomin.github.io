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
3. 线下测试就避免OOM的问题

#### 相关的排查方法

##### 程序Hang住

除了查看日志以外，还可以查看Java的堆栈

1 jps -lvm或者ps -ef | grep java  可以显示出程序，对应的进程号等

```
jps -lvm

ps -ef | grep java
```

2 查看堆栈的信息

```
jstack -l <pid>
```

这样就可以看堆栈信息了，这个是实时刷的，可以 写入一个文件中进行查看：  jstack -l 219  > jstack.txt

此时就可以看到堆栈Hang在哪里并可以排查问题了。

##### 程序中出现OOM

1 jmap查看jvm内存的占用情况

jmap 能查看jvm内存中，对象占用内存的情况，还提供非常方便的命令将jvm的内存信息导出的文件。

```
jmap -dump:format=b,file=heap.bin <pid>
```

我们也可以在实例运行的时候直接查看内存的占用

```
jmap -histo 219 | head -n 10
```

2 jhat 解析内存堆文件，生成相关的信息，并启动webServer提供查询

也就说，我们可以通过浏览器来看这些内存信息。jhat还提供了一个类sql的查询语言---OQL来给我们使用。

```
jhat -J-Xmx512m heap.bin  
```

不过我们这样分析的获取的信息是不多的，最好使用Memory Analyzer分析heap的信息，这样的话，得到的信息更加直观。

##### 线下测试避免OOM

1 我们可以使用Jprofiler绑定我们的测试，可以看到很多的程序相关的问题。

![gras](/img/Use_Jprofiler.png)

2 使用Eclipse的Memory Analyzer插件，改小测试可用的Memory，得出PPM的hprof文件，然后使用Memory Analyzer分析。







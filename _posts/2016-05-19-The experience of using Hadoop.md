---
layout: post
title: The experience of using Hadoop
comments: true
author: "Yin Haomin"
tags:
    - Experience
    - Hadoop
---

最近使用Hadoop的过程中，出现了几个问题，总结一下

#### 1. 使用Filesystem的时候，出现Filesystem closed的异常问题

##### 问题的原因

当读写文件的时候，Hadoop抛异常说文件系统已经关闭。
在一个多线程的程序中，FileSystem.get(getConf())返回的可能是一个cache中的结果，它并不是每次都创建一个新的实例。
这就意味着，如果每个线程都自己去get一个文件系统，然后使用，然后关闭，就会有问题。

##### 解决方案

1 设置Hadoop连接的Configuration属性disable对于FileSystem的缓存(这个方案试了下，似乎不行)

```
configuration.setBoolean("fs.hdfs.impl.disable.cache", true);
```

2 每次用完FileSystem都不手动关闭，就好了

#### 2. 在本地运行测试时没有问题，但是打包后，在tomcat中运行出现NoClassDefFoundError: org/apache/hadoop/conf/Configuration

##### 问题的原因

NoClassDefFoundError comes when a class is not visible at run time but was at compile time. Which may be related to JAR files, because all the required class files were not included.

##### 解决方案

You should already added hadoop-core.jar in your build path, so no compile error detected in your program. But you get the error when you run it, because hadoop-core is dependent on commons-logging.jar (as well as some other jars). You may need to add the jars under /lib to your build path.

相关的链接[java.lang.NoClassDefFoundError in Hadoop Basics' MapReduce Program](http://stackoverflow.com/questions/13776795/java-lang-noclassdeffounderror-in-hadoop-basics-mapreduce-program)



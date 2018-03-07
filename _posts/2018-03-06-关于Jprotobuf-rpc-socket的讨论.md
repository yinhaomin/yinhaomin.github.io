---
layout: post
title: 关于Jprotobuf-rpc-socket的讨论
comments: true
author: "Yin Haomin"
date: 2018-03-07 01:00:00
tags:
    - Jprotobuf-rpc-socket
    - Jprotobuf
    - 讨论
---

#### 问题背景<br>
我们要开发一个内部使用的平台，考虑系统交互的稳定性，打算使用RPC框架进行交互。考虑到之前在百度开发使用RPC框架的经验，果断使用了Baidu [Jprotobuf-rpc-socket](https://github.com/baidu/Jprotobuf-rpc-socket)，这里是[user guide](https://github.com/baidu/Jprotobuf-rpc-socket/wiki/User-Guide)。<br>
但是这里存在一个问题，百度的框架虽然开源了，但是市面上用的太少了，导致资料QA，经验贴和教程等等都太少了。我整理一下Demo，也整理一下之前使用的问题，期望自此开始积累Jprotobuf-rpc-socket的使用经验和教程资料。<br>


---
layout: post
title: How to design an IM
comments: true
author: "Yin Haomin"
tags:
    - System design
    - IM
---

In this blog, I want to explore how to design an IM, for designing and developing a IM system has always been my expectation, thus I have studied some IM system design and tried to design it. Here we go.

#### Here are some important questions to be answered

1. How to know someone is online?
2. How to get the information informed to the server and to the user?
3. How to make two online user send messages rapidly and messages are in order?
4. How to transfer information in limited network environment, like 2G network?
5. What tools or open source framework can we use?
6. What if a server fail down?
7. What if there are large scale of network transport?

In the following, we will try to solve the questions above.

##### 1. How to know someone is online?

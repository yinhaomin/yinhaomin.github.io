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

1. How to know someone is online or offline?
2. How to get the information informed to the server and to the user?
3. How to make two online user send messages rapidly and messages are in order?
4. How to transfer information in limited network environment, like 2G network?
5. What tools or open source framework can we use?
6. What if a server fail down?
7. What if there are large scale of network transport?

In the following, we will try to solve the questions above.

##### 1. How to know someone is online or offline?

Not only the user need to know if their friends are online or not, but also the server need to know it, so that to take some needed measures. So easily, we can let the client send a message regularly to the server, and client regularly request for their friends status from server.

However, when the user number grows too large and the cocurrent number becomes so big, that the server can hardly bear the network traffic. There is several strategies to solve this problem.

1. Lower the frequency of the status synchronizing, like every 5 or 10 minutes, to sync the status.
2. Sync with pre actions, ex: If the client has taken some moves, like some clicks or refresh, only then, request will be sent.
3. Rank the user, if the user is less active on the IM, or the other user talks so few to this one, the others's status change will not so frenquently updated to this user.

##### 2. How to get the information informed to the server and to the user?

We can use the websocket or the comet to make the client and server communicate. And there are two kind of clients, web client and application client. For different client, strategy may differs.

Firstly, for the web client.





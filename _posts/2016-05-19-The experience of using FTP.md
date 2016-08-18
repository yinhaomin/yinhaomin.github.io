---
layout: post
title: The experience of using FTP
comments: true
author: "Yin Haomin"
tags:
    - Experience
    - FTP
---

最近使用FTP的过程中，出现了几个问题，总结一下

#### 1. 我们连接远程FTP出现问题，FTP连接后，无法上传和下载文件

##### 问题的原因

FTP是一种文件传输协议，它支持两种模式,一种方式叫做Standard (也就是 Active,主动方式，或者叫PORT模式),一种是 Passive (也就是PASV,被动方式)。 Standard模式 FTP的客户端发送 PORT 命令到FTP server。Passive模式FTP的客户端发送 PASV命令到 FTP Server。  
下面介绍一个这两种方式的工作原理:   
Standard模式FTP 客户端首先和FTP Server的TCP 21端口建立连接，通过这个通道发送命令，客户端需要接收数据的时候在这个通道上发送PORT命令。 PORT命令包含了客户端用什么端口接收数据。在传送数据的时候，服务器端通过自己的TCP 20端口发送数据。FTP server必须和客户端建立一个新的连接用来传送数据。  
缺省情况下Standard模式的数据端口是20，控制端口是21（控制端口可以设定，本文假定使用21）。当进行连接时,客户端使用一个随机的端口N（N大于1024)连接服务器的控制端口21，然后客户端开始监听端口N+1，并向服务器发送命令PORT N+1，服务器用自己的数据端口20连回客户的N+1端口。
由于Standard模式仅仅是发送端口给服务器，由服务器连回客户端，如果客户端有防火墙，这样的连接会被认为是外部主机试图连接内部的主机，通常情况下是不允许的。

Passive模式在建立控制通道的时候和Standard模式类似，当客户端通过这个通道发送PASV 命令的时候，FTP server打开一个位于1024和5000之间的随机端口并且通知客户端在这个端口上传送数据的请求，然后FTP server 将通过这个端口进行数据的传送，这个时候FTP server不再需要建立一个新的和客户端之间的连接。
当进行连接时,客户端使用一个随机的端口N（N大于1024)
连接服务器的控制端口21，并向服务器发送命令PASV，服务器使用一个随机的数据端口M(M>1024)并发回客户端,客户端用数据端口N+1连接服务器的端口M。由于客户端发起数据连接，这样就解决了防火墙带来的问题。

##### 解决方案

修改起来很简单（以下对java来说，并且ftp服务使用的是org.ache.commons.net.ftp包）：
在建立发ftp client的连接之前或者之后添加ftpClient.enterLocalPassiveMode();即可（目前采用在连接建立之后添加此操作） 


#### 2. 使用匿名的FTP的时候，出现链接大量处于WAIT_STATE的问题

这个问题是由如下代码引起的，这段代码试图判断一个FTP路径的文件是否存在。

```
    /**
     * 判断一个文件在FTP上是否存在
     * 
     * @param filePath
     * @return
     * @throws Exception
     */
    public static boolean checkFileExists(String url, int maxContinuousEmptyLinesAllowed, String charset,
            Integer connectTimeout, Integer readTimeout) throws Exception {
        URLConnection connection = null;
        InputStream inputStream = null;
        try {
            URL remoteUrl = new URL(url);
            connection = (URLConnection) remoteUrl.openConnection();
            connection.setConnectTimeout(connectTimeout == null ? 5000 : connectTimeout.intValue());
            connection.setReadTimeout(readTimeout == null ? 5000 : readTimeout.intValue());
            connection.connect();
            inputStream = connection.getInputStream();
            if (inputStream == null) {
                return false;
            } else {
                return true;
            }
        } catch (Exception e) {
            e.printStackTrace();
            throw new Exception("Exception in FtpUtil.checkFileExists: " + e.getMessage(), e);
        } finally {
            if (inputStream != null) {
                try {
                    inputStream.close();
                    inputStream = null;
                } catch (Exception e) {
                    // No use exception.
                }
            }
            if (connection != null) {
                try {
                    connection = null;
                } catch (Exception e) {
                    // No use exception.
                }
            }
        }
    }
```

##### 问题的原因

待分析

##### 解决方案

我们删除了该段代码，并使用FTPClient判断路径是否存在。并调整了FTP的设置，具体见这里[Reduce TIME_WAIT socket connections](http://www.linuxbrigade.com/reduce-time_wait-socket-connections/)

#### 3. 当下载的文件很大的时候，没有root的权限不能设置FTP的下载文件大小限制

##### 解决方案

在Linux中启动Http服务器下载数据

```
python -m SimpleHTTPServer 8080
```





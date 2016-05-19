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

#### 1. 使用匿名的FTP的时候，构造出来的FTPClient访问FTP出现InputStream返回为null，并且返回码为530的问题

##### 问题的原因

实际上此时FTP根本没有连上去。无论是Client端还是Server的问题，当有问题时，现InputStream返回为null。

##### 解决方案

采用另外的方式，而不是FTPClient访问FTP地址

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





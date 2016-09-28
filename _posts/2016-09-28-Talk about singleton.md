---
layout: post
title: Talk about singleton
comments: true
author: "Yin Haomin"
tags:
    - singleton
    - enum
    - Double-Check Locking
---

今天来总结下单例模式，Effectice Java书中推荐使用Enum实现单例。然而，我们在实际工作中，依然不习惯，为了将错误扭过来，我实现了一个使用Enum的单例，
并在下面讨论一下，单例的其他实现方式。

#### 单例的Enum实现

我们要实现拿到本机的IP地址。在Enum中我们的代码是这样的

```
public enum LocalIPAddress {
    INSTANCE;
    private String ipAddress;
    private LocalIPAddress() {
        System.out.println("Running");
        ipAddress = IpUtil.getIp();
    }

    public String getIpAddress() {
        return ipAddress;
    }
}

```

分析一下代码:

我们需要获取ipAddress，在第一次使用Enum INSTANCE的时候，private LocalIPAddress()会被调用，而之后就不会被调用。调用getIpAddress()方法，
就获得了我们需要的ip。

在测试代码中

```
    @Test
    public void testGetIp() {
        String ipAddress = LocalIPAddress.INSTANCE.getIpAddress();
        System.out.println(ipAddress);
        ipAddress = LocalIPAddress.INSTANCE.getIpAddress();
        System.out.println(ipAddress);
        ipAddress = LocalIPAddress.INSTANCE.getIpAddress();
        System.out.println(ipAddress);
    }
```

输出的结果为:

```
Running
xx.46.189.xx
xx.46.189.xx
xx.46.189.xx
```

我们可以看到，这样就使用枚举实现了单例。

未完待续。。。


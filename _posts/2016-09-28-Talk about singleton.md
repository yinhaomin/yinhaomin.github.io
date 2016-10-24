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

今天来总结下单例模式，Effectice Java书中推荐使用Enum实现单例。然而，我们在实际工作中，依然不习惯，为了将不好的习惯扭过来，我实现了一个使用Enum的单例，并在下面讨论一下，单例的其他实现方式。

#### 单例的Enum实现

我们要实现拿到本机的IP地址。在Enum中我们的代码是这样的:

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

##### 分析一下代码:

我们需要获取ipAddress，在第一次使用Enum INSTANCE的时候，private LocalIPAddress()会被调用，而之后就不会再被调用。调用getIpAddress()方法，
就获得了我们需要的ip。

在测试代码中: 

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

#### 使用Double checked locking实现单例

以下的代码，及其每一行注释了为什么要这样实现的原因

```
public class Singleton{
    // 使用volatile，可以保证对其读写是有序的，并且是按照地址写入的
    // 否则，假如A线程在初始化singleton 的时候，当初始化未完成，而B线程前来读取，就会读取到错误的object.
    private static volatile Singleton singleton = null;

    private Singleton(){}

    private Singleton(Context context){}

    public Singleton getInstance(Context context){
        // 可以有效的降低volatile的访问次数，因为volatile的访问开销较大
        Singleton singletonInside = singleton;
        if(singletonInside == null){
            // 可以降低synchronized的访问次数，因为synchronized的访问开销很大
            synchronized(Singleton.class){
                 // 可能会被外部赋值
                 singletonInside = singleton;
                 if(singletonInside == null){
                     singletonInside = new Singleton(context);
                     singleton = singletonInside;
                 }
            }
        }
        return singletonInside;
    }
}
```





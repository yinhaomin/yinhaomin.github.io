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

开个Post一起讨论百度的Jprotobuf-rpc-socket

#### 背景说明
考虑多内部系统交互的稳定性，我们一般使用RPC框架进行交互，我在百度开发使用过Baidu [Jprotobuf-rpc-socket](https://github.com/baidu/Jprotobuf-rpc-socket)，这里是[User guide](https://github.com/baidu/Jprotobuf-rpc-socket/wiki/User-Guide)。<br>
百度的框架虽然开源了，但是市面上用的太少了，导致资料经验贴和教程等等都太少了，期望自此开始积累Jprotobuf-rpc-socket的使用经验和教程资料。这是我整理的[Jprotobuf-rpc-demo](https://github.com/yinhaomin/Jprotobuf-rpc-demo)，里面有使用的详细的说明。<br>

#### 问题1：不能支持Proxy对象的服务发布
当我们发布一个代理类的时候，在下面代码中会出现问题：<br>
```
com.baidu.jprotobuf.pbrpc.server.RpcServiceRegistry

    /**
     * Register service.
     *
     * @param target the target
     */
    public void registerService(final Object target) {
        if (target == null) {
            throw new IllegalArgumentException("Param 'target' is null.");
        }
        Class<? extends Object> cls = target.getClass();
        ReflectionUtils.doWithMethods(cls, new ReflectionUtils.MethodCallback() {
            public void doWith(Method method) throws IllegalArgumentException, IllegalAccessException {
                ProtobufRPCService protobufPRCService = method.getAnnotation(ProtobufRPCService.class);
                if (protobufPRCService != null) {
                    doRegiterService(method, target, protobufPRCService);
                }
            }
        });
    }
```

在运行的时候，下面的代码，拿到的类为"class com.sun.proxy.$Proxy81"。<br>
```
Class<? extends Object> cls = target.getClass();
```
这就导致在下面的语句中，我们拿不到需要注册的方法.<br>
```
ProtobufRPCService protobufPRCService = method.getAnnotation(ProtobufRPCService.class);
```
实际上我们的实现代码是这样的。<br>
```
@Slf4j
@RpcExporter(port = "1033", rpcServerOptionsBeanName = "rpcServerOptions")
@Service(DemoConstants.PMP_SERVICE)
public class PmpServiceImpl implements PmpService {

    @Autowired
    private HelloWorldRunService helloWorldRunService;

    @ProtobufRPCService(serviceName = DemoConstants.PMP_SERVICE, methodName = "helloWorld")
    @Override
    public HelloWorldResponse helloWorld(HelloWorldRequest request) {
        log.info("Called by request:" + request);
        HelloWorldResponse response = helloWorldRunService.helloWorld(request);
        return response;
    }

}
```

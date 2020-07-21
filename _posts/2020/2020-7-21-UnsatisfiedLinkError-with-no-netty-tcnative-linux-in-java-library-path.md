---
title: UnsatisfiedLinkError with no netty tcnative linux in java library path
date: 2020-7-21 09:16:38 +0800
categories: [QAS]
tags: [netty]
---

## Q: 使用netty，代码加载过程中报java.lang.UnsatisfiedLinkError: no netty_tcnative_linux_x86_64 in java.library.path

```
2020-07-20 15:47:52.421 DEBUG [main] io.netty.util.internal.NativeLibraryLoader:140 - netty_tcnative_linux_x86_64 cannot be loaded from java.library.path, now trying export to -Dio.netty.native.wor
kdir: /tmp
java.lang.UnsatisfiedLinkError: no netty_tcnative_linux_x86_64 in java.library.path
        at java.lang.ClassLoader.loadLibrary(ClassLoader.java:1867)
        at java.lang.Runtime.loadLibrary0(Runtime.java:870)
        at java.lang.System.loadLibrary(System.java:1122)
        at io.netty.util.internal.NativeLibraryUtil.loadLibrary(NativeLibraryUtil.java:38)
        at io.netty.util.internal.NativeLibraryLoader.loadLibrary(NativeLibraryLoader.java:349)
        at io.netty.util.internal.NativeLibraryLoader.load(NativeLibraryLoader.java:136)
        at io.netty.util.internal.NativeLibraryLoader.loadFirstAvailable(NativeLibraryLoader.java:96)
        at io.netty.handler.ssl.OpenSsl.loadTcNative(OpenSsl.java:554)
        at io.netty.handler.ssl.OpenSsl.<clinit>(OpenSsl.java:135)
        at io.grpc.netty.GrpcSslContexts.defaultSslProvider(GrpcSslContexts.java:217)
        at io.grpc.netty.GrpcSslContexts.configure(GrpcSslContexts.java:144)
        at io.grpc.netty.GrpcSslContexts.forClient(GrpcSslContexts.java:93)
```

## A: netty-tcnative编译过程中区分OS架构，Windows编译的包，在Linux上运行不适配

Maven依赖坐标增加classifier，例如：

```
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-tcnative-boringssl-static</artifactId>
            <version>2.0.30.Final</version>
			<classifier>linux-x86_64</classifier>
        </dependency>
```

## S: 开源包加载出错，优先考虑平台兼容性和版本兼容性


## 参考

- https://github.com/netty/netty/issues/8703
- https://blog.csdn.net/HaegThe/article/details/81779172

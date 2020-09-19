##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 **Android Zygote 启动分析** 知识点，通过阅读本篇文章，您将收获以下内容:
>一、Android 系统基本服务
>二、虚拟机创建和第一个Java 程序引导
>三、Dalvik 虚拟机基本配置
>四、Zygote 启动流程
>五、Zygote 启动分析
>六、Zygote 创建system_server主要方法
>七、Zygote 创建System_server 分析
>八、Zygote 创建应用
>九、Zygote 创建应用流程
>十、Zygote 预加载资源
>十一、Zygote 预加载的目的
>十二、优化Zygote 启动方法： 线程池
>十三、fork  SystemServer



# 一、 Android 系统基本服务

`Android `系统包含`netd`、`servicemanager`、`surfaceflinger`、`zygote`、`media`、`installd`、`bootanimation` 等基本服务，具体作用请看下图。


![Android 系统基本服务](https://upload-images.jianshu.io/upload_images/5851256-3c8865e5dff2343f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 二、虚拟机创建和第一个Java 程序引导

为了让`APK`在不同的虚拟机都可以运行，`Google` 采取了适配器模式，在让虚拟机运行之前先执行 `dexopt` ，即将`dex `文件优化成`odex` 文件，可以让虚拟机更加优化的执行。

在`ART `虚拟机中，`dexopt` 将`dex`文件优化成二进制格式的问题，从而可以让`ART`虚拟机执行。`dexopt `会调用`dex2oat` 进行优化，`dex2oat` 的任务是将原来的`dex`文件进行预翻译，从而可以加快`app`运行的时间，但是由于某些`app`比较复杂，所以优化的时间就比较长。
优化是以`dex `文件中的`Method `方法为单位，`dex2oat` 在优化时候，会根据需求优化一定量的`Method`，即不是所有的`Method `都回翻译成`oat`模式。



![虚拟机创建和第一个Java 程序引导](https://upload-images.jianshu.io/upload_images/5851256-8fe8391d26ef7dec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 三、Dalvik 虚拟机基本配置

在`Android `系统中，`Dalvik` 虚拟机 和`ART`、应用程序进程，以及运行系统的关键服务`SystemServer `进程都是由 `Zygote `进程创建孵化的。

## 1.Dalvik 虚拟机基本配置 


![Dalvik 虚拟机基本配置](https://upload-images.jianshu.io/upload_images/5851256-6f661fb0945e8380.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 四、Zygote 启动流程

## 1.Zygote 启动代码

`Zygote` 服务时通过 `init.rc`进程启动的，`Zygote` 的 `classname` 为`main `.
`init.rc `文件配置代码如下：
```
... ... 
on nonencrypted
    class_start main
    class_start late_start

on property:sys.init_log_level=*
    loglevel ${sys.init_log_level}

... ...
```
详细可以参考 `init.rc`启动分析。
[Android 9.0 init 启动流程](https://mp.weixin.qq.com/s/8sDgxLLI8jYAxTiq28jBXw)

##2.Zygote  main 函数
`app_main.cpp `是`Zygote`进程的`main`函数，`frameworks\base\cmds\app_process\app_main.cpp`


`Zygote ` 是由 `init.rc` 脚本启动，在`init `脚本中，我们可以看到会导入`import /init.${ro.zygote}.rc` 脚本
```
# Copyright (C) 2012 The Android Open Source Project
#
# IMPORTANT: Do not create world writable files or directories.
# This is a common source of Android security bugs.
#

import /init.environ.rc
import /init.usb.rc
import /init.${ro.hardware}.rc
import /vendor/etc/init/hw/init.${ro.hardware}.rc
import /init.usb.configfs.rc
... ...
import /init.${ro.zygote}.rc

... ...
```
在 `system/core/rootdir`目录下，会根据`ro.zygote`属性值不同，启动不同的脚本，主要包含以下四个`zygote `脚本。

- 1.init.zygote32.rc  支持32为系统
- 2.init.zygote32_64.rc
- 3.init.zygote64.rc
- 4.init.zygote64_32.rc


![init.zygte.rc脚本](https://upload-images.jianshu.io/upload_images/5851256-bb2680d085b46c3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




![Zygote 启动流程](https://upload-images.jianshu.io/upload_images/5851256-60407c4139d7cecc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 五、Zygote 启动分析



![Zygote 启动分析](https://upload-images.jianshu.io/upload_images/5851256-9fec21753b9df9c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#六、Zygote 创建system_server主要方法

![Zygote 创建system_server主要方法](https://upload-images.jianshu.io/upload_images/5851256-24d03353b3ca0f5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 七、Zygote 创建System_server 分析



![Zygote 创建System_server](https://upload-images.jianshu.io/upload_images/5851256-557b2f403e8a7be0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 八、Zygote 创建应用


![Zygote 创建应用](https://upload-images.jianshu.io/upload_images/5851256-4811bf4f3b9b0626.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 九、Zygote 创建应用流程


![Zygote 创建应用流程](https://upload-images.jianshu.io/upload_images/5851256-d6de79d0af0f772f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 十、Zygote 预加载资源


![Zygote 预加载资源](https://upload-images.jianshu.io/upload_images/5851256-48890df4448efece.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![preloadClasses()](https://upload-images.jianshu.io/upload_images/5851256-43f5baed800a41f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![preloadResources()](https://upload-images.jianshu.io/upload_images/5851256-11367b12cbcc0c0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 十一、Zygote 预加载的目的


![Zygote 预加载的目的](https://upload-images.jianshu.io/upload_images/5851256-ee3918951dba3b85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 十二、优化Zygote 启动方法： 线程池

## 1.Zygote 启动优化 
- 1：加载类和资源是可重入操作，所以在并行模式下，不存在互斥的场景
- 2：`Android`提供了`Executors`和`ExecutorService`多线程类，因此可以使用多线程来加载类和资源。
- 3：硬件平台最好是多核，否则加速也不明显；


![线程池 优化Zygote 启动](https://upload-images.jianshu.io/upload_images/5851256-b85f1cbb92368a7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2.Zygote 启动优化实质 

使我们的进程最大限度的抢占CPU

#十三、fork SystemServer

经过一系列初始化后，在`ZygoteInit`类中 `forkSystemServer`,为启动`SystemServer` 做准备。`ZygoteInit.java`代码路径如下：`alps\frameworks\base\core\java\com\android\internal\os\ZygoteInit.java`
```

    /**
     * Prepare the arguments and forks for the system server process.
     *
     * Returns an {@code Runnable} that provides an entrypoint into system_server code in the
     * child process, and {@code null} in the parent.
     */
    private static Runnable forkSystemServer(String abiList, String socketName,
            ZygoteServer zygoteServer) {
        long capabilities = posixCapabilitiesAsBits(
            OsConstants.CAP_IPC_LOCK,
            OsConstants.CAP_KILL,
            OsConstants.CAP_NET_ADMIN,
            OsConstants.CAP_NET_BIND_SERVICE,
            OsConstants.CAP_NET_BROADCAST,
            OsConstants.CAP_NET_RAW,
            OsConstants.CAP_SYS_MODULE,
            OsConstants.CAP_SYS_NICE,
            OsConstants.CAP_SYS_PTRACE,
            OsConstants.CAP_SYS_TIME,
            OsConstants.CAP_SYS_TTY_CONFIG,
            OsConstants.CAP_WAKE_ALARM,
            OsConstants.CAP_BLOCK_SUSPEND
        );
        /* Containers run without some capabilities, so drop any caps that are not available. */
        StructCapUserHeader header = new StructCapUserHeader(
                OsConstants._LINUX_CAPABILITY_VERSION_3, 0);
        StructCapUserData[] data;
        try {
            data = Os.capget(header);
        } catch (ErrnoException ex) {
            throw new RuntimeException("Failed to capget()", ex);
        }
        capabilities &= ((long) data[0].effective) | (((long) data[1].effective) << 32);

        /* Hardcoded command line to start the system server */
        String args[] = {
            "--setuid=1000",
            "--setgid=1000",
            /// M: [Wi-Fi Hotspot Manager] system_server add dhcp (1014) group to access
            /// "/data/misc/dhcp/dnsmasq.leases"
            "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1014,1018,1021,1023," +
                        "1024,1032,1065,3001,3002,3003,3006,3007,3009,3010",
            "--capabilities=" + capabilities + "," + capabilities,
            "--nice-name=system_server",
            "--runtime-args",
            "--target-sdk-version=" + VMRuntime.SDK_VERSION_CUR_DEVELOPMENT,
            "com.android.server.SystemServer",
        };
        ZygoteConnection.Arguments parsedArgs = null;

        int pid;

        try {
            parsedArgs = new ZygoteConnection.Arguments(args);
            ZygoteConnection.applyDebuggerSystemProperty(parsedArgs);
            ZygoteConnection.applyInvokeWithSystemProperty(parsedArgs);

            boolean profileSystemServer = SystemProperties.getBoolean(
                    "dalvik.vm.profilesystemserver", false);
            if (profileSystemServer) {
                parsedArgs.runtimeFlags |= Zygote.PROFILE_SYSTEM_SERVER;
            }

            /* Request to fork the system server process */
            pid = Zygote.forkSystemServer(
                    parsedArgs.uid, parsedArgs.gid,
                    parsedArgs.gids,
                    parsedArgs.runtimeFlags,
                    null,
                    parsedArgs.permittedCapabilities,
                    parsedArgs.effectiveCapabilities);
        } catch (IllegalArgumentException ex) {
            throw new RuntimeException(ex);
        }

        /* For child process */
        if (pid == 0) {
            if (hasSecondZygote(abiList)) {
                waitForSecondaryZygote(socketName);
            }

            zygoteServer.closeServerSocket();
            return handleSystemServerProcess(parsedArgs);
        }

        return null;
    }

```
 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


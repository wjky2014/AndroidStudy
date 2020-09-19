#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、前言
>二、编写AIDL文件
>三、编写Manager类
>四、 编写系统服务
>五、 注册系统服务
>六、注册Manager
>七、App调用
>八、添加JNI部分代码
>九、总结

#一、前言

系统服务是Android中非常重要的一部分, 像ActivityManagerService, PackageManagerService, WindowManagerService, 这些系统服务都是Framework层的关键服务, 本篇文章主要讲一下如何基于Android源码添加一个系统服务的完整流程, 除了添加基本系统服务, 其中还包含添加JNI部分代码和App通过AIDL调用的演示Demo, 调用包含App调用服务端, 也包含服务端回调App, 也就是完成一个简单的双向通信.

注: 测试代码基于Android 7.1.1, 其他Android版本都是大同小异.

#二、编写AIDL文件

添加服务首先是编写AIDL文件, AIDL文件路径如下:

```
frameworks/base/core/java/com/example/utils/
```
##1.ISystemEvent.aidl 内容如下:

```
package com.example.utils;

import com.example.utils.IEventCallback;

interface ISystemEvent {
    void registerCallback(IEventCallback callback);

    void unregisterCallback(IEventCallback callback);

    void sendEvent(int type, String value);
}

```

## 2.IEventCallback.aidl 内容如下

```
package com.example.utils;

interface IEventCallback
{
    oneway void onSystemEvent(int type, String value);
}

```

AIDL文件编写, 教程很多, 我这里就不详细说明了, 需要注意的是, 由于我们要实现回调功能, 所以必须写一个回调接口 IEventCallback, 另外AIDL文件中 oneway 关键字表明调用此函数不会阻塞当前线程, 调用端调用此函数会立即返回, 接收端收到函数调用是在Binder线程池中的某个线程中. 可以根据实际项目需求选择是否需要加 oneway 关键字.

AIDL只支持传输基本java类型数据, 要想传递自定义类, 类需要实现 Parcelable 接口, 另外, 如果传递基本类型数组, 需要指定 in out 关键字, 比如 `void test(in byte[] input, out byte[] output)` , 用 in 还是 out, 只需要记住: 数组如果作为参数, 通过调用端传给被调端, 则使用 in, 如果数组只是用来接受数据, 实际数据是由被调用端来填充的, 则使用 out, 这里之所以没有说服务端和客户端, 是因为 in out 关键字用哪个和是服务端还是客户端没有联系, 远程调用和被调用更适合描述.

文件写完后, 添加到编译的 Android.mk 中 LOCAL_SRC_FILES 后面:

##3.frameworks/base/Android.mk

```
LOCAL_SRC_FILES += \
    core/java/android/view/IWindow.aidl \
    core/java/android/view/IWindowFocusObserver.aidl \
    core/java/android/view/IWindowId.aidl \
    部分代码省略 ...
    core/java/com/example/utils/ISystemEvent.aidl \
    core/java/com/example/utils/IEventCallback.aidl \
    部分代码省略 ...

```

编译代码, **编译前需执行 make update-api**, 更新接口, 然后编译代码,确保AIDL编写没有错误, 编译后会生成对应java文件, 服务端要实现对应接口.

#三、编写Manager类

我们可以看到, Android API 中有很多Manager类, 这些类一般都是某个系统服务的客户端代理类, 其实我们不写Manager类, 只通过AIDL文件自动生成的类, 也可以完成功能, 但封装一下AIDL接口使用起来更方便, 我们测试用的Manager类为 SystemEventManager, 代码如下:
`
frameworks/base/core/java/com/example/utils/SystemEventManager.java
`
```
package com.example.utils;

import android.content.Context;
import android.os.RemoteException;
import android.util.Log;

import com.example.example.ISystemEvent;
import com.example.IEventCallback;

public class SystemEventManager {

    private static final String TAG = SystemEventManager.class.getSimpleName();
    // 系统服务注册时使用的名字, 确保和已有的服务名字不冲突
    public static final String SERVICE = "test_systemevent";

    private final Context mContext;
    private final ISystemEvent mService;

    public SystemEventManager(Context context, ISystemEvent service) {
        mContext = context;
        mService = service;
        Log.d(TAG, "SystemEventManager init");
    }

    public void register(IEventCallback callback) {
        try {
            mService.registerCallback(callback);
        } catch (RemoteException e) {
            Log.w(TAG, "remote exception happen");
            e.printStackTrace();
        }
    }

    public void unregister(IEventCallback callback) {
        try {
            mService.unregisterCallback(callback);
        } catch (RemoteException e) {
            Log.w(TAG, "remote exception happen");
            e.printStackTrace();
        }
    }

    /**
     * Send event to SystemEventService.
     */
    public void sendEvent(int type, String value) {
        try {
            mService.sendEvent(type, value);
        } catch (RemoteException e) {
            Log.w(TAG, "remote exception happen");
            e.printStackTrace();
        }
    }
}

```

代码很简单, 就封装了下AIDL接口, 定义了系统服务注册时用的名字.
```
public SystemEventManager(Context context, ISystemEvent service)
```
构造函数中的 ISystemEvent 参数在后面注册Manager时候会通过Binder相关接口获取.

编译代码, 确保没有错误, 下面编写系统服务.

#四、 编写系统服务

路径以及代码如下:
` 
frameworks/base/services/core/java/com/android/server/example/SystemEventService.java
` 
```
package com.android.server.example;

import android.content.Context;
import android.os.Binder;
import android.os.RemoteCallbackList;
import android.os.RemoteException;
import android.os.ServiceManager;
import android.util.Log;

import com.example.utils.ISystemEvent;
import com.example.utils.IEventCallback;

public class SystemEventService extends ISystemEvent.Stub {

    private static final String TAG = SystemEventService.class.getSimpleName();
    private RemoteCallbackList<IEventCallback> mCallbackList = new RemoteCallbackList<>();

    private Context mContext;

    public SystemEventService(Context context) {
        mContext = context;
        Log.d(TAG, "SystemEventService init");
    }

    @Override
    public void registerCallback(IEventCallback callback) {
        boolean result = mCallbackList.register(callback);
        Log.d(TAG, "register pid:" + Binder.getCallingPid()
                + " uid:" + Binder.getCallingUid() + " result:" + result);

    }

    @Override
    public void unregisterCallback(IEventCallback callback) {
        boolean result = mCallbackList.unregister(callback);
        Log.d(TAG, "unregister pid:" + Binder.getCallingPid()
                + " uid:" + Binder.getCallingUid() + " result:" + result);

    }

    @Override
    public void sendEvent(int type, String value) {
        sendEventToRemote(type, value + " remote");
    }

    public void sendEventToRemote(int type, String value) {
        int count = mCallbackList.getRegisteredCallbackCount();
        Log.d(TAG, "remote callback count:" + count);
        if (count > 0) {
            final int size = mCallbackList.beginBroadcast();
            for (int i = 0; i < size; i++) {
                IEventCallback cb = mCallbackList.getBroadcastItem(i);
                try {
                    if (cb != null) {
                        cb.onSystemEvent(type, value);
                    }
                } catch (RemoteException e) {
                    e.printStackTrace();
                    Log.d(TAG, "remote exception:" + e.getMessage());
                }
            }
            mCallbackList.finishBroadcast();
        }
    }
}

```

服务端继承自 ISystemEvent.Stub, 实现对应的三个方法即可, 需要注意的是, 由于有回调功能, 所以要把注册的 IEventCallback 加到链表里面, 这里使用了 RemoteCallbackList, 之所以不能使用普通的 List 或者 Map, 原因是, 跨进程调用, App调用 registerCallback 和 unregisterCallback 时, 即便每次传递的都是同一个 IEventCallback 对象, 但到服务端, 经过跨进程处理后, 就会生成不同的对象, 所以不能通过直接比较是否是同一个对象来判断是不是同一个客户端对象, Android中专门用来处理跨进程调用回调的类就是 RemoteCallbackList, RemoteCallbackList 还能自动处理App端异常死亡情况, 这种情况会自动移除已经注册的回调.

RemoteCallbackList 使用非常简单, 注册和移除分别调用 register() 和 unregister() 即可, 遍历所有Callback 稍微麻烦一点, 代码参考上面的 sendEventToRemote() 方法.

可以看到, 我们测试用的的系统服务逻辑很简单, 注册和移除 Callback 调用 RemoteCallbackList 对应方法即可, sendEvent() 方法在App端调用的基础上, 在字符串后面加上 " remote" 后回调给App, 每个方法也加了log方便理解流程, 服务端代码就完成了.

#五、 注册系统服务

代码写好后, 要注册到SystemServer中, 所有系统服务都运行在名为 system_server 的进程中, 我们要把编写好的服务加进去, SystemServer中有很多服务, 我们把我们的系统服务加到最后面, 对应路径和代码如下:
` 
frameworks/base/services/java/com/android/server/SystemServer.java
` 
```

import com.android.server.example.SystemEventService;
import com.example.utils.SystemEventManager;

/**
 * Starts a miscellaneous grab bag of stuff that has yet to be refactored
 * and organized.
 */
private void startOtherServices() {
    // 部分代码省略...
    // start SystemEventService
    try {
        ServiceManager.addService(SystemEventManager.SERVICE,
                    new SystemEventService(mSystemContext));
    } catch (Throwable e) {
        reportWtf("starting SystemEventService", e);
    }
    // 部分代码省略...
}

```

通过 ServiceManager 将服务加到SystemServer中, 名字使用 SystemEventManager.SERVICE, 后面获取服务会通过名字来获取. 此时, 如果直接编译运行, 开机后会出现如下错误:

```
E SystemServer: java.lang.SecurityException

E SELinux : avc:  denied  { add } for service=test_systemevent pid=1940 uid=1000 scontext=u:r:system_server:s0 tcontext=u:object_r:default_android_service:s0 tclass=service_manager permissive=0

```

这个是没有Selinux权限, 我们需要加上添加服务的权限, 代码如下:

首先定义类型, test_systemevent 要和添加服务用的名字保持一致
` 
system/sepolicy/service_contexts
` 
```
wifiscanner                               u:object_r:wifiscanner_service:s0
wifi                                      u:object_r:wifi_service:s0
window                                    u:object_r:window_service:s0
# 部分代码省略...
test_systemevent                          u:object_r:test_systemevent_service:s0
*                                         u:object_r:default_android_service:s0

```

`system/sepolicy/service.te`

```
# 加入刚刚定义好的 test_systemevent_service 类型, 表明它是系统服务
type test_systemevent_service, system_api_service, system_server_service, service_manager_type;

```

加入上面代码后, 编译刷机开机后, 服务就能正常运行了.

#六、注册Manager

系统服务运行好了, 接下来就是App怎么获取的问题了, App获取系统服务, 我们也用通用接口:
`context.getSystemService()`
在调用 getSystemService() 之前, 需要先注册, 代码如下:

`frameworks/base/core/java/android/app/SystemServiceRegistry.java`

```
import com.example.utils.ISystemEvent;
import com.example.utils.SystemEventManager;

static { 
    // 部分代码省略, 参考其他代码, 注册Manger
    registerService(SystemEventManager.SERVICE, SystemEventManager.class,
            new CachedServiceFetcher<SystemEventManager>() {
        @Override
        public SystemEventManager createService(ContextImpl ctx) {
            // 获取服务
            IBinder b = ServiceManager.getService(SystemEventManager.SERVICE);
            // 转为 ISystemEvent
            ISystemEvent service = ISystemEvent.Stub.asInterface(b);
            return new SystemEventManager(ctx.getOuterContext(), service);
        }});
}

```

注册后, 如果你在App里面通过 getSystemService(SystemEventManager.SERVICE); 获取Manager并调用接口, 会发现又会出错, 又是Selinux权限问题:

```
E SELinux : avc:  denied  { find } for service=test_systemevent pid=4123 uid=10035 scontext=u:r:untrusted_app:s0:c512,c768 tcontext=u:object_r:test_systemevent_service:s0 tclass=service_manager permissive=0

```

说是没有 find 权限, 因此又要加权限, 修改代码如下:
`system/sepolicy/untrusted_app.te`

```
# 允许 untrusted_app 查找  test_systemevent_service
allow untrusted_app test_systemevent_service:service_manager find;

```

这个 Selinux 的知识有兴趣自己去学一下, 报了什么权限, 就按照错误信息去对应文件添加权限.

至此, 系统代码修改完成了, 编译系统刷机, 下面通过App调用.

#七、App调用

文件拷贝和准备:
我们需要复制三个文件到App中, 两个AIDL文件, 一个Manager文件:

```
IEventCallback.aidl
ISystemEvent.aidl
SystemEventManager.java

```

所有AIDL文件和java文件, 在App工程中的包名和路径都需要和系统保持一致, 这三个文件App不能做任何修改, 除非系统源码中也做对应修改, 总的来说, 这三个文件App和系统中要完全保持一致, 类名包名和包路径都需一致, 复制这三个文件到工程中后, 编译后, 调用方式如下.

获取服务:

```
SystemEventManager eventManager = (SystemEventManager)     
        context.getSystemService(SystemEventManager.SERVICE);

```

这里Android Studio可能会报 getSystemService() 参数不是Context里面的某个服务的错误, 可以直接忽略, 不影响编译.

注册/取消注册:

```
eventManager.register(eventCallback);

eventManager.unregister(eventCallback);

private IEventCallback.Stub eventCallback = new IEventCallback.Stub() {
    @Override
    public void onSystemEvent(int type, String value) throws RemoteException {
        Log.d("SystemEvent", "type:" + type + " value:" + value);
    }
};

```

调用:

```
eventManager.sendEvent(1, "test string");

```

测试Log如下:

```
D SystemEventManager: SystemEventManager init
D SystemEventService: register pid:3944 uid:10035 result:true
D SystemEventService: remote callback count:1
D SystemEvent: type:1 value:test string remote
D SystemEventService: unregister pid:3944 uid:10035 result:true

```

可以看到调用了服务端并成功收到服务端拼接的字符串.

#八、添加JNI部分代码

我们一般添加系统服务, 可能是为了调用驱动里面的代码, 所有一般要用JNI部分代码, 这里不是讲怎么编写JNI代码, 而是说下系统服务中已有的JNI代码, 我们可以直接在这基础上增加我们的功能.

JNI部分代码位置为:
```
frameworks/base/services/core/jni/
```

编译对应mk为:
```
frameworks/base/services/Android.mk
frameworks/base/services/core/jni/Android.mk
```
此部分代码直接编译为 libandroid_servers 动态库, 在SystemServer进行加载:
`frameworks/base/services/java/com/android/server/SystemServer.java`

```
// Initialize native services.
System.loadLibrary("android_servers");

```

如果需要添加JNI部分代码, 直接在 `frameworks/base/services/core/jni/ `目录下增加对应文件,
在`frameworks/base/services/core/jni/Android.mk`中加入新增文件进行编译即可.
同时按照已有文件中JNI函数注册方式, 写好对应注册方法, 统一在
`frameworks/base/services/core/jni/onload.cpp `中动态注册函数.
关于JNI动态注册知识, 可参考之前写的一篇文章: [两种JNI注册方式](https://www.jianshu.com/p/1d6ec5068d05)

#九、总结

从上面一个完整的流程下来, 基本就理解了我们平常调用 getSystemService() 具体是怎么工作的, 总体来说也不麻烦, 真正有技术含量的跨进程调用被隐藏起来了, 我们只管按照规则调用接口即可,以上就是Android系统中添加一个系统服务和App调用的完整流程, 如有疑问, 欢迎讨论!
文章转载网络，原文链接如下：
[原文链接](https://www.jianshu.com/p/0f46a95d576f)


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

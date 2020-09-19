 
##### 和您一起终身学习，这里是程序员Android 


本篇文章主要介绍 `Android` 开发中的 **AMS**部分知识点，通过阅读本篇文章，您将收获以下内容:
>1. AMS简单关系
>2. AMS 构造函数
>3. AMS 父类
>4. AMS 常用
>5. AMS部分方法实现




# 1. AMS简单关系

AMS 继承实现关系图

![AMS 继承实现关系图](https://upload-images.jianshu.io/upload_images/5851256-4f9d3e5ab047e403.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

AMS代码路径
`\frameworks\base\services\core\java\com\android\server\ActivityManagerService.java`

# 2.AMS 构造函数

AMS 构造函数思维导图

![AMS 构造函数思维导图 一](https://upload-images.jianshu.io/upload_images/5851256-31f496cbf56f7b97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![AMS 构造函数思维导图 二](https://upload-images.jianshu.io/upload_images/5851256-321ba417e1bbbda6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

AMS 继承实现方法如下

![AMS 测试构造函数](https://upload-images.jianshu.io/upload_images/5851256-a6fa6a382a6ef906.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

AMS 构造方法是在主线程上调用，但可能需要附加各种处理程序到其他线程，因此要注意区分Looper

![AMS 构造函数 一 ](https://upload-images.jianshu.io/upload_images/5851256-6920d2fbd66cefc5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![AMS 构造函数 二 ](https://upload-images.jianshu.io/upload_images/5851256-888b5b2d17ae1f80.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![AMS构造函数 三 ](https://upload-images.jianshu.io/upload_images/5851256-d656b0a4b451b0dc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 3. AMS 父类 IActivityManager.Stub

ActivityMangerService 父类是由`frameworks\base\core\java\android\app\IActivityManager.aidl`文件生成，可以实现跨进程通讯。此类同提供多种接口方法，共不同的进程调用。

# 4. AMS 常用变量


- 1. 控制CPU 电池检测时间
```
/** Control over CPU and battery monitoring */
    // write battery stats every 30 minutes.
    static final long BATTERY_STATS_TIME = 30 * 60 * 1000;
    // don't sample cpu less than every 5 seconds.
    static final long MONITOR_CPU_MIN_TIME = 5 * 1000;
```
- 2. 广播超时,事件分发超时，网络连接超时 时间
```
// How long we allow a receiver to run before giving up on it.
    static final int BROADCAST_FG_TIMEOUT = 10*1000;
    static final int BROADCAST_BG_TIMEOUT = 60*1000;
 // How long we wait until we timeout on key dispatching.
    static final int KEY_DISPATCHING_TIMEOUT = 5*1000;
  /**
     * Default value for {@link Settings.Global#NETWORK_ACCESS_TIMEOUT_MS}.
     */
    private static final long NETWORK_ACCESS_TIMEOUT_DEFAULT_MS = 200; // 0.2 sec
```

# 5. AMS 部分方法实现

- KillHandler 内部类实现

![KillHandler 内部类实现 ](https://upload-images.jianshu.io/upload_images/5851256-8e52cc918432a315.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- UiHandler 内部实现 

![UiHandler 内部实现Crash ANR 等问题 ](https://upload-images.jianshu.io/upload_images/5851256-a548c94790461695.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![ensureBootCompleted](https://upload-images.jianshu.io/upload_images/5851256-df71ce26c7ace73a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![finishBooting 方法](https://upload-images.jianshu.io/upload_images/5851256-61813b1deb753f10.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![enableScreenAfterBoot ](https://upload-images.jianshu.io/upload_images/5851256-2bc3f7b8edc54577.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![enableScreenAfterBoot ](https://upload-images.jianshu.io/upload_images/5851256-2eff97247d03293f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![hideBootMessagesLocked](https://upload-images.jianshu.io/upload_images/5851256-d527880d2cba6469.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  实现父类的一些方法

![实现父类接口中的一些方案](https://upload-images.jianshu.io/upload_images/5851256-052ca9bf38214fe6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 判断是否是前台app的方法

![isAppForeground](https://upload-images.jianshu.io/upload_images/5851256-b280f6fd3e4b3490.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 多窗口 以及画中画模式

![多窗口  以及画中画模式](https://upload-images.jianshu.io/upload_images/5851256-b4b3bbb0e30d5f65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  PROCESS INFO 接口服务类

![ PROCESS INFO 接口服务类 ](https://upload-images.jianshu.io/upload_images/5851256-b0cd6f9e8e4b170a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- PermissionController 接口类

![PermissionController ](https://upload-images.jianshu.io/upload_images/5851256-4dba8045a9f3267f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- IntentFirewallInterface Intent防火墙 接口

![IntentFirewallInterface](https://upload-images.jianshu.io/upload_images/5851256-17b4591135c7d930.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- checkCallingPermission 方法
 
![checkCallingPermission](https://upload-images.jianshu.io/upload_images/5851256-89c2483ec73dabeb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- findUriPermissionLocked 相关方法

![findUriPermissionLocked](https://upload-images.jianshu.io/upload_images/5851256-d632a4dba88a0edd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- getMemoryInfo 获取内存信息

![getMemoryInfo](https://upload-images.jianshu.io/upload_images/5851256-478f5f4a21d6561a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- TASK MANAGEMENT 相关方法实现

![TASK MANAGEMENT 相关方法实现](https://upload-images.jianshu.io/upload_images/5851256-e5d144a642815960.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- createRecentTaskInfoFromTaskRecord 方法

![createRecentTaskInfoFromTaskRecord](https://upload-images.jianshu.io/upload_images/5851256-fe6b1643305bbba0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 - getTaskSnapshot 

![getTaskSnapshot](https://upload-images.jianshu.io/upload_images/5851256-75851346cde465d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- removeTasksByPackageNameLocked 方法

![removeTasksByPackageNameLocked](https://upload-images.jianshu.io/upload_images/5851256-9b1bb99d1b0facda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 实现部分 kill 进程的方法 killProcessesBelowForeground killUid

![实现部分 kill 进程的方法](https://upload-images.jianshu.io/upload_images/5851256-9c8ef8f308b40c3f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 实现 restart 方法

![实现 restart 方法](https://upload-images.jianshu.io/upload_images/5851256-8209965ab1567be4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- systemReady 方法

![systemReady 方法](https://upload-images.jianshu.io/upload_images/5851256-269d2f2cc27924c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- handleApplicationCrash 相关方法实现
![handleApplicationCrash 相关方法实现](https://upload-images.jianshu.io/upload_images/5851256-794f7d8d20a21bff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- addErrorToDropBox

将Crash  WTF ANR 信息导入到Drop box

![addErrorToDropBox ](https://upload-images.jianshu.io/upload_images/5851256-971c308d47046392.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- getProcessesInErrorState 状态信息

![getProcessesInErrorState ](https://upload-images.jianshu.io/upload_images/5851256-005f9772603f9254.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 获取正在运行的app 进程 getRunningAppProcesses

![getRunningAppProcesses](https://upload-images.jianshu.io/upload_images/5851256-a97e1d91fb20aa59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- dump方法实现

![dump](https://upload-images.jianshu.io/upload_images/5851256-ad78d9627e17fb0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- dumpOomLocked
![image.png](https://upload-images.jianshu.io/upload_images/5851256-d49326d17c82ccbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![dumpActivity](https://upload-images.jianshu.io/upload_images/5851256-2fda2fc18a3caf98.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- cleanUpApplicationRecordLocked

![cleanUpApplicationRecordLocked ](https://upload-images.jianshu.io/upload_images/5851256-36e4907a6ed005de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 实现 Service 相关方法
1.getServices 
2.getRunningServiceControlPanel 
3.startService
4.stopService
5.peekService
6.stopServiceToken
7.setServiceForeground
8.bindService
9.unbindService
10.publishService
11.unbindFinished
12.serviceDoneExecuting

![Service 相关方法](https://upload-images.jianshu.io/upload_images/5851256-82554ab7aeeca2c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Service 相关方法](https://upload-images.jianshu.io/upload_images/5851256-29e469e51a5afd7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Service 相关方法](https://upload-images.jianshu.io/upload_images/5851256-c0af7f1fc6a1f137.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- BACKUP AND RESTORE

![BACKUP AND RESTORE](https://upload-images.jianshu.io/upload_images/5851256-6054211f9cf02163.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- BROADCASTS 相关方法

![BROADCASTS](https://upload-images.jianshu.io/upload_images/5851256-be99aedffdedda9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![BROADCASTS](https://upload-images.jianshu.io/upload_images/5851256-814ad62ea5a6eaee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![BROADCASTS](https://upload-images.jianshu.io/upload_images/5851256-f7109d5c51853918.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- INSTRUMENTATION 仪表仪器相关
![INSTRUMENTATION ](https://upload-images.jianshu.io/upload_images/5851256-4f57c77a471096fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- update Configuration更新相关
![updateConfiguration](https://upload-images.jianshu.io/upload_images/5851256-fbe49959dce7fd16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- computeOomAdjLocked
![computeOomAdjLocked](https://upload-images.jianshu.io/upload_images/5851256-9d0d2e52f68411a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- LocalService

![LocalService](https://upload-images.jianshu.io/upload_images/5851256-654ff2be52da47ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- waitForNetworkStateUpdate

![waitForNetworkStateUpdate](https://upload-images.jianshu.io/upload_images/5851256-3c3a1176e705d56d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- AppTaskImpl 接口类

![AppTaskImpl 接口类](https://upload-images.jianshu.io/upload_images/5851256-8b1a634d1ad0d1f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

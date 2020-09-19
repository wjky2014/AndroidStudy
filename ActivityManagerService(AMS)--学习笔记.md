##### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
 
>一、AMS 是如何启动的？
>二、通过 Lifecycle 启动初始化 AMS 系统服务
>三、AMS 构造函数实例实现
>四、启动AMS.Start()方法实现
>五、AMS设置系统进程实现
>六、AMS 安装系统 Provider实现
>七、AMS.systemReady准备完成
>八、AMS 启动 Launcher实现


# 一、AMS  是如何启动的？

AMS`（ActivityManagerService 简称）`是通过SystemServer.java `(frameworks\base\services\java\com\android\server\SystemServer.java)`类中的 `startBootstrapServices() ` 方法启动的。

## 1. startBootstrapServices主要作用
startBootstrapServices 主要是启动以下引导服务，引导系统Service 起来。
部分通过系统服务如下：
1. Installer安装服务、 
2. Activity管理服务AMS、
3. 电源管理服务PowerManagerService、
4. Recovery管理服务RecoverySystemService、
5. LED 背光显示服务LightsService、
6. 显示管理服务DisplayManagerService、
7. app 包管理服务PackageManagerService等。

## 2.AMS启动流程

 

AMS启动大致流程如下：

1. SystemServer.main 
2. SystemServer.SystemServer()
3. SystemServer.run() 
4. SystemServer.startBootstrapServices()

 

AMS启动详细流程如下：
```

public final class SystemServer {
... ...

    /**
     * The main entry point from zygote.
     */
    public static void main(String[] args) {
        new SystemServer().run();
    }

    public SystemServer() {
        ... ...
        mRuntimeRestart = "1".equals(SystemProperties.get("sys.boot_completed"));
        ... ...
    }

    private void run() {
        ... ...
            //启动系统服务
        try {
            traceBeginAndSlog("StartServices");
           // 系统引导服务启动，比如：AMS 启动
            startBootstrapServices();
            startCoreServices();
            startOtherServices();
			resetSerial();
            SystemServerInitThreadPool.shutdown();
        } catch (Throwable ex) {
            Slog.e("System", "******************************************");
            Slog.e("System", "************ Failure starting system services", ex);
            throw ex;
        } finally {
            traceEnd();
        }
        ... ...
        // Set up the Application instance for the system process and get started.
        traceBeginAndSlog("SetSystemProcess");
        //设置系统进程 详见 五
        mActivityManagerService.setSystemProcess();
        traceEnd();
        ... ...

    }

   //系统引导服务启动的方法
   private void startBootstrapServices() {
     
       ... ...
        // Activity manager runs the show.
        traceBeginAndSlog("StartActivityManager");
        //  通过 Lifecycle 启动 AMS 系统服务流程，详见 二
        mActivityManagerService = mSystemServiceManager.startService(
                ActivityManagerService.Lifecycle.class).getService();
        mActivityManagerService.setSystemServiceManager(mSystemServiceManager);
        mActivityManagerService.setInstaller(installer);
        traceEnd();
       ... ...
     }
    private void startOtherServices() {
        ... ...
		// 调用AMS 的 安装系统Provider installSystemProviders 方法，详见步骤 六
		traceBeginAndSlog("InstallSystemProviders");
		mActivityManagerService.installSystemProviders();
		// Now that SettingsProvider is ready, reactivate SQLiteCompatibilityWalFlags
		SQLiteCompatibilityWalFlags.reset();
		traceEnd();
		... ...
             // 系统准备完毕，可以让第三代码调用 详见 步骤七
        mActivityManagerService.systemReady(() -> {
            Slog.i(TAG, "Making services ready");
            traceBeginAndSlog("StartActivityManagerReadyPhase");
			// ActivityManager 准备完毕
            mSystemServiceManager.startBootPhase(
                    SystemService.PHASE_ACTIVITY_MANAGER_READY);
            traceEnd();
            ... ...
            }, BOOT_TIMINGS_TRACE_LOG);
    }
... ...
}
```

# 二、 通过 Lifecycle 启动初始化 AMS 系统服务

## 1. 静态全局 Lifecycle 类

在 AMS 中，我们看到静态全局的 final Lifecycle 类对外提供 getService()方法，方便供 `SystemServer`类调用。
调用方法如下：
```
        //  通过 Lifecycle 启动 AMS 系统服务流程，详见 二
        mActivityManagerService = mSystemServiceManager.startService(
                ActivityManagerService.Lifecycle.class).getService();
```
## 2. AMS 继承实现关系 

AMS 的继承关系如下：

```
public class ActivityManagerService extends IActivityManager.Stub
        implements Watchdog.Monitor, BatteryStatsImpl.BatteryCallback {
... ...
}
```

AMS 继承实现关系图如下：

![AMS 继承实现关系图](https://upload-images.jianshu.io/upload_images/5851256-4f9d3e5ab047e403.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上图我们可以看出
1.AMS 继承`IActivityManager.Stub`类
2.AMS 实现`Watchdog.Monitor`和`BatteryStatsImpl.BatteryCallback`接口。

## 3.通过 Lifecycle 启动初始化 AMS 系统服务实现

AMS 是通过Lifecycle启动 AMS 并实现初始化。

具体实现代码如下：

```
public class ActivityManagerService extends IActivityManager.Stub
        implements Watchdog.Monitor, BatteryStatsImpl.BatteryCallback {
 ... ...
 public static final class Lifecycle extends SystemService {
        private final ActivityManagerService mService;

        public Lifecycle(Context context) {
            super(context);
           //  new ActivityManagerService 实例 见分析三
            mService = new ActivityManagerService(context);
        }

        @Override
        public void onStart() {
           //  启动AMS ，见分析四 
            mService.start();
        }
       ... ... 
        public ActivityManagerService getService() {
            return mService;
        }
    }
... ...
}
```

# 三、AMS 构造函数实例实现

通过 ` mService = new ActivityManagerService(context); `，调用 AMS  构造函数，并运行在主线程中（main Thread），需要注意的是，AMS 中有多个Handler，请在loop 的时候注意，不要使用错了。

## 1. AMS 构造函数主要实现的功能
1. 创建 `ActivityManager` 、`android.ui`、`ActivityManager:procStart` 、`ActivityManager:kill `服务线程。
2. 设置前台广播、后台广播的超时时间。
3. 设置后台Service 最大个数(8个，低 RAM 1个)
4. 初始化 系统 Provider，以及AppErrors、AppWarnings。
5. 创建` /data/system `目录，并保存` procstats、appops.xml`等文件。
6. 创建 Intent 防火墙`IntentFirewall`，以及Activity 带来对象`ActivityStartController`。
7. 创建` CpuTracker `线程 ，收集` ANRS `，电池电量信息，更新CPU 状态等。
8. 获取 Watchdog 实例，并将AMS 添加到看门狗Monitor中，以及将mHandler 添加到看门狗 线程中。
9. 更新 oom_adj 状态。

## 2. AMS 构造函数实现方法

AMS 构造函数实现如下：

```
    public ActivityManagerService(Context systemContext) {
        ... ...

	    //创建名为"ActivityManager"的前台线程，并获取mHandler
	    // TAG 线程名，请看ActivityManagerDebugConfig，
	    //默认 TAG_AM   即为 ActivityManager
        mHandlerThread = new ServiceThread(TAG,
                THREAD_PRIORITY_FOREGROUND, false /*allowIo*/);
        mHandlerThread.start();
        mHandler = new MainHandler(mHandlerThread.getLooper());

		//通过UiThread类，创建名为"android.ui"的线程
        mUiHandler = mInjector.getUiHandler(this);

        //创建名为"ActivityManager:procStart"的前台线程，并获取mProcStartHandler
        mProcStartHandlerThread = new ServiceThread(TAG + ":procStart",
                THREAD_PRIORITY_FOREGROUND, false /* allowIo */);
        mProcStartHandlerThread.start();
        mProcStartHandler = new Handler(mProcStartHandlerThread.getLooper());
        //创建可以修改ActivityManager实例的对象
        mConstants = new ActivityManagerConstants(this, mHandler);

        //不重复创建 “ActivityManager:kill”后台线程，并获取sKillHandler
        if (sKillHandler == null) {
            sKillThread = new ServiceThread(TAG + ":kill",
                    THREAD_PRIORITY_BACKGROUND, true /* allowIo */);
            sKillThread.start();
            sKillHandler = new KillHandler(sKillThread.getLooper());
        }
        //前台广播接收器，在运行超过10s将放弃执行
        mFgBroadcastQueue = new BroadcastQueue(this, mHandler,
                "foreground", BROADCAST_FG_TIMEOUT, false);
		//后台广播接收器，在运行超过60s将放弃执行
        mBgBroadcastQueue = new BroadcastQueue(this, mHandler,
                "background", BROADCAST_BG_TIMEOUT, true);
        mBroadcastQueues[0] = mFgBroadcastQueue;
        mBroadcastQueues[1] = mBgBroadcastQueue;
		//创建ActiveServices，其中非低内存手机后台服务最大为8个
		//        mMaxStartingBackground = maxBg > 0
        //        ? maxBg : ActivityManager.isLowRamDeviceStatic() ? 1 : 8;
        mServices = new ActiveServices(this);
		//创建 系统 Provider
        mProviderMap = new ProviderMap(this);
		// 创建 Apperror 对象
        mAppErrors = new AppErrors(mUiContext, this);
        //创建目录/data/system
        File dataDir = Environment.getDataDirectory();
        File systemDir = new File(dataDir, "system");
        systemDir.mkdirs();
        // 创建 管理 app Warning 的Dialog 
        mAppWarnings = new AppWarnings(this, mUiContext, mHandler, mUiHandler, systemDir);

        //创建服务BatteryStatsService
        mBatteryStatsService = new BatteryStatsService(systemContext, systemDir, mHandler);
        mBatteryStatsService.getActiveStatistics().readLocked();
        mBatteryStatsService.scheduleWriteToDisk();
        mOnBattery = DEBUG_POWER ? true
                : mBatteryStatsService.getActiveStatistics().getIsOnBattery();
        mBatteryStatsService.getActiveStatistics().setCallback(this);
        //创建进程统计服务，信息保存在目录/data/system/procstats
        mProcessStats = new ProcessStatsService(this, new File(systemDir, "procstats"));

        mAppOpsService = mInjector.getAppOpsService(new File(systemDir, "appops.xml"), mHandler);

        mGrantFile = new AtomicFile(new File(systemDir, "urigrants.xml"), "uri-grants");
        //创建多用户、VR controller 
        mUserController = new UserController(this);
        mVrController = new VrController(this);
        ... ...
		// 创建 Intent 防火墙
        mIntentFirewall = new IntentFirewall(new IntentFirewallInterface(), mHandler);
        mTaskChangeNotificationController =
                new TaskChangeNotificationController(this, mStackSupervisor, mHandler);
		// 创建控制 Activity 启动代理对象
        mActivityStartController = new ActivityStartController(this);
		// 创建最近任务列表，并保存在最近任务列表栈中
        mRecentTasks = createRecentTasks();
        mStackSupervisor.setRecentTasks(mRecentTasks);
		// 创建 任务栈锁，比如在Screen pinning Mode 下
        mLockTaskController = new LockTaskController(mContext, mStackSupervisor, mHandler);
        mLifecycleManager = new ClientLifecycleManager();
        //创建名为"CpuTracker"的线程， 主要用于 收集 ANRS、电池状态信息等
        mProcessCpuThread = new Thread("CpuTracker") {
            @Override
            public void run() {
                synchronized (mProcessCpuTracker) {
                    mProcessCpuInitLatch.countDown();
                    mProcessCpuTracker.init();
                }
              ... ...
						// 更新CPU 状态
                        updateCpuStatsNow();
              ... ...
            }
        };

        mHiddenApiBlacklist = new HiddenApiSettings(mHandler, mContext);
        // 获取 Watchdog看门狗实例 ，并添加到Monitor监控 以及mHandler 添加到Thread中
        Watchdog.getInstance().addMonitor(this);
        Watchdog.getInstance().addThread(mHandler);

		// 更新 oom_adj 
        updateOomAdjLocked();
        ... ...
    }

```
## 3.后台Service 最大限制数设置
```
class ActivityManagerDebugConfig {
    ... ...
    static final boolean TAG_WITH_CLASS_NAME = false;

    // Default log tag for the activity manager package.
    static final String TAG_AM = "ActivityManager";
    ... ...
}
```
# 四、启动AMS.Start() 实现

## 1.AMS.Start() 主要功能

1.  移除所有的进程组
2.  启动CpuTracker线程
3.  启动 app 操作服务AppOpsService
4. 将ActivityManagerInternal 添加到本地Service中

## 2.AMS.Start() 功能 实现

AMS.Start() 功能实现如下：

```
    private void start() {
        //移除所有的进程组
        removeAllProcessGroups();
		//启动CpuTracker线程
        mProcessCpuThread.start();
        //启动电池统计服务
        mBatteryStatsService.publish();
		// 启动 app 操作Service
        mAppOpsService.publish(mContext);
        Slog.d("AppOps", "AppOpsService published");
		// 将ActivityManagerInternal 添加到本地Service中
        LocalServices.addService(ActivityManagerInternal.class, new LocalService());
    }
```
# 五、AMS设置系统进程实现

setSystemProcess 主要作用是 
添加 各种服务 包括`meminfo、gfxinfo、dbinfo、cpuinfo`以及`permission`和`processinfo`系统Service。同时， 更新进程相关的 Lru 算法 ，以及oom_adj 值。
实现代码如下：
```
    public void setSystemProcess() {
        try {
			//注册添加各种Service，可以使用adb shell dumpsys 
            ServiceManager.addService(Context.ACTIVITY_SERVICE, this, /* allowIsolated= */ true,
                    DUMP_FLAG_PRIORITY_CRITICAL | DUMP_FLAG_PRIORITY_NORMAL | DUMP_FLAG_PROTO);
            ServiceManager.addService(ProcessStats.SERVICE_NAME, mProcessStats);
			//adb shell dumpsys meminfo 注册内存信息Service
            ServiceManager.addService("meminfo", new MemBinder(this), /* allowIsolated= */ false,
                    DUMP_FLAG_PRIORITY_HIGH);
			//adb shell dumpsys gfxinfo 注册GraphicsBinder
            ServiceManager.addService("gfxinfo", new GraphicsBinder(this));
			//adb shell dumpsys dbinfo 注册 DbBinder 
            ServiceManager.addService("dbinfo", new DbBinder(this));
            if (MONITOR_CPU_USAGE) {
			//adb shell dumpsys cpuinfo	注册CPU 信息Service
                ServiceManager.addService("cpuinfo", new CpuBinder(this),
                        /* allowIsolated= */ false, DUMP_FLAG_PRIORITY_CRITICAL);
            }
			//adb shell dumpsys packages permissions 注册 系统权限 信息Service
            ServiceManager.addService("permission", new PermissionController(this));
            ServiceManager.addService("processinfo", new ProcessInfoService(this));
            // 获取包名为 android 的应用信息，framework-res.apk
            ApplicationInfo info = mContext.getPackageManager().getApplicationInfo(
                    "android", STOCK_PM_FLAGS | MATCH_SYSTEM_ONLY);
            mSystemThread.installSystemApplicationInfo(info, getClass().getClassLoader());

            synchronized (this) {
                ProcessRecord app = newProcessRecordLocked(info, info.processName, false, 0);
                app.persistent = true;
                app.pid = MY_PID;
                app.maxAdj = ProcessList.SYSTEM_ADJ;
                app.makeActive(mSystemThread.getApplicationThread(), mProcessStats);
                synchronized (mPidsSelfLocked) {
                    mPidsSelfLocked.put(app.pid, app);
                }
				//更新  进程 Lru 算法 ，以及oom_adj 值
                updateLruProcessLocked(app, false, null);
                updateOomAdjLocked();
            }
          } 

        //  当 packager manager 启动并运行时开始监听 app 操作
        mAppOpsService.startWatchingMode(AppOpsManager.OP_RUN_IN_BACKGROUND, null,
                new IAppOpsCallback.Stub() {
                    @Override public void opChanged(int op, int uid, String packageName) {
                        if (op == AppOpsManager.OP_RUN_IN_BACKGROUND && packageName != null) {
                            if (mAppOpsService.checkOperation(op, uid, packageName)
                                    != AppOpsManager.MODE_ALLOWED) {
                                runInBackgroundDisabled(uid);
                            }
                        }
                    }
                });
    }
```

# 六、安装系统 Provider 

通过 SystemServer.java 类中的`startOtherServices() `方法`mActivityManagerService.installSystemProviders();`调用 AMS中的`installSystemProviders`方法。
下面我们看看installSystemProviders 方法的主要功能

```
   public final void installSystemProviders() {
        List<ProviderInfo> providers;
        synchronized (this) {
            ProcessRecord app = mProcessNames.get("system", SYSTEM_UID);
            providers = generateApplicationProvidersLocked(app);
            if (providers != null) {
                for (int i=providers.size()-1; i>=0; i--) {
                    ProviderInfo pi = (ProviderInfo)providers.get(i);
					// 1&1=1 1&0=0 0&0=0
                    if ((pi.applicationInfo.flags&ApplicationInfo.FLAG_SYSTEM) == 0) {
                        Slog.w(TAG, "Not installing system proc provider " + pi.name
                                + ": not system .apk");
						//移除非系统 app
                        providers.remove(i);
                    }
                }
            }
        }
        ... ...
        mConstants.start(mContext.getContentResolver());
		// 创建 CoreSettingsObserver ，监控核心设置的变化
        mCoreSettingsObserver = new CoreSettingsObserver(this);
		// 创建 FontScaleSettingObserver，监控字体的变化
        mFontScaleSettingObserver = new FontScaleSettingObserver();
		// 创建 DevelopmentSettingsObserver 监控开发者选项
        mDevelopmentSettingsObserver = new DevelopmentSettingsObserver();
        GlobalSettingsToPropertiesMapper.start(mContext.getContentResolver());

        //对外公布 settings provider
        RescueParty.onSettingsProviderPublished(mContext);

        //mUsageStatsService.monitorPackages();
    }
```
# 七、AMS.systemReady准备完成

SystemServer 中的 AMS.systemReady 主要完成以下功能
1. 确保系统Service已经准备完成。
2. ActivityManager 引导启动完成。
3. 开始监听NativeCrash。
4. WebView 准备完毕，方便三方apk 调用。
5. 启动车载相关的服务。
6. 启动SystemUI。
7. 确保 MakeNetworkManagementService 准备完成。
8.  启动 Watchdog 看门狗程序。
9. 等待所有的app数据预加载，然后，可以启动三方app。
10.  Location、telephony、输入法、Media、MMS、Daemon等相关的Service已经运行并准备好

SystemServer中具体实现代码情况下文：

```
public final class SystemServer {
... ...

    /**
     * The main entry point from zygote.
     */
    public static void main(String[] args) {
        new SystemServer().run();
    }

    public SystemServer() {
        ... ...
    }

    private void run() {
        ... ...
            //启动系统服务 
            startOtherServices();			 
        ... ...

    }

    private void startOtherServices() {
		... ...
        // 系统准备完毕，可以让第三代码调用
        mActivityManagerService.systemReady(() -> {
            Slog.i(TAG, "Making services ready");
            traceBeginAndSlog("StartActivityManagerReadyPhase");
			// ActivityManager 准备完毕
            mSystemServiceManager.startBootPhase(
                    SystemService.PHASE_ACTIVITY_MANAGER_READY);
            traceEnd();
            traceBeginAndSlog("StartObservingNativeCrashes");
            try {
			// 开始监听 Native Crash
                mActivityManagerService.startObservingNativeCrashes();
            } catch (Throwable e) {
                reportWtf("observing native crashes", e);
            }
            traceEnd();

            //WebView 准备好，方便三方apk 调用
            final String WEBVIEW_PREPARATION = "WebViewFactoryPreparation";
            Future<?> webviewPrep = null;
            if (!mOnlyCore && mWebViewUpdateService != null) {
                webviewPrep = SystemServerInitThreadPool.get().submit(() -> {
                    Slog.i(TAG, WEBVIEW_PREPARATION);
                    TimingsTraceLog traceLog = new TimingsTraceLog(
                            SYSTEM_SERVER_TIMING_ASYNC_TAG, Trace.TRACE_TAG_SYSTEM_SERVER);
                    traceLog.traceBegin(WEBVIEW_PREPARATION);
                    ConcurrentUtils.waitForFutureNoInterrupt(mZygotePreload, "Zygote preload");
                    mZygotePreload = null;
                    mWebViewUpdateService.prepareWebViewInSystemServer();
                    traceLog.traceEnd();
                }, WEBVIEW_PREPARATION);
            }
            // 启动车载相关的服务
            if (mPackageManager.hasSystemFeature(PackageManager.FEATURE_AUTOMOTIVE)) {
                traceBeginAndSlog("StartCarServiceHelperService");
                mSystemServiceManager.startService(CAR_SERVICE_HELPER_SERVICE_CLASS);
                traceEnd();
            }
            // 启动SystemUI 
            traceBeginAndSlog("StartSystemUI");
            try {
                startSystemUi(context, windowManagerF);
            } catch (Throwable e) {
                reportWtf("starting System UI", e);
            }
            traceEnd();
			// MakeNetworkManagementService 准备完成
            traceBeginAndSlog("MakeNetworkManagementServiceReady");
            try {
                if (networkManagementF != null) networkManagementF.systemReady();
            } catch (Throwable e) {
                reportWtf("making Network Managment Service ready", e);
            }
            CountDownLatch networkPolicyInitReadySignal = null;
            if (networkPolicyF != null) {
                networkPolicyInitReadySignal = networkPolicyF
                        .networkScoreAndNetworkManagementServiceReady();
            }
            traceEnd();
            traceBeginAndSlog("MakeIpSecServiceReady");
            try {
                if (ipSecServiceF != null) ipSecServiceF.systemReady();
            } catch (Throwable e) {
                reportWtf("making IpSec Service ready", e);
            }
            traceEnd();
            traceBeginAndSlog("MakeNetworkStatsServiceReady");
            try {
                if (networkStatsF != null) networkStatsF.systemReady();
            } catch (Throwable e) {
                reportWtf("making Network Stats Service ready", e);
            }
            traceEnd();
            traceBeginAndSlog("MakeConnectivityServiceReady");
            try {
                if (connectivityF != null) connectivityF.systemReady();
            } catch (Throwable e) {
                reportWtf("making Connectivity Service ready", e);
            }
            traceEnd();
            traceBeginAndSlog("MakeNetworkPolicyServiceReady");
            try {
                if (networkPolicyF != null) {
                    networkPolicyF.systemReady(networkPolicyInitReadySignal);
                }
            } catch (Throwable e) {
                reportWtf("making Network Policy Service ready", e);
            }
            traceEnd();
            // 启动 Watchdog 看门狗程序 
            traceBeginAndSlog("StartWatchdog");
            Watchdog.getInstance().start();
            traceEnd();

            //等待所有的app数据预加载
            mPackageManagerService.waitForAppDataPrepared();

            // It is now okay to let the various system services start their
            // third party code...
            traceBeginAndSlog("PhaseThirdPartyAppsCanStart");
            // confirm webview completion before starting 3rd party
            if (webviewPrep != null) {
                ConcurrentUtils.waitForFutureNoInterrupt(webviewPrep, WEBVIEW_PREPARATION);
            }
			// 三方 app 已准备好，可以启动 
            mSystemServiceManager.startBootPhase(
                    SystemService.PHASE_THIRD_PARTY_APPS_CAN_START);
            traceEnd();

            traceBeginAndSlog("MakeLocationServiceReady");
            try {
			// 定位服务已经运行，并准备好
                if (locationF != null) locationF.systemRunning();
            } catch (Throwable e) {
                reportWtf("Notifying Location Service running", e);
            }
            traceEnd();
            traceBeginAndSlog("MakeCountryDetectionServiceReady");
            try {
                if (countryDetectorF != null) countryDetectorF.systemRunning();
            } catch (Throwable e) {
                reportWtf("Notifying CountryDetectorService running", e);
            }
            traceEnd();
            traceBeginAndSlog("MakeNetworkTimeUpdateReady");
            try {
                if (networkTimeUpdaterF != null) networkTimeUpdaterF.systemRunning();
            } catch (Throwable e) {
                reportWtf("Notifying NetworkTimeService running", e);
            }
            traceEnd();
            traceBeginAndSlog("MakeCommonTimeManagementServiceReady");
            try {
                if (commonTimeMgmtServiceF != null) {
                    commonTimeMgmtServiceF.systemRunning();
                }
            } catch (Throwable e) {
                reportWtf("Notifying CommonTimeManagementService running", e);
            }
            traceEnd();
            traceBeginAndSlog("MakeInputManagerServiceReady");
            try {
			// 输入法相关的Service已经运行并准备好
                // TODO(BT) Pass parameter to input manager
                if (inputManagerF != null) inputManagerF.systemRunning();
            } catch (Throwable e) {
                reportWtf("Notifying InputManagerService running", e);
            }
            traceEnd();
            traceBeginAndSlog("MakeTelephonyRegistryReady");
            try {
		    // telephony相关的Service已经运行并准备好
                if (telephonyRegistryF != null) telephonyRegistryF.systemRunning();
            } catch (Throwable e) {
                reportWtf("Notifying TelephonyRegistry running", e);
            }
            traceEnd();
            traceBeginAndSlog("MakeMediaRouterServiceReady");
            try {
			// media 相关的Service已经运行并准备好	
                if (mediaRouterF != null) mediaRouterF.systemRunning();
            } catch (Throwable e) {
                reportWtf("Notifying MediaRouterService running", e);
            }
            traceEnd();
            traceBeginAndSlog("MakeMmsServiceReady");
            try {
			// 短信 相关的Service已经运行并准备好	
                if (mmsServiceF != null) mmsServiceF.systemRunning();
            } catch (Throwable e) {
                reportWtf("Notifying MmsService running", e);
            }
            traceEnd();

            traceBeginAndSlog("IncidentDaemonReady");
            try {
                // TODO: Switch from checkService to getService once it's always
                // in the build and should reliably be there.
                final IIncidentManager incident = IIncidentManager.Stub.asInterface(
                        ServiceManager.getService(Context.INCIDENT_SERVICE));
                if (incident != null) incident.systemRunning();
            } catch (Throwable e) {
                reportWtf("Notifying incident daemon running", e);
            }
            traceEnd();
        }, BOOT_TIMINGS_TRACE_LOG);
    }
... ...
}

```

AMS 中 SystemReady 主要作用有以下功能
1. kill 掉非persistent app进程。
2. 检测 Setting中的一些配置索引。
3. 注册低电量模式的关闭。
4. 启动 persistent app。
5. 启动Home Activity 比如Launcher
6. 恢复显示TopActivity

具体实现代码如下：
```
   public void systemReady(final Runnable goingCallback, TimingsTraceLog traceLog) {
        traceLog.traceBegin("PhaseActivityManagerReady");
        synchronized(this) {
            if (mSystemReady) {
                // 如果我们完成所有的receiver 调用，然后需要调用 SystemServer 的boot phase 方法
                if (goingCallback != null) {
                    goingCallback.run();
                }
                return;
            }
			... ...
			// 确保 VR 、多用户控制、最近任务列表、最近app操作已完成
            mVrController.onSystemReady();
            mUserController.onSystemReady();
            mRecentTasks.onSystemReadyLocked();
            mAppOpsService.systemReady();
            mSystemReady = true;
        }
        ... ...
        // 非 persistent 应用都会添加到kill list中
        ArrayList<ProcessRecord> procsToKill = null;
        synchronized(mPidsSelfLocked) {
            for (int i=mPidsSelfLocked.size()-1; i>=0; i--) {
                ProcessRecord proc = mPidsSelfLocked.valueAt(i);
				// 判断是否是persistent 应用
                if (!isAllowedWhileBooting(proc.info)){
                    if (procsToKill == null) {
                        procsToKill = new ArrayList<ProcessRecord>();
                    }
                    procsToKill.add(proc);
                }
            }
        }

        synchronized(this) {
            if (procsToKill != null) {
                for (int i=procsToKill.size()-1; i>=0; i--) {
                    ProcessRecord proc = procsToKill.get(i);
                    //杀掉非persistent 应用
                    removeProcessLocked(proc, true, false, "system update done");
                }
            }

            // Now that we have cleaned up any update processes, we
            // are ready to start launching real processes and know that
            // we won't trample on them any more.
            mProcessesReady = true;
        }

        Slog.i(TAG, "System now ready");
        EventLog.writeEvent(EventLogTags.BOOT_PROGRESS_AMS_READY,
            SystemClock.uptimeMillis());
         
        synchronized(this) {
		... ...
		// 检索Settings的一些信息
		// 比如多用户、画中画、分屏、fullScreen等
        retrieveSettings();
        final int currentUserId = mUserController.getCurrentUserId();
        synchronized (this) {
		// 读取授权的URI 权限文件	
            readGrantedUriPermissionsLocked();
        }

        final PowerManagerInternal pmi = LocalServices.getService(PowerManagerInternal.class);
        if (pmi != null) {
		// 注册并更新低电量模式广播	
            pmi.registerLowPowerModeObserver(ServiceType.FORCE_BACKGROUND_CHECK,
                    state -> updateForceBackgroundCheck(state.batterySaverEnabled));
            updateForceBackgroundCheck(
                    pmi.getLowPowerState(ServiceType.FORCE_BACKGROUND_CHECK).batterySaverEnabled);
        } else {
            Slog.wtf(TAG, "PowerManagerInternal not found.");
        }
        ... ...

        synchronized (this) {
            // 启动 persistent app
            startPersistentApps(PackageManager.MATCH_DIRECT_BOOT_AWARE);

            // Start up initial activity.
            mBooting = true;
            // Enable home activity for system user, so that the system can always boot. We don't
            // do this when the system user is not setup since the setup wizard should be the one
            // to handle home activity in this case.
            if (UserManager.isSplitSystemUser() &&
                    Settings.Secure.getInt(mContext.getContentResolver(),
                         Settings.Secure.USER_SETUP_COMPLETE, 0) != 0) {
                ComponentName cName = new ComponentName(mContext, SystemUserHomeActivity.class);
                try {
                    AppGlobals.getPackageManager().setComponentEnabledSetting(cName,
                            PackageManager.COMPONENT_ENABLED_STATE_ENABLED, 0,
                            UserHandle.USER_SYSTEM);
                } catch (RemoteException e) {
                    throw e.rethrowAsRuntimeException();
                }
            }
			// 启动 Home Activity ，比如 通过 StartHomeActivity(inten,ainfo,myReason) 启动Launcher 
            // 启动 Launcher方法请见 步骤八 			 
            startHomeActivityLocked(currentUserId, "systemReady");
            long ident = Binder.clearCallingIdentity();
            try {
				// 发送 ACTION_USER_STARTED 的广播
                Intent intent = new Intent(Intent.ACTION_USER_STARTED);
                intent.addFlags(Intent.FLAG_RECEIVER_REGISTERED_ONLY
                        | Intent.FLAG_RECEIVER_FOREGROUND);
                intent.putExtra(Intent.EXTRA_USER_HANDLE, currentUserId);
                broadcastIntentLocked(null, null, intent,
                        null, null, 0, null, null, null, OP_NONE,
                        null, false, false, MY_PID, SYSTEM_UID,
                        currentUserId);
				// 发送 ACTION_USER_STARTING 的广播
                intent = new Intent(Intent.ACTION_USER_STARTING);
                intent.addFlags(Intent.FLAG_RECEIVER_REGISTERED_ONLY);
                intent.putExtra(Intent.EXTRA_USER_HANDLE, currentUserId);
                broadcastIntentLocked(null, null, intent,
                        null, new IIntentReceiver.Stub() {
                            @Override
                            public void performReceive(Intent intent, int resultCode, String data,
                                    Bundle extras, boolean ordered, boolean sticky, int sendingUser)
                                    throws RemoteException {
                            }
                        }, 0, null, null,
                        new String[] {INTERACT_ACROSS_USERS}, OP_NONE,
                        null, true, false, MY_PID, SYSTEM_UID, UserHandle.USER_ALL);
            } catch (Throwable t) {
                Slog.wtf(TAG, "Failed sending first user broadcasts", t);
            } finally {
                Binder.restoreCallingIdentity(ident);
            }
			// 恢复显示Top Activity
            mStackSupervisor.resumeFocusedStackTopActivityLocked();
            mUserController.sendUserSwitchBroadcasts(-1, currentUserId);
            ... ...
            traceLog.traceEnd(); // ActivityManagerStartApps
            traceLog.traceEnd(); // PhaseActivityManagerReady
        }
    }

```
# 八、 AMS 启动 Launcher实现

startHomeActivityLocked 主要用来启动Home Launcher，具体实现代码如下:

```
boolean startHomeActivityLocked(int userId, String reason) {
        if (mFactoryTest == FactoryTest.FACTORY_TEST_LOW_LEVEL
                && mTopAction == null) {
            // We are running in factory test mode, but unable to find
            // the factory test app, so just sit around displaying the
            // error message and don't try to start anything.
            return false;
        }
		// 获取 Intent.CATEGORY_HOME intent 
        Intent intent = getHomeIntent();
        ActivityInfo aInfo = resolveActivityInfo(intent, STOCK_PM_FLAGS, userId);
        if (aInfo != null) {
            intent.setComponent(new ComponentName(aInfo.applicationInfo.packageName, aInfo.name));
            // Don't do this if the home app is currently being
            // instrumented.
            aInfo = new ActivityInfo(aInfo);
            aInfo.applicationInfo = getAppInfoForUser(aInfo.applicationInfo, userId);
            ProcessRecord app = getProcessRecordLocked(aInfo.processName,
                    aInfo.applicationInfo.uid, true);
            if (app == null || app.instr == null) {
                intent.setFlags(intent.getFlags() | FLAG_ACTIVITY_NEW_TASK);
                final int resolvedUserId = UserHandle.getUserId(aInfo.applicationInfo.uid);
                // For ANR debugging to verify if the user activity is the one that actually
                // launched.
                final String myReason = reason + ":" + userId + ":" + resolvedUserId;
				// 启动Home Activity
                mActivityStartController.startHomeActivity(intent, aInfo, myReason);
            }
        } else {
            Slog.wtf(TAG, "No home screen found for " + intent, new Throwable());
        }

        return true;
    }
```


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

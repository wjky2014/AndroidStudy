##### 和你一起终身学习，这是是程序员Android


本篇文章主要介绍 `Android` 开发中 **SystemServer进程启动** 部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、SystemServer 启动的服务有哪些
>二、SystemServer启动总体流程概述
>三、SystemServer 如何启动，是谁启动的？
>四、 SystemServer 启动入门 main 方法
>五、SystemServer Run 方法初始与启动
>六、SystemServer 的引导服务有哪些
>七、SystemServer 的核心服务有哪些
>八、SystemServer 的其他服务有哪些



# 一、SystemServer 启动的服务有哪些

`SystemServer`  主要启动 `ActivityManagerService`、`PackageManagerService`、`WindowManagerService`、`LightsService`、`LightsService`、`BatteryService`、`TelephonyRegistry`、`RecoverySystemService` 等等,主要分三大类，后文会详细列举。

## 1.SystemServer 启动的服务  

![SystemServer  启动的服务](https://upload-images.jianshu.io/upload_images/5851256-9b2a4b1276cab201.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![SystemServer  启动的服务](https://upload-images.jianshu.io/upload_images/5851256-d4beff56a8836071.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#二、SystemServer启动总体流程概述
## 1.SystemServer 代码 
`\alps\frameworks\base\services\java\com\android\server\SystemServer.java`

![SystemServer 进程启动导图](https://upload-images.jianshu.io/upload_images/5851256-33fde45dbf464601.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#三、SystemServer  如何启动，是谁启动的？

`SystemServer `是通过`Zygote` 启动的，在`ZygoteInit.java`类的`（frameworks\base\core\java\com\android\internal\os\ZygoteInit.java） ` `main `方法中通过`forkSystemServer `启动。
```

    public static void main(String argv[]) {
        ....
            //设置变量区分是否启动SystemServer
            boolean startSystemServer = false;
            String socketName = "zygote";
            String abiList = null;
            boolean enableLazyPreload = false;
            for (int i = 1; i < argv.length; i++) {
                if ("start-system-server".equals(argv[i])) {
                   // 需要启动时候，将标志位设置为true
                    startSystemServer = true;
                } else if ("--enable-lazy-preload".equals(argv[i])) {
                    enableLazyPreload = true;
                } 
            ... ...

            if (startSystemServer) {
                // 通过 Zygote  fork 出 SystemServer 
                Runnable r = forkSystemServer(abiList, socketName, zygoteServer);

                // {@code r == null} in the parent (zygote) process, and {@code r != null} in the
                // child (system_server) process.
                if (r != null) {
                    r.run();
                    return;
                }
            }
```

#四、 SystemServer 启动入门 main 方法

`main` 入口 通过  `new SystemServer().run();`开启`SystemServer`启动。

`main` 入口代码如下：
```
    /**
     * The main entry point from zygote.
     */
    public static void main(String[] args) {
        new SystemServer().run();
    }
```

通过`Main `入口,调用`SystemServer` 构造方法。

```
    public SystemServer() {
        // 检查工程模式.
        mFactoryTestMode = FactoryTest.getMode();
        // 判断是否是重启
        mRuntimeRestart = "1".equals(SystemProperties.get("sys.boot_completed"));

        mRuntimeStartElapsedTime = SystemClock.elapsedRealtime();
        mRuntimeStartUptime = SystemClock.uptimeMillis();
    }

    private void run() {
       ... ...
    }
```
# 五、SystemServer Run 方法初始与启动

##1.SystemServer Run 方法  

```
    private void run() {
        try {
            traceBeginAndSlog("InitBeforeStartServices");
			//初始化时间
            if (System.currentTimeMillis() < EARLIEST_SUPPORTED_TIME) {
                Slog.w(TAG, "System clock is before 1970; setting to 1970.");
                SystemClock.setCurrentTimeMillis(EARLIEST_SUPPORTED_TIME);
            }

            //设置默认时区
            String timezoneProperty =  SystemProperties.get("persist.sys.timezone");
            if (timezoneProperty == null || timezoneProperty.isEmpty()) {
                Slog.w(TAG, "Timezone not set; setting to GMT.");
                SystemProperties.set("persist.sys.timezone", "GMT");
            }
            ... ... 

            // 开始进入Android SystemServer
            Slog.i(TAG, "Entered the Android system server!");
            int uptimeMillis = (int) SystemClock.elapsedRealtime();
            EventLog.writeEvent(EventLogTags.BOOT_PROGRESS_SYSTEM_RUN, uptimeMillis);
            if (!mRuntimeRestart) {
                MetricsLogger.histogram(null, "boot_system_server_init", uptimeMillis);
            }
            ... ...

            //如果支持指纹，需初始化指纹ro.build.fingerprint
            Build.ensureFingerprintProperty();

            ... ...
				
            // 初始化 native services.
            System.loadLibrary("android_servers");

            // 检查最近一次关机是否失败
            performPendingShutdown();

            // 初始化 the system context.
            createSystemContext();

            // 创建 system service manager.
            mSystemServiceManager = new SystemServiceManager(mSystemContext);
            mSystemServiceManager.setStartInfo(mRuntimeRestart,
                    mRuntimeStartElapsedTime, mRuntimeStartUptime);
            LocalServices.addService(SystemServiceManager.class, mSystemServiceManager);
            // 初始化SystemServer 线程池
            SystemServerInitThreadPool.get();
        } finally {
            traceEnd();  // InitBeforeStartServices
        }
		 ... ...
		 
        // 开始启动 services.
        try {
            traceBeginAndSlog("StartServices");
	    // 1. 启动引导服务  详见分析六
            startBootstrapServices();
            // 2. 启动核心服务  详见分析七
            startCoreServices();
            // 3.启动其他服务  详见分析八
            startOtherServices();
            SystemServerInitThreadPool.shutdown();
        } catch (Throwable ex) {
            Slog.e("System", "******************************************");
            Slog.e("System", "************ Failure starting system services", ex);
            throw ex;
        } finally {
            traceEnd();
        }

        StrictMode.initVmDefaults(null);


    }
```

# 六、SystemServer 的引导服务有哪些

` SystemServer `启动的常用引导服务有` installed 、DeviceIdentifiersPolicyService、 ActivityManagerService.、PowerManagerService 、 RecoverySystemService 、 LightsService 、 PackageManagerService、UserManagerService、OverlayManagerService`等。
 


![启动的引导服务的大致流程](https://upload-images.jianshu.io/upload_images/5851256-47888f9eab97105d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##1.startBootstrapServices代码 
```

    /**
     * Starts the small tangle of critical services that are needed to get
     * the system off the ground.  These services have complex mutual dependencies
     * which is why we initialize them all in one place here.  Unless your service
     * is also entwined in these dependencies, it should be initialized in one of
     * the other functions.
     */
    private void startBootstrapServices() {
        Slog.i(TAG, "Reading configuration...");
        final String TAG_SYSTEM_CONFIG = "ReadingSystemConfig";
        traceBeginAndSlog(TAG_SYSTEM_CONFIG);
        SystemServerInitThreadPool.get().submit(SystemConfig::getInstance, TAG_SYSTEM_CONFIG);
        traceEnd();

        // 启动 installed
        traceBeginAndSlog("StartInstaller");
        Installer installer = mSystemServiceManager.startService(Installer.class);
        traceEnd();

        //启动 设备标识符 服务
        traceBeginAndSlog("DeviceIdentifiersPolicyService");
        mSystemServiceManager.startService(DeviceIdentifiersPolicyService.class);
        traceEnd();

        // 启动 AMS.
        traceBeginAndSlog("StartActivityManager");
        mActivityManagerService = mSystemServiceManager.startService(
                ActivityManagerService.Lifecycle.class).getService();
        mActivityManagerService.setSystemServiceManager(mSystemServiceManager);
        mActivityManagerService.setInstaller(installer);
        traceEnd();

        //启动 PMS 
        traceBeginAndSlog("StartPowerManager");
        mPowerManagerService = mSystemServiceManager.startService(PowerManagerService.class);
        traceEnd();

        //初始化电源管理功能
        traceBeginAndSlog("InitPowerManagement");
        mActivityManagerService.initPowerManagement();
        traceEnd();

        // 启动 RecoverySystemService 
        traceBeginAndSlog("StartRecoverySystemService");
        mSystemServiceManager.startService(RecoverySystemService.class);
        traceEnd();

        // 为启动事件添加记录
        RescueParty.noteBoot(mSystemContext);

        // 启动 LightsService 管理LEDs 和背光显示
        traceBeginAndSlog("StartLightsService");
        mSystemServiceManager.startService(LightsService.class);
        traceEnd();

        traceBeginAndSlog("StartSidekickService");
        // Package manager isn't started yet; need to use SysProp not hardware feature
        if (SystemProperties.getBoolean("config.enable_sidekick_graphics", false)) {
            mSystemServiceManager.startService(WEAR_SIDEKICK_SERVICE_CLASS);
        }
        traceEnd();

        // Display manager is needed to provide display metrics before package manager
        // starts up.
        traceBeginAndSlog("StartDisplayManager");
        mDisplayManagerService = mSystemServiceManager.startService(LightsService.class);
        traceEnd();

        // We need the default display before we can initialize the package manager.
        traceBeginAndSlog("WaitForDisplay");
        mSystemServiceManager.startBootPhase(SystemService.PHASE_WAIT_FOR_DEFAULT_DISPLAY);
        traceEnd();

        // Only run "core" apps if we're encrypting the device.
        String cryptState = SystemProperties.get("vold.decrypt");
        if (ENCRYPTING_STATE.equals(cryptState)) {
            Slog.w(TAG, "Detected encryption in progress - only parsing core apps");
            mOnlyCore = true;
        } else if (ENCRYPTED_STATE.equals(cryptState)) {
            Slog.w(TAG, "Device encrypted - only parsing core apps");
            mOnlyCore = true;
        }

        // 启动 PackageManagerService
        if (!mRuntimeRestart) {
            MetricsLogger.histogram(null, "boot_package_manager_init_start",
                    (int) SystemClock.elapsedRealtime());
        }
        traceBeginAndSlog("StartPackageManagerService");
        mPackageManagerService = PackageManagerService.main(mSystemContext, installer,
                mFactoryTestMode != FactoryTest.FACTORY_TEST_OFF, mOnlyCore);
        mFirstBoot = mPackageManagerService.isFirstBoot();
        mPackageManager = mSystemContext.getPackageManager();
        traceEnd();
        if (!mRuntimeRestart && !isFirstBootOrUpgrade()) {
            MetricsLogger.histogram(null, "boot_package_manager_init_ready",
                    (int) SystemClock.elapsedRealtime());
        }
        // Manages A/B OTA dexopting. This is a bootstrap service as we need it to rename
        // A/B artifacts after boot, before anything else might touch/need them.
        // Note: this isn't needed during decryption (we don't have /data anyways).
        if (!mOnlyCore) {
            boolean disableOtaDexopt = SystemProperties.getBoolean("config.disable_otadexopt",
                    false);
            if (!disableOtaDexopt) {
                traceBeginAndSlog("StartOtaDexOptService");
                try {
                    OtaDexoptService.main(mSystemContext, mPackageManagerService);
                } catch (Throwable e) {
                    reportWtf("starting OtaDexOptService", e);
                } finally {
                    traceEnd();
                }
            }
        }
        //启动多用户    UserManagerService
        traceBeginAndSlog("StartUserManagerService");
        mSystemServiceManager.startService(UserManagerService.LifeCycle.class);
        traceEnd();

        // 初始化属性缓存
        traceBeginAndSlog("InitAttributerCache");
        AttributeCache.init(mSystemContext);
        traceEnd();

        // Set up the Application instance for the system process and get started.
        traceBeginAndSlog("SetSystemProcess");
        mActivityManagerService.setSystemProcess();
        traceEnd();

        // DisplayManagerService needs to setup android.display scheduling related policies
        // since setSystemProcess() would have overridden policies due to setProcessGroup
        mDisplayManagerService.setupSchedulerPolicies();

        /// M: CTA requirement - permission control  @{
        /*
         * M: MOTA for CTA permissions handling
         * This function is used for granting CTA permissions after OTA upgrade.
         * This should be placed after AMS is added to ServiceManager and before
         * starting other services since granting permissions needs AMS instance
         * to do permission checking.
         */
        mPackageManagerService.onAmsAddedtoServiceMgr();
        /// @}

        //  启动 OverlayManagerService
        traceBeginAndSlog("StartOverlayManagerService");
        mSystemServiceManager.startService(new OverlayManagerService(mSystemContext, installer));
        traceEnd();

        // The sensor service needs access to package manager service, app ops
        // service, and permissions service, therefore we start it after them.
        // Start sensor service in a separate thread. Completion should be checked
        // before using it.
        mSensorServiceStart = SystemServerInitThreadPool.get().submit(() -> {
            TimingsTraceLog traceLog = new TimingsTraceLog(
                    SYSTEM_SERVER_TIMING_ASYNC_TAG, Trace.TRACE_TAG_SYSTEM_SERVER);
            traceLog.traceBegin(START_SENSOR_SERVICE);
            startSensorService();
            traceLog.traceEnd();
        }, START_SENSOR_SERVICE);
    }


```
# 七、SystemServer 的核心服务有哪些

## 1.SystemServer 核心服务
 有 `BatteryService 、UsageStatsService、WebViewUpdateService、BinderCallsStatsService `4种核心服务。

![SystemServer 4种核心服务](https://upload-images.jianshu.io/upload_images/5851256-271bd91e1d6d85a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.startCoreServices 代码 
```

    /**
     * Starts some essential services that are not tangled up in the bootstrap process.
     */
    private void startCoreServices() {
        // 启动 BatteryService 管理电池服务（电压、电量、温度）
        traceBeginAndSlog("StartBatteryService");
        // Tracks the battery level.  Requires LightService.
        mSystemServiceManager.startService(BatteryService.class);
        traceEnd();

        // 启动 UsageStatsService 收集应用持久化数据的服务
        traceBeginAndSlog("StartUsageService");
        mSystemServiceManager.startService(UsageStatsService.class);
        mActivityManagerService.setUsageStatsManager(
                LocalServices.getService(UsageStatsManagerInternal.class));
        traceEnd();

        // 启动 WebViewUpdateService 监视 WebView 是否更新
        if (mPackageManager.hasSystemFeature(PackageManager.FEATURE_WEBVIEW)) {
            traceBeginAndSlog("StartWebViewUpdateService");
            mWebViewUpdateService = mSystemServiceManager.startService(WebViewUpdateService.class);
            traceEnd();
        }

        //启动 CPU Binder 调度服务
        traceBeginAndSlog("StartBinderCallsStatsService");
        BinderCallsStatsService.start();
        traceEnd();
    }

```
# 八、SystemServer 的其他服务有哪些

## 1.startOtherServices 启动的服务主要有： 
`KeyChainSystemService、TelecomLoaderService、AccountManagerService、ContentService、DropBoxManagerService、VibratorService、AlarmManagerService、Watchdog、
InputManagerService、WindowManagerService、IpConnectivityMetrics、NetworkWatchlistService、PinnerService`等服务。

![SystemServer 的其他服务](https://upload-images.jianshu.io/upload_images/5851256-c749f4a94da7de8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2.startOtherServices 代码 
```

    /**
     * Starts a miscellaneous grab bag of stuff that has yet to be refactored
     * and organized.
     */
    private void startOtherServices() {
            ... ...

            
            traceBeginAndSlog("StartKeyAttestationApplicationIdProviderService");
            ServiceManager.addService("sec_key_att_app_id_provider",
                    new KeyAttestationApplicationIdProviderService(context));
            traceEnd();
            // 启动 KeyChainSystemService 
            traceBeginAndSlog("StartKeyChainSystemService");
            mSystemServiceManager.startService(KeyChainSystemService.class);
            traceEnd();

            traceBeginAndSlog("StartSchedulingPolicyService");
            ServiceManager.addService("scheduling_policy", new SchedulingPolicyService());
            traceEnd();
            // 启动 TelecomLoaderService  
            traceBeginAndSlog("StartTelecomLoaderService");
            mSystemServiceManager.startService(TelecomLoaderService.class);
            traceEnd();

            traceBeginAndSlog("StartTelephonyRegistry");
            telephonyRegistry = new TelephonyRegistry(context);
            ServiceManager.addService("telephony.registry", telephonyRegistry);
            traceEnd();

            traceBeginAndSlog("StartEntropyMixer");
            mEntropyMixer = new EntropyMixer(context);
            traceEnd();

            mContentResolver = context.getContentResolver();

            // 启动 用户管理服务 ，必现在StartContentService 之前
            traceBeginAndSlog("StartAccountManagerService");
            mSystemServiceManager.startService(ACCOUNT_SERVICE_CLASS);
            traceEnd();
            // 启动 ContentService
            traceBeginAndSlog("StartContentService");
            mSystemServiceManager.startService(CONTENT_SERVICE_CLASS);
            traceEnd();
            // 安装系统Provider 例如 SettingProvider CantacttProvider
            traceBeginAndSlog("InstallSystemProviders");
            mActivityManagerService.installSystemProviders();
            // Now that SettingsProvider is ready, reactivate SQLiteCompatibilityWalFlags
            SQLiteCompatibilityWalFlags.reset();
            traceEnd();

            // 启动 DropBoxManagerService  
            // 由于 依赖SettingsProvider，必须在InstallSystemProviders之后启动
            traceBeginAndSlog("StartDropBoxManager");
            mSystemServiceManager.startService(DropBoxManagerService.class);
            traceEnd();
            //启动 VibratorService 震动服务
            traceBeginAndSlog("StartVibratorService");
            vibrator = new VibratorService(context);
            ServiceManager.addService("vibrator", vibrator);
            traceEnd();

            if (!isWatch) {
                traceBeginAndSlog("StartConsumerIrService");
                consumerIr = new ConsumerIrService(context);
                ServiceManager.addService(Context.CONSUMER_IR_SERVICE, consumerIr);
                traceEnd();
            }
            // 启动 AlarmManagerService
            traceBeginAndSlog("StartAlarmManagerService");
            if(!sMtkSystemServerIns.startMtkAlarmManagerService()){
                mSystemServiceManager.startService(AlarmManagerService.class);
            }
            traceEnd();
            // 初始化 看门狗
            traceBeginAndSlog("InitWatchdog");
            final Watchdog watchdog = Watchdog.getInstance();
            watchdog.init(context, mActivityManagerService);
            traceEnd();
            //启动 InputManagerService
            traceBeginAndSlog("StartInputManagerService");
            inputManager = new InputManagerService(context);
            traceEnd();
            //启动 WindowManagerService
            traceBeginAndSlog("StartWindowManagerService");
            // WMS needs sensor service ready
            ConcurrentUtils.waitForFutureNoInterrupt(mSensorServiceStart, START_SENSOR_SERVICE);
            mSensorServiceStart = null;
            wm = WindowManagerService.main(context, inputManager,
                    mFactoryTestMode != FactoryTest.FACTORY_TEST_LOW_LEVEL,
                    !mFirstBoot, mOnlyCore, new PhoneWindowManager());
            ServiceManager.addService(Context.WINDOW_SERVICE, wm, /* allowIsolated= */ false,
                    DUMP_FLAG_PRIORITY_CRITICAL | DUMP_FLAG_PROTO);
            ServiceManager.addService(Context.INPUT_SERVICE, inputManager,
                    /* allowIsolated= */ false, DUMP_FLAG_PRIORITY_CRITICAL);
            traceEnd();

            traceBeginAndSlog("SetWindowManagerService");
            mActivityManagerService.setWindowManager(wm);
            traceEnd();

            traceBeginAndSlog("WindowManagerServiceOnInitReady");
            wm.onInitReady();
            traceEnd();

            // Start receiving calls from HIDL services. Start in in a separate thread
            // because it need to connect to SensorManager. This have to start
            // after START_SENSOR_SERVICE is done.
            SystemServerInitThreadPool.get().submit(() -> {
                TimingsTraceLog traceLog = new TimingsTraceLog(
                        SYSTEM_SERVER_TIMING_ASYNC_TAG, Trace.TRACE_TAG_SYSTEM_SERVER);
                traceLog.traceBegin(START_HIDL_SERVICES);
                startHidlServices();
                traceLog.traceEnd();
            }, START_HIDL_SERVICES);

            if (!isWatch) {
                traceBeginAndSlog("StartVrManagerService");
                mSystemServiceManager.startService(VrManagerService.class);
                traceEnd();
            }

            traceBeginAndSlog("StartInputManager");
            inputManager.setWindowManagerCallbacks(wm.getInputMonitor());
            inputManager.start();
            traceEnd();

            // TODO: Use service dependencies instead.
            traceBeginAndSlog("DisplayManagerWindowManagerAndInputReady");
            mDisplayManagerService.windowManagerAndInputReady();
            traceEnd();

            // Skip Bluetooth if we have an emulator kernel
            // TODO: Use a more reliable check to see if this product should
            // support Bluetooth - see bug 988521
            if (isEmulator) {
                Slog.i(TAG, "No Bluetooth Service (emulator)");
            } else if (mFactoryTestMode == FactoryTest.FACTORY_TEST_LOW_LEVEL) {
                Slog.i(TAG, "No Bluetooth Service (factory test)");
            } else if (!context.getPackageManager().hasSystemFeature
                       (PackageManager.FEATURE_BLUETOOTH)) {
                Slog.i(TAG, "No Bluetooth Service (Bluetooth Hardware Not Present)");
            } else {
                traceBeginAndSlog("StartBluetoothService");
                mSystemServiceManager.startService(BluetoothService.class);
                traceEnd();
            }
            // 启动 IpConnectivityMetrics
            traceBeginAndSlog("IpConnectivityMetrics");
            mSystemServiceManager.startService(IpConnectivityMetrics.class);
            traceEnd();
            // 启动 NetworkWatchlistService
            traceBeginAndSlog("NetworkWatchlistService");
            mSystemServiceManager.startService(NetworkWatchlistService.Lifecycle.class);
            traceEnd();
            // 启动 PinnerService 
            traceBeginAndSlog("PinnerService");
            mSystemServiceManager.startService(PinnerService.class);
            traceEnd();
        } catch (RuntimeException e) {

```




**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

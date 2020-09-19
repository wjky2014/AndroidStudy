 
##### 和您一起终身学习，这里是程序员Android 

>文章转载于网络，原文如下：[原文链接](https://blog.csdn.net/FightFightFight/article/details/79532191)

![](https://upload-images.jianshu.io/upload_images/5851256-600dca83084c7424.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##概述
        
**PowerManagerService** 是负责管理、协调设备电源管理的系统服务之一，设备常见功能如亮灭屏、亮度调节、低电量模式、保持**CPU**唤醒等，都会通过**PMS**的协调和处理。其继承自**SystemService**,因此具有**SystemService**子类的共性：具有生命周期方法，由**SystemServer**启动、注册到系统服务中,通过**Binder**和其他组件进行交互等。

其生命周期方法如下：
    
- 构造方法：通过反射调用，获取实例；
 - `onstart()`方法：开启对应的`SystemService`；
 - `onBootPhase()`方法：在`SystemService`服务的启动过程中指定服务的启动阶段，每个阶段指定特定的工作；

由于是系统服务，不仅需要掌握其启动流程，还要了解和其他服务之间的交互，下面就`PMS`的流程和功能进行具体的分析。

` PMS`类图如下，其中红色部分表示在本篇文章中分析到的类：
![ PMS类图](https://upload-images.jianshu.io/upload_images/5851256-31e412ca22c36a99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1.1.PMS的启动
        
和`SystemService`的其他子类一样，`PMS`由`SystemServer`通过反射的方式启动，首先，在`SystemServer`的`main()`方法中，调用了自身的`run()`方法，并在`run()`方法中启动三类服务:引导服务、核心服务和其他服务，引导服务中启动的是一些依赖性比较强的服务，其中就包括了`PMS`，具体如下：

在SystemServer的`main()`中：
```
//The main entry point from zygote.
public static void main(String[] args) {
    new SystemServer().run();
}
```
在SystemServer的`run(`)中：
```

private void run() {
    //......
    try {
        startBootstrapServices();//启动引导服务
        startCoreServices();//启动核心服务
        startOtherServices();//启动其他服务
    } catch (Throwable ex) {
    } finally {
        Trace.traceEnd(Trace.TRACE_TAG_SYSTEM_SERVER);
    }
    ......
}
```
 所谓引导服务是指一些具有高度相互依赖性的服务，在启动引导服务时，启动了PMS，以下代码是PMS的启动以及和其他服务的依赖关系：
```
private void startBootstrapServices() {
 	//通过SystemManagerService启动PMS服务
    mPowerManagerService = mSystemServiceManager.
                startService(PowerManagerService.class);
    //AMS中初始化PowerManager
    mActivityManagerService.initPowerManagement();
}
```
逐步分析，首先看PMS的启动，调用了`SystemServiceManager`的`startService()`方法进行了启动，在`startService()`中，首先通过了传入的类名获取了Class对象，然后使用反射机制，通过Class对象获取了PMS的构造函数，从而获得了一个PMS对象：
```
public SystemService startService(String className) {
    final Class<SystemService> serviceClass;
    serviceClass = (Class<SystemService>)Class.forName(className);
    return startService(serviceClass);
}
public <T extends SystemService> T startService(Class<T> serviceClass) {
    try {
        final T service;
        try {
        	//获取实例
            Constructor<T> constructor = 
                  serviceClass.getConstructor(Context.class);
            service = constructor.newInstance(mContext);
        } catch (InstantiationException ex) {
        }
        mServices.add(service);//添加到List中，进行生命周期管理
        try {
            service.onStart();//启动服务
        } catch (RuntimeException ex) {
        }
        return service;
    }
}
```
   在获取了PMS的实例后，添加到了一个`ArrayList`中，只有在该`ArrayList`中的`SystemService`才能接收到生命周期方法回调;然后调用了`onStart()`方法.因此，对于PMS，先执行了PMS的构造方法，接着执行了`onStart()`方法。首先来看其构造方法：
```
public PowerManagerService(Context context) {
    super(context);
    mContext = context;
    //获取一个系统级别的HandlerThread，继承于Thread
    mHandlerThread = new ServiceThread(TAG,
            Process.THREAD_PRIORITY_DISPLAY, false );
    mHandlerThread.start();//开启线程
    //根据Looper实例化一个Handler
    mHandler = new PowerManagerHandler(mHandlerThread.getLooper());
    synchronized (mLock) {
    	//获取当应用申请wakelock后让CUP保持激活状态的Suspendlocker实例
        mWakeLockSuspendBlocker = 
             createSuspendBlockerLocked("PowerManagerService.WakeLocks");
        //获取当显示屏开启、显示屏准备就绪或者有用户活动后让CPU保持激活状态的Suspendlocker
        mDisplaySuspendBlocker = 
            createSuspendBlockerLocked("PowerManagerService.Display");
        //申请PowerManagerService.Display类型的suspendBloker锁
        mDisplaySuspendBlocker.acquire();
       //持有Display锁的bool值
        mHoldingDisplaySuspendBlocker = true;
        //AutoSuspend模式是否开启
        mHalAutoSuspendModeEnabled = false;
        //是否处于交互模式
        mHalInteractiveModeEnabled = true;
        //设置wakefulness表示亮屏状态
        mWakefulness = WAKEFULNESS_AWAKE;
		//本地方法
        nativeInit();//初始化
        nativeSetAutoSuspend(false);//设置是否开启anto suspend模式
        nativeSetInteractive(true);//设置是否处于交互模式
        nativeSetFeature(POWER_FEATURE_DOUBLE_TAP_TO_WAKE, 0);
        getPowerHintSceneIdConfig();
    }
}
```
在PMS构造方法中，首先获取了一个`HandlerThread`,然后使用该`HandlerThread`的Looper实例化`PowerManagerHandler`,如果不指定`looper`,则使用当前线程默认`Looper`，因此`PowerManagerHandler`会根据`HandlerThread`中的`Looper`，在HandlerThread中进行异步的操作；其次获取了两个`Suspend`锁，`SuspendBlocker`是一种锁机制，只用于系统内部，上层申请的`wakelock`锁在`PMS`中都会反映为`SuspendBlocker`锁。这里获取的两个`Suspend`锁在申请`wakelock`时会用到，这块在`wakelock`申请时会进行详细分析；最后，调用了本地方法，这几个方法会通过`JNI`层调用到`HAL`层。
        构造方法执行完毕后，执行`onStart()`方法：
```
@Override
public void onStart() {
    //发布到系统服务中
    publishBinderService(Context.POWER_SERVICE, new BinderService());
    //发布到本地服务
    publishLocalService(PowerManagerInternal.class, new LocalService());
	//设置Watchdog监听
    Watchdog.getInstance().addMonitor(this);
    Watchdog.getInstance().addThread(mHandler);
}
```
  在该方法中，首先对该服务进行`Binder`注册和本地注册，当进行`Binder`注册后，在其他模块中就可以通过`Binder`机制获取其实例，同理，当进行本地注册后，只有在`System`进程才能获取到其实例；最后设置`Watchdog`的监听。
        对服务进行远程注册，是通过`Binder`机制实现的，实际上，从代码中可以出，注册的并不是PMS本身，而是其内部类——`BinderService`,`BinderService`继承自`IPowerManager.Stub`，`IPowerManager.Stub`继承自`Binder`并且实现了`IPowerManager,`因此可以知道，`BinderService`作为`Binder`的服务端，可以和客户端进行交互；注册的过程在`ServiceManager`中进行的，如下：
```
public static void addService(String name, IBinder service, boolean allowIsolated) {
    try {
        getIServiceManager().addService(name, service, allowIsolated);
    } catch (RemoteException e) {
        Log.e(TAG, "error in addService", e);
    }
}
```
 当通过`ServiceManager`注册后，就可以根据`Context.POWER_SERVICE`在其他服务中获取对应的`IBinder`了，以PMS为例，当从其他应用中获取了PMS服务后，就可以调用`PMS.BinderService`中的方法。所以，PMS中的`BinderService`相当于服务端，其中的方法可以供其他应用进行调用，从而完成和PMS的交互。
       通过`Binder`进行注册是为了供其他应用或系统服务和PMS进行交互，那么本地注册则表示只能在`System进程`内部调用，和`BinderService`一样，本地注册也并非注册的是PMS，而是另一个内部类——`LocalService`,`LocalService`继承自`PowerManagerInternal`（带有Internal的类一般都在System进程内使用）；`Binder`注册是在`SystemManager`中进行注册的，本地注册则是在`LocalServices`中进行注册，其注册方法如下：
```
public static <T> void addService(Class<T> type, T service) {
    synchronized (sLocalServiceObjects) {
        if (sLocalServiceObjects.containsKey(type)) {
            throw new IllegalStateException("Overriding service registration");
        }
        sLocalServiceObjects.put(type, service);
    }
```
注册完成之后，就可以在System进程内，通过`PowerManagerInternal.class`获取`PMS.LocalService`对象，从而完成交互了。在远程注册和本地注册都完成以后，给PMS设置了watchDog监听，onStart()方法调用完毕。此时，继续回到SytemServer中，`startBootstrapServices()`方法中启动PMS部分执行完成了，然后根据`SystemService`的生命周期，会开始执行`onBootPhase()`，这个方法的功能是为所有的已启动的服务指定启动阶段,从而可以在指定的启动阶段来做指定的工作。源代码如下：
```
* Starts the specified boot phase for all system services that have been started   
 * up to this point.
 * @param phase The boot phase to start.
public void startBootPhase(final int phase) {
    if (phase <= mCurrentPhase) {
        throw new IllegalArgumentException("Next phase must be larger than previous");
    }
    mCurrentPhase = phase;
    try {
        final int serviceLen = mServices.size();
        for (int i = 0; i < serviceLen; i++) {
            final SystemService service = mServices.get(i);
            try {
                service.onBootPhase(mCurrentPhase);
            } catch (Exception ex) {
            }
        }
    }
}
```
 在`SystemServiceManager`的`startBootPhase()`中，调用`SystemService`的`onBootPhase(int)`方法，此时每个`SystemService`都会执行其对应的`onBootPhase()`方法。通过在`SystemServiceManager`中传入不同的形参，回调所有`SystemService`的`onBootPhase()`,根据形参的不同，在方法实现中完成不同的工作，在`SystemService`中定义了六个阶段：

- 1.`SystemService.PHASE_WAIT_FOR_DEFAULT_DISPLAY`
这是一个依赖 项，只有DisplayManagerService中进行了对应处理；

- 2. `SystemService.PHASE_LOCK_SETTINGS_READY:`
经过这个引导阶段后，服务才可以接收到wakelock相关设置数据；

- 3.`SystemService.PHASE_SYSTEM_SERVICES_READY`
经过这个引导阶段 后，服务才可以安全地使用核心系统服务

- 4.`SystemService.PHASE_ACTIVITY_MANAGER_READY:`
经过这个引导阶 段后，服务可以发送广播

- 5.`SystemService.PHASE_THIRD_PARTY_APPS_CAN_START:`
经过这个引 导阶段后，服务可以启动第三方应用，第三方应用也可以通过Binder来调 用服务。

- 6.`SystemService.PHASE_BOOT_COMPLETED:`
经过这个引导阶段后，说明服务启动完成，这时用户就可以和设备进行交互。

因此，只要在其他模块中调用了`SystemServiceManager.startBootPhase()`,都会触发各自的`onBootPhase()`。PMS的`onBootPhase()`方法只对引导阶段的2个阶段做了处理，具体代码如下：

```
@Override
public void onBootPhase(int phase) {
    synchronized (mLock) {
        if (phase == PHASE_THIRD_PARTY_APPS_CAN_START) {
        	//统计启动的apk个数
            incrementBootCount();
        } else if (phase == PHASE_BOOT_COMPLETED) {
            mBootCompleted = true;
            PMSFactory.getInstance().createPowerManagerServiceUtils(mContext)
                                .setBootCompleted(true);
            //mDirty置位
            mDirty |= DIRTY_BOOT_COMPLETED;
            //更新用户活动时间
            userActivityNoUpdateLocked(
                    now, PowerManager.USER_ACTIVITY_EVENT_OTHER, 
                              0, Process.SYSTEM_UID);
            //更新电源状态信息
            updatePowerStateLocked();
            ......
        }
    }
}
```
在这个方法中，mDirty是一个二进制的标记位，用来表示电源状态哪一部分发生了改变，通过对其进行置位（|操作）、清零（～操作），得到二进制数各个位的值(0或1)，进行不同的处理 ，此外，这个方法中调用到了`updatePowerStateLocked()`方法，这是整个PMS中最重要的方法，这块会在下文中进行详细分析。此时，`SystemServer.startBootstrapServices()`执行完毕，生命周期方法也执行完毕。对于PMS，在`SystemServer.startOtherServices()`中还进行了一步操作：
```
mPowerManagerService.systemReady(mActivityManagerService.getAppOpsService());
```
  因此，当PMS依次执行完`构造方法`、`onStart()`、`onBootPhase()`（会调用多次）方法后，执行的下一个方法是`systemReady()`方法，这个方法中主要有以下5个功能：

- 1.获取各类本地服务和远程服务，如屏保(`DreamMangerService`)、窗口(`PhoneWindowManager`)、电池状态监听服务(`BatteryService`)等服务,用于进行交互：

```
synchronized (mLock) {
    mSystemReady = true;
    mAppOps = appOps;
    //和DreamManagerService交互
    mDreamManager = getLocalService(DreamManagerInternal.class);
    //和DisplayManagerService交互
    mDisplayManagerInternal = getLocalService(DisplayManagerInternal.class);
    //和WindowManagerService交互
    mPolicy = getLocalService(WindowManagerPolicy.class);
    //和BatteryService交互
    mBatteryManagerInternal = getLocalService(BatteryManagerInternal.class);
    //获取屏幕亮度
    PowerManager pm = (PowerManager) 
            mContext.getSystemService(Context.POWER_SERVICE);
    mScreenBrightnessSettingMinimum = pm.getMinimumScreenBrightnessSetting();
    mScreenBrightnessSettingMaximum = pm.getMaximumScreenBrightnessSetting();
    mScreenBrightnessSettingDefault = pm.getDefaultScreenBrightnessSetting();
    mScreenBrightnessForVrSettingDefault = 
            pm.getDefaultScreenBrightnessForVrSetting();
    SensorManager sensorManager = new SystemSensorManager(mContext, 
            mHandler.getLooper());
    //获取BatteryStatsService
    mBatteryStats = BatteryStatsService.getService();
    //mNotifier用于PMS和其他系统服务间的交互,以及广播的发送
    mNotifier = new Notifier(Looper.getMainLooper(), mContext, mBatteryStats,
            mAppOps, 
            createSuspendBlockerLocked("PowerManagerService.Broadcasts"),
            mPolicy);
    //无线充电相关
    mWirelessChargerDetector = new WirelessChargerDetector(sensorManager,
            createSuspendBlockerLocked
            ("PowerManagerService.WirelessChargerDetector"),
            mHandler);
    //监听Stetings中值的变化
    mSettingsObserver = new SettingsObserver(mHandler);
    //和LightsManager交互
    mLightsManager = getLocalService(LightsManager.class);
    mAttentionLight = 
             mLightsManager.getLight(LightsManager.LIGHT_ID_ATTENTION);
//mDisplayPowerCallbacks提供PMS和Display的接口，当
//DisplayPowerController发生改变，通过该接口回调PMS中的实现
    //initPowerManagement()方法中实例化了DisplayPowerController,
//DPC是和显示有关,如亮灭屏、背光调节
    mDisplayManagerInternal.initPowerManagement(
        mDisplayPowerCallbacks, mHandler, sensorManager);
```
- 2.注册用于和其他System交互的广播，如：
```
  // Register for broadcasts from other components of the system.
    //注册BatteryService中ACTION_BATTERY_CHANGED广播
    IntentFilter filter = new IntentFilter();
    filter.addAction(Intent.ACTION_BATTERY_CHANGED);
    filter.setPriority(IntentFilter.SYSTEM_HIGH_PRIORITY);
    mContext.registerReceiver(new BatteryReceiver(), filter, null, mHandler);
    //Dream相关
    filter = new IntentFilter();
    filter.addAction(Intent.ACTION_DREAMING_STARTED);
    filter.addAction(Intent.ACTION_DREAMING_STOPPED); 
    mContext.registerReceiver(new DreamReceiver(), filter, null, mHandler);
    //？？
    filter = new IntentFilter();
    filter.addAction(Intent.ACTION_USER_SWITCHED);
    mContext.registerReceiver(new UserSwitchedReceiver(), filter, null, mHandler);
    //Dock相关
    filter = new IntentFilter();
    filter.addAction(Intent.ACTION_DOCK_EVENT);
    mContext.registerReceiver(new DockReceiver(), filter, null, mHandler);

```
- 3.调用`updateSettingsLocked()`方法更新`Settings`中值的变化：
```
private void updateSettingsLocked() {
    final ContentResolver resolver = mContext.getContentResolver();
    //屏保是否支持
    mDreamsEnabledSetting = (Settings.Secure.getIntForUser(resolver,
            Settings.Secure.SCREENSAVER_ENABLED,
            mDreamsEnabledByDefaultConfig ? 1 : 0,
            UserHandle.USER_CURRENT) != 0);
    //休眠时是否启用屏保
    mDreamsActivateOnSleepSetting = (Settings.Secure.getIntForUser(resolver,
            Settings.Secure.SCREENSAVER_ACTIVATE_ON_SLEEP,
            mDreamsActivatedOnSleepByDefaultConfig ? 1 : 0,
            UserHandle.USER_CURRENT) != 0);
    //插入基座时屏保是否激活
    mDreamsActivateOnDockSetting = (Settings.Secure.getIntForUser(resolver,
            Settings.Secure.SCREENSAVER_ACTIVATE_ON_DOCK,
            mDreamsActivatedOnDockByDefaultConfig ? 1 : 0,
            UserHandle.USER_CURRENT) != 0);
    //设备在一段时间不活动后进入休眠或者屏保状态的时间，15*1000ms
    mScreenOffTimeoutSetting = Settings.System.getIntForUser(resolver,
            Settings.System.SCREEN_OFF_TIMEOUT, 
            DEFAULT_SCREEN_OFF_TIMEOUT,
            UserHandle.USER_CURRENT);
    /*设备在一段时间不活动后完全进入休眠状态之前的超时时间，
    该值必须大于SCREEN_OFF_TIMEOUT，否则设置了屏保后来不及显示屏保就sleep*/
    mSleepTimeoutSetting = Settings.Secure.getIntForUser(resolver,
            Settings.Secure.SLEEP_TIMEOUT, DEFAULT_SLEEP_TIMEOUT,
            UserHandle.USER_CURRENT);
    //充电时屏幕一直开启
    mStayOnWhilePluggedInSetting = Settings.Global.getInt(resolver,
            Settings.Global.STAY_ON_WHILE_PLUGGED_IN, 
            BatteryManager.BATTERY_PLUGGED_AC);
    //是否支持剧院模式
    mTheaterModeEnabled = Settings.Global.getInt(mContext.getContentResolver(),
            Settings.Global.THEATER_MODE_ON, 0) == 1;
    //屏幕保持常亮
    mAlwaysOnEnabled = mAmbientDisplayConfiguration.
            alwaysOnEnabled(UserHandle.USER_CURRENT);
    //双击唤醒屏幕设置
    if (mSupportsDoubleTapWakeConfig) {
        boolean doubleTapWakeEnabled = Settings.Secure.getIntForUser(resolver,
                Settings.Secure.DOUBLE_TAP_TO_WAKE, 
                DEFAULT_DOUBLE_TAP_TO_WAKE,
                        UserHandle.USER_CURRENT) != 0;
        if (doubleTapWakeEnabled != mDoubleTapWakeEnabled) {
            mDoubleTapWakeEnabled = doubleTapWakeEnabled;
            nativeSetFeature(POWER_FEATURE_DOUBLE_TAP_TO_WAKE, 
            mDoubleTapWakeEnabled ? 1 : 0);
        }
    }
    final String retailDemoValue = UserManager.isDeviceInDemoMode(mContext) ?
            "1" : "0";
    ...... 
//屏幕亮度
    mScreenBrightnessSetting = Settings.System.getIntForUser(resolver,
            Settings.System.SCREEN_BRIGHTNESS, mScreenBrightnessSettingDefault,
            UserHandle.USER_CURRENT);
    //自动调节亮度值(>0.0 <1.0)
    final float oldScreenAutoBrightnessAdjustmentSetting =
            mScreenAutoBrightnessAdjustmentSetting;
    mScreenAutoBrightnessAdjustmentSetting = 
        Settings.System.getFloatForUser(resolver,
            Settings.System.SCREEN_AUTO_BRIGHTNESS_ADJ, 0.0f,
            UserHandle.USER_CURRENT);
//重置临时亮度值
    if (oldScreenBrightnessSetting != getCurrentBrightnessSettingLocked()) {
        mTemporaryScreenBrightnessSettingOverride = -1;
    }
    if (oldScreenAutoBrightnessAdjustmentSetting != 
    mScreenAutoBrightnessAdjustmentSetting) {
        mTemporaryScreenAutoBrightnessAdjustmentSettingOverride = Float.NaN;
    }
    //亮度调节模式，自动1，正常0
    mScreenBrightnessModeSetting = Settings.System.getIntForUser(resolver,
            Settings.System.SCREEN_BRIGHTNESS_MODE,
            Settings.System.SCREEN_BRIGHTNESS_MODE_MANUAL, 
        UserHandle.USER_CURRENT);
    //低电量模式是否可用，1表示true
    final boolean lowPowerModeEnabled = Settings.Global.getInt(resolver,
            Settings.Global.LOW_POWER_MODE, 0) != 0;
    if (lowPowerModeEnabled != mLowPowerModeSetting
            || autoLowPowerModeConfigured != mAutoLowPowerModeConfigured) {
        mLowPowerModeSetting = lowPowerModeEnabled;
        mAutoLowPowerModeConfigured = autoLowPowerModeConfigured;
        //更新低电量模式
        updateLowPowerModeLocked();
    }
    //标志位置位
    mDirty |= DIRTY_SETTINGS;
}
```
这里需要注意这两个值：`mTemporaryScreenBrightnessSettingOverride`和`mTemporaryScerrnAutoBrightnessAdjustmentSettingsOverride`,这两个值分别表示一个临时亮度值和临时自动亮度调节比例，和亮度的调节有关。之所以是临时亮度值，是因为在自动调节亮度关闭时，从`Settings`中或者`SystemUI`中拉进度条时，首先会通过Binder调用`setTemporaryScreenBrightnessSettingOverride()`方法将亮度值赋给它，如果调节完毕手指抬起，此时Settings.Global等里面亮度值发生变化，则在PMS中会回调`SettingsObserver`中的onChange()方法，又会回调`updateSettingsLocked()`方法，又置为-1和NaN，也就是上面这段方法。

- 4.调用`readConfigurationLocked()`方法读取配置文件中的默认值：
```
private void readConfigurationLocked() {
    final Resources resources = mContext.getResources();
    /**
     * auto_suspend模式是否和display分离
     * 如果为false，则在亮屏前调用autosuspend_disable(),灭屏后调用
     * autosuspend_enable();
     * 如果为ture，则调用autosuspend_display()和autosuspend_enable()独立于display 
     * on/off.
     */
    mDecoupleHalAutoSuspendModeFromDisplayConfig = resources.getBoolean(
                 com.android.internal.R.bool.
                 config_powerDecoupleAutoSuspendModeFromDisplay);
    /**
     * interactive模式是否和display分离
     * 如果为false，则在亮屏前调用setInteractive(..., true),灭屏后调用
     * setInteractive(...,false);
     * 如果为ture，则调用setInteractive(...)独立于display on/off.
     */
    mDecoupleHalInteractiveModeFromDisplayConfig = resources.getBoolean(        
          com.android.internal.R.bool
           .config_powerDecoupleInteractiveModeFromDisplay);
    //插拔USB是否亮屏
    mWakeUpWhenPluggedOrUnpluggedConfig = resources.getBoolean(
            com.android.internal.R.bool.config_unplugTurnsOnScreen);
    //设备处于剧院模式时，插拔USB是否亮屏
    mWakeUpWhenPluggedOrUnpluggedInTheaterModeConfig = resources.getBoolean(
            com.android.internal.R.bool.config_allowTheaterModeWakeFromUnplug);
    //是否允许设备由于接近传感器而关闭屏幕时CPU挂起，进入suspend状态
    mSuspendWhenScreenOffDueToProximityConfig = resources.getBoolean(
            com.android.internal.R.bool.
                config_suspendWhenScreenOffDueToProximity);
    //是否支持屏保
    mDreamsSupportedConfig = resources.getBoolean(
            com.android.internal.R.bool.config_dreamsSupported);
    //是否屏保默认打开--false
    mDreamsEnabledByDefaultConfig = resources.getBoolean(
            com.android.internal.R.bool.config_dreamsEnabledByDefault);
    //充电和睡眠时屏保是否激活
    mDreamsActivatedOnSleepByDefaultConfig = resources.getBoolean(
            com.android.internal.R.bool.config_dreamsActivatedOnSleepByDefault);
    //Dock时屏保是否激活
    mDreamsActivatedOnDockByDefaultConfig = resources.getBoolean(
            com.android.internal.R.bool.config_dreamsActivatedOnDockByDefault);
    //放电时是否允许进入屏保
    mDreamsEnabledOnBatteryConfig = resources.getBoolean(
            com.android.internal.R.bool.config_dreamsEnabledOnBattery);
    //充电时允许屏保的最低电量，使用-1禁用此功能
    mDreamsBatteryLevelMinimumWhenPoweredConfig = resources.getInteger(
            com.android.internal.R.integer.
             config_dreamsBatteryLevelMinimumWhenPowered);
    //放电时允许屏保的最低电量，使用-1禁用此功能，默认15
    mDreamsBatteryLevelMinimumWhenNotPoweredConfig = resources.getInteger(
            com.android.internal.R.integer.
             config_dreamsBatteryLevelMinimumWhenNotPowered);
    //电亮下降到该百分点，当用户活动超时后不进入屏保，默认5
    mDreamsBatteryLevelDrainCutoffConfig = resources.getInteger(
            com.android.internal.R.integer.config_dreamsBatteryLevelDrainCutoff);
    //如果为true，则直到关闭屏幕并执行屏幕关闭动画之后，才开始Doze，默认false
    mDozeAfterScreenOffConfig = resources.getBoolean(
            com.android.internal.R.bool.config_dozeAfterScreenOff);
    //用户活动超时的最小时间，默认10000ms,必须大于0
    mMinimumScreenOffTimeoutConfig = resources.getInteger(
            com.android.internal.R.integer.config_minimumScreenOffTimeout);
    //用户活动超时进入且关闭屏幕前屏幕变暗的最大时间，默认7000ms，必须大于0
    mMaximumScreenDimDurationConfig = resources.getInteger(
            com.android.internal.R.integer.config_maximumScreenDimDuration);
    //屏幕变暗的时长比例，如果用于超时时间过短，则在7000ms的基础上按还比例减少，默认20%
    mMaximumScreenDimRatioConfig = resources.getFraction(
            com.android.internal.R.fraction.config_maximumScreenDimRatio, 1, 1);
    //是否支持双击唤醒屏幕
    mSupportsDoubleTapWakeConfig = resources.getBoolean(
            com.android.internal.R.bool.config_supportDoubleTapWake);
}
```
- 5.注册`SettingsObserver`监听：
```

// Register for settings changes.
resolver.registerContentObserver(Settings.Secure.getUriFor(
        Settings.Secure.SCREENSAVER_ENABLED),
        false, mSettingsObserver, UserHandle.USER_ALL);
resolver.registerContentObserver(Settings.Secure.getUriFor(
        Settings.Secure.SCREENSAVER_ACTIVATE_ON_SLEEP),
        false, mSettingsObserver, UserHandle.USER_ALL);
resolver.registerContentObserver(Settings.Secure.getUriFor(
        Settings.Secure.SCREENSAVER_ACTIVATE_ON_DOCK),
        false, mSettingsObserver, UserHandle.USER_ALL);
resolver.registerContentObserver(Settings.System.getUriFor(
        Settings.System.SCREEN_OFF_TIMEOUT),
```
 `systemReady()`方法中分析完毕。执行完这个方法后，PMS的启动过程分析完毕。在下文中进行核心方法的分析。


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

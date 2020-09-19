 

##### 和您一起终身学习，这里是程序员Android 

**PowerManagerService** 提供Android系统的电源管理服务，主要功能是控制系统待机状态，屏幕显示，亮度调节，光线/距离传感器的控制等。

相关代码在以下文件中：
```
frameworks/base/services/java/com/android/server/SystemServer.java
frameworks/base/core/java/android/os/PowerManager.java
frameworks/base/services/core/java/com/android/server/power/PowerManagerService.java
frameworks/base/services/core/jni/com_android_server_power_PowerManagerService.cpp
frameworks/base/core/java/android/os/PowerManagerInternal.java
frameworks/base/services/core/java/com/android/server/power/Notifier.java
device/qcom/common/power/power.c
system/core/libsuspend/autosuspend.c
hardware/libhardware_legacy/power/power.c
```

**初始化流程**

跟其他系统服务一样，PowerManagerService也是继承于SystemService并通过SystemServer启动。
**SystemServer**
` frameworks/base/services/java/com/android/server/SystemServer.java`
```
private void startBootstrapServices() {
  ......
  mPowerManagerService = mSystemServiceManager.startService(PowerManagerService.class);
  ......
}
```
**PowerManagerService**
` frameworks/base/services/core/java/com/android/server/power/PowerManagerService.java`
```
public final class PowerManagerService extends SystemService
        implements Watchdog.Monitor {
    ......
}
```
在SystemServer的startBootstrapServices中，通过SystemServiceManager.startService启动了PowerManagerService，下面首先来看PowerManagerService构造方法。
` frameworks/base/services/core/java/com/android/server/power/PowerManagerService.java`

```
public PowerManagerService(Context context) {
    super(context);
    // mContext赋值为SystemContext
    mContext = context;
    // 创建消息处理线程并启动，创建关联消息处理线程的handler对象
    mHandlerThread = new ServiceThread(TAG,
            Process.THREAD_PRIORITY_DISPLAY, false /*allowIo*/);
    mHandlerThread.start();
    mHandler = new PowerManagerHandler(mHandlerThread.getLooper());
    qcNsrmPowExt = new QCNsrmPowerExtension(this);
    synchronized (mLock) {
        // 创建"PowerManagerService.WakeLocks"的SuspendBlocker
        mWakeLockSuspendBlocker = createSuspendBlockerLocked("PowerManagerService.WakeLocks");
        // 创建"PowerManagerService.Display"的SuspendBlocker
        mDisplaySuspendBlocker = createSuspendBlockerLocked("PowerManagerService.Display");
        // 请求DisplaySuspendBlocker，禁止系统进入休眠
        mDisplaySuspendBlocker.acquire();
        mHoldingDisplaySuspendBlocker = true;
        mHalAutoSuspendModeEnabled = false;
        mHalInteractiveModeEnabled = true;
        // 设置mWakefulness为唤醒状态
        mWakefulness = WAKEFULNESS_AWAKE;
        // 进入到native层初始化
        nativeInit();
        nativeSetAutoSuspend(false);
        nativeSetInteractive(true);
        nativeSetFeature(POWER_FEATURE_DOUBLE_TAP_TO_WAKE, 0);
    }
}
```
PowerManagerService构造函数中首先创建了处理消息的进程及对应的handler对象以进行消息处理，然后创建SuspendBlocker对象，用于WakeLocks与Display，并设置mWakefulness的初始状态为WAKEFULNESS_AWAKE，最后进入到native层初始化。下面先看一下关于mWakefulness的定义。

```
>>> frameworks/base/core/java/android/os/PowerManagerInternal.java

/**
 * 设备处于休眠状态，只能被wakeUp()唤醒．
 */
public static final int WAKEFULNESS_ASLEEP = 0;

/**
 * 设备处于正常工作(fully awake)状态．
 */
public static final int WAKEFULNESS_AWAKE = 1;

/**
 * 设备处于播放屏保状态．
 */
public static final int WAKEFULNESS_DREAMING = 2;

/**
 * 设备处于doze状态，只有低耗电的屏保可以运行，其他应用被挂起．
 */
public static final int WAKEFULNESS_DOZING = 3;
```
继续回到PowerManagerService构造函数的native初始化中，首先来看nativeInit的实现。
`frameworks/base/services/core/jni/com_android_server_power_PowerManagerService.cpp`

```
static const JNINativeMethod gPowerManagerServiceMethods[] = {
    /* name, signature, funcPtr */
    { "nativeInit", "()V",
            (void*) nativeInit },
    { "nativeAcquireSuspendBlocker", "(Ljava/lang/String;)V",
            (void*) nativeAcquireSuspendBlocker },
    { "nativeReleaseSuspendBlocker", "(Ljava/lang/String;)V",
            (void*) nativeReleaseSuspendBlocker },
    { "nativeSetInteractive", "(Z)V",
            (void*) nativeSetInteractive },
    { "nativeSetAutoSuspend", "(Z)V",
            (void*) nativeSetAutoSuspend },
    { "nativeSendPowerHint", "(II)V",
            (void*) nativeSendPowerHint },
    { "nativeSetFeature", "(II)V",
            (void*) nativeSetFeature },
};
```
PowerManagerService中的native方法定义如上，nativeInit即调用nativeInit()。
` frameworks/base/services/core/jni/com_android_server_power_PowerManagerService.cpp`

```
static void nativeInit(JNIEnv* env, jobject obj) {
    // 创建一个全局对象，引用PMS
    gPowerManagerServiceObj = env->NewGlobalRef(obj);
    // 利用hw_get_module加载power模块
    status_t err = hw_get_module(POWER_HARDWARE_MODULE_ID,
            (hw_module_t const**)&gPowerModule);
    if (!err) {
        gPowerModule->init(gPowerModule);
    } else {
        ALOGE("Couldn't load %s module (%s)", POWER_HARDWARE_MODULE_ID, strerror(-err));
    }
}
```
nativeInit的主要任务时装载power模块，该模块由厂商实现，以高通为例，如下。
`device/qcom/common/power/power.c`
```
tatic struct hw_module_methods_t power_module_methods = {
    .open = NULL,
};

struct power_module HAL_MODULE_INFO_SYM = {
    .common = {
        .tag = HARDWARE_MODULE_TAG,
        .module_api_version = POWER_MODULE_API_VERSION_0_2,
        .hal_api_version = HARDWARE_HAL_API_VERSION,
        .id = POWER_HARDWARE_MODULE_ID,
        .name = "QCOM Power HAL",
        .author = "Qualcomm",
        .methods = &power_module_methods,
    },

    .init = power_init,
    .powerHint = power_hint,
    .setInteractive = set_interactive,
};
```

power_module中实现了init，powerHint，setInteractive，nativeInit最终调用到HAL power模块的power_init具体实现中。接着看native初始化nativeSetAutoSuspend的实现。

`frameworks/base/services/core/jni/com_android_server_power_PowerManagerService.cpp`

```
static void nativeSetAutoSuspend(JNIEnv* /* env */, jclass /* clazz */, jboolean enable) {
    if (enable) {
        ALOGD_IF_SLOW(100, "Excessive delay in autosuspend_enable() while turning screen off");
        autosuspend_enable();
    } else {
        ALOGD_IF_SLOW(100, "Excessive delay in autosuspend_disable() while turning screen on");
        autosuspend_disable();
    }
}
```
`system/core/libsuspend/autosuspend.c`

```
int autosuspend_disable(void)
{
    int ret;

    ret = autosuspend_init();
    if (ret) {
        return ret;
    }

    ALOGV("autosuspend_disable\n");

    if (!autosuspend_enabled) {
        return 0;
    }

    ret = autosuspend_ops->disable();
    if (ret) {
        return ret;
    }

    autosuspend_enabled = false;
    return 0;
}
```
nativeSetAutoSuspend最终调用到libsuspend(参考Android电源管理系列之libsuspend)的autosuspend_disable禁止系统休眠。继续看native初始化nativeSetInteractive，nativeSetFeature的实现
`frameworks/base/services/core/jni/com_android_server_power_PowerManagerService.cpp`

```
static void nativeSetInteractive(JNIEnv* /* env */, jclass /* clazz */, jboolean enable) {
    if (gPowerModule) {
        if (enable) {
            ALOGD_IF_SLOW(20, "Excessive delay in setInteractive(true) while turning screen on");
            gPowerModule->setInteractive(gPowerModule, true);
        } else {
            ALOGD_IF_SLOW(20, "Excessive delay in setInteractive(false) while turning screen off");
            gPowerModule->setInteractive(gPowerModule, false);
        }
    }
}

static void nativeSetFeature(JNIEnv *env, jclass clazz, jint featureId, jint data) {
    int data_param = data;

    if (gPowerModule && gPowerModule->setFeature) {
        gPowerModule->setFeature(gPowerModule, (feature_t)featureId, data_param);
    }
}
```
同nativeInit一样，最终都是调用到HAL power模块的具体实现中。以上是构造函数的分析流程，下面继续看PowerManagerService在系统启动过程中回调onStart()，onBootPhase()，systemReady()的实现。
`frameworks/base/services/core/java/com/android/server/power/PowerManagerService.java`
```

public void onStart() {
    publishBinderService(Context.POWER_SERVICE, new BinderService());
    publishLocalService(PowerManagerInternal.class, new LocalService());

    Watchdog.getInstance().addMonitor(this);
    Watchdog.getInstance().addThread(mHandler);
}

private final class BinderService extends IPowerManager.Stub ｛
    ......
｝

private final class LocalService extends PowerManagerInternal {
    ......
}
```
onStart()中发布了BinderService，LocalService分别供其他进程，进程内其他服务调用，并将PowerManagerService加入到Watchdog监控中。

`frameworks/base/services/core/java/com/android/server/power/PowerManagerService.java`
```
public void onBootPhase(int phase) {
    synchronized (mLock) {
        if (phase == PHASE_THIRD_PARTY_APPS_CAN_START) {
            ......
        } else if (phase == PHASE_BOOT_COMPLETED) {
            final long now = SystemClock.uptimeMillis();
            // 设置mBootCompleted状态
            mBootCompleted = true;
            mDirty |= DIRTY_BOOT_COMPLETED;
            // 更新userActivity及PowerState，后面分析
            userActivityNoUpdateLocked(
                    now, PowerManager.USER_ACTIVITY_EVENT_OTHER, 0, Process.SYSTEM_UID);
            updatePowerStateLocked();
            // 执行mBootCompletedRunnables中的runnable方法
            if (!ArrayUtils.isEmpty(mBootCompletedRunnables)) {
                Slog.d(TAG, "Posting " + mBootCompletedRunnables.length + " delayed runnables");
                for (Runnable r : mBootCompletedRunnables) {
                    BackgroundThread.getHandler().post(r);
                }
            }
            mBootCompletedRunnables = null;
        }
    }
}
```
onBootPhase中主要设置mBootCompleted状态，更新PowerState状态，并执行mBootCompletedRunnables中的runnables方法(低电量模式会设置)。
`frameworks/base/services/core/java/com/android/server/power/PowerManagerService.java`

```
public void systemReady(IAppOpsService appOps) {
    synchronized (mLock) {
        mSystemReady = true;
        // 获取AppOpsService
        mAppOps = appOps;
        // 获取DreamManager
        mDreamManager = getLocalService(DreamManagerInternal.class);
        // 获取DisplayManagerService
        mDisplayManagerInternal = getLocalService(DisplayManagerInternal.class);
        mPolicy = getLocalService(WindowManagerPolicy.class);
        // 获取mBatteryService
        mBatteryManagerInternal = getLocalService(BatteryManagerInternal.class);

        PowerManager pm = (PowerManager) mContext.getSystemService(Context.POWER_SERVICE);
        // 获取屏幕默认，最大，最小亮度
        mScreenBrightnessSettingMinimum = pm.getMinimumScreenBrightnessSetting();
        mScreenBrightnessSettingMaximum = pm.getMaximumScreenBrightnessSetting();
        mScreenBrightnessSettingDefault = pm.getDefaultScreenBrightnessSetting();
        // 获取SensorManager
        SensorManager sensorManager = new SystemSensorManager(mContext, mHandler.getLooper());

        mBatteryStats = BatteryStatsService.getService();
        // 创建Notifier对象，用于广播power state的变化
        mNotifier = new Notifier(Looper.getMainLooper(), mContext, mBatteryStats,
                mAppOps, createSuspendBlockerLocked("PowerManagerService.Broadcasts"),
                mPolicy);
        // 无线充电检测
        mWirelessChargerDetector = new WirelessChargerDetector(sensorManager,
                createSuspendBlockerLocked("PowerManagerService.WirelessChargerDetector"),
                mHandler);
        // 监听设置的变化
        mSettingsObserver = new SettingsObserver(mHandler);

        mLightsManager = getLocalService(LightsManager.class);
        mAttentionLight = mLightsManager.getLight(LightsManager.LIGHT_ID_ATTENTION);

        // Initialize display power management.
        mDisplayManagerInternal.initPowerManagement(
                mDisplayPowerCallbacks, mHandler, sensorManager);

        // Register for settings changes.
        final ContentResolver resolver = mContext.getContentResolver();
        resolver.registerContentObserver(Settings.Secure.getUriFor(
                Settings.Secure.SCREENSAVER_ENABLED),
        ......
        IVrManager vrManager =
                (IVrManager) getBinderService(VrManagerService.VR_MANAGER_BINDER_SERVICE);
        try {
            vrManager.registerListener(mVrStateCallbacks);
        } catch (RemoteException e) {
            Slog.e(TAG, "Failed to register VR mode state listener: " + e);
        }
        // 读取配置
        readConfigurationLocked();
        updateSettingsLocked();
        mDirty |= DIRTY_BATTERY_STATE;
        updatePowerStateLocked();
    }

    // Register for broadcasts from other components of the system.
    IntentFilter filter = new IntentFilter();
    filter.addAction(Intent.ACTION_BATTERY_CHANGED);
    filter.setPriority(IntentFilter.SYSTEM_HIGH_PRIORITY);
    mContext.registerReceiver(new BatteryReceiver(), filter, null, mHandler);

    filter = new IntentFilter();
    filter.addAction(Intent.ACTION_DREAMING_STARTED);
    filter.addAction(Intent.ACTION_DREAMING_STOPPED);
    mContext.registerReceiver(new DreamReceiver(), filter, null, mHandler);

    filter = new IntentFilter();
    filter.addAction(Intent.ACTION_USER_SWITCHED);
    mContext.registerReceiver(new UserSwitchedReceiver(), filter, null, mHandler);

    filter = new IntentFilter();
    filter.addAction(Intent.ACTION_DOCK_EVENT);
    mContext.registerReceiver(new DockReceiver(), filter, null, mHandler);
}
```

**userActivity**
userActivity是定义在PowerManager中的SystemApi，用户向PowerManagerService报告用户活动，以更新PowerManagerService内部时间/状态值，推迟系统休眠的时间。下面首先来看userActivity的定义。

` frameworks/base/core/java/android/os/PowerManager.java`
```
/**
 * User activity event type: Unspecified event type.
 */
public static final int USER_ACTIVITY_EVENT_OTHER = 0;

/**
 * User activity event type: Button or key pressed or released.
 */
public static final int USER_ACTIVITY_EVENT_BUTTON = 1;

/**
 * User activity event type: Touch down, move or up.
 */
public static final int USER_ACTIVITY_EVENT_TOUCH = 2;

/**
 * User activity event type: Accessibility taking action on behalf of user.
 */
public static final int USER_ACTIVITY_EVENT_ACCESSIBILITY = 3;

@SystemApi
public void userActivity(long when, int event, int flags) {
    try {
        mService.userActivity(when, event, flags);
    } catch (RemoteException e) {
        throw e.rethrowFromSystemServer();
    }
}
```
`frameworks/base/services/core/java/com/android/server/power/PowerManagerService.java`
```
private final class BinderService extends IPowerManager.Stub {
  ......
  public void userActivity(long eventTime, int event, int flags) {
      final long now = SystemClock.uptimeMillis();
      ......

      if (eventTime > now) {
          throw new IllegalArgumentException("event time must not be in the future");
      }

      final int uid = Binder.getCallingUid();
      final long ident = Binder.clearCallingIdentity();
      try {
          userActivityInternal(eventTime, event, flags, uid);
      } finally {
          Binder.restoreCallingIdentity(ident);
      }
  }
  ......
}
```
PowerManager中userActivity请求调用服务端PowerManagerService BinderService的userActivity，即调用内部方法userActivityInternal。

`frameworks/base/services/core/java/com/android/server/power/PowerManagerService.java`
```
private void userActivityInternal(long eventTime, int event, int flags, int uid) {
    synchronized (mLock) {
        if (userActivityNoUpdateLocked(eventTime, event, flags, uid)) {
            updatePowerStateLocked();
        }
    }
}
```
userActivityInternal中首先调用userActivityNoUpdateLocked更新相关数据及状态(***NoUpdateLocked仅仅更新内部状态并不采取任何操作)，然后调用updatePowerStateLocked更新所有PowerState，下面分析userActivityNoUpdateLocked的实现，updatePowerStateLocked是PowerManagerService的核心方法，在最后进行分析。

`frameworks/base/services/core/java/com/android/server/power/PowerManagerService.java`
```
private boolean userActivityNoUpdateLocked(long eventTime, int event, int flags, int uid) {
    // 如果发生时间是上一次休眠或唤醒前，或当前没有开机完成到systemReady，不采取操作直接返回
    if (eventTime < mLastSleepTime || eventTime < mLastWakeTime
            || !mBootCompleted || !mSystemReady) {
        return false;
    }

    try {
        // 更新mLastInteractivePowerHintTime时间
        if (eventTime > mLastInteractivePowerHintTime) {
            powerHintInternal(POWER_HINT_INTERACTION, 0);
            mLastInteractivePowerHintTime = eventTime;
        }

        // 通过mNotifier通知BatteryStats UserActivity事件
        mNotifier.onUserActivity(event, uid);

        if (mUserInactiveOverrideFromWindowManager) {
            mUserInactiveOverrideFromWindowManager = false;
            mOverriddenTimeout = -1;
        }

        // 如果系统处于休眠状态，不进行处理
        if (mWakefulness == WAKEFULNESS_ASLEEP
                || mWakefulness == WAKEFULNESS_DOZING
                || (flags & PowerManager.USER_ACTIVITY_FLAG_INDIRECT) != 0) {
            return false;
        }

        // 根据flag是否在已变暗的情况下是否重启活动超时更新mLastUserActivityTimeNoChangeLights或mLastUserActivityTime
        // 并且设置mDirty DIRTY_USER_ACTIVITY
        if ((flags & PowerManager.USER_ACTIVITY_FLAG_NO_CHANGE_LIGHTS) != 0) {
            if (eventTime > mLastUserActivityTimeNoChangeLights
                    && eventTime > mLastUserActivityTime) {
                mLastUserActivityTimeNoChangeLights = eventTime;
                mDirty |= DIRTY_USER_ACTIVITY;
                return true;
            }
        } else {
            if (eventTime > mLastUserActivityTime) {
                mLastUserActivityTime = eventTime;
                mDirty |= DIRTY_USER_ACTIVITY;
                return true;
            }
        }
    } finally {
        Trace.traceEnd(Trace.TRACE_TAG_POWER);
    }
    return false;
}
```
**gotoSleep**
gotoSleep在PowerManager中的定义如下：
`frameworks/base/core/java/android/os/PowerManager.java`
```
public void goToSleep(long time) {
    goToSleep(time, GO_TO_SLEEP_REASON_APPLICATION, 0);
}

public void goToSleep(long time, int reason, int flags) {
    try {
        mService.goToSleep(time, reason, flags);
    } catch (RemoteException e) {
        throw e.rethrowFromSystemServer();
    }
}
```
与userActivity一样，gotoSleep最终将调用到goToSleepInternal。
`frameworks/base/services/core/java/com/android/server/power/PowerManagerService.java
`
```
private final class BinderService extends IPowerManager.Stub {
  ......
  public void goToSleep(long eventTime, int reason, int flags) {
      if (eventTime > SystemClock.uptimeMillis()) {
          throw new IllegalArgumentException("event time must not be in the future");
      }

      mContext.enforceCallingOrSelfPermission(
              android.Manifest.permission.DEVICE_POWER, null);

      final int uid = Binder.getCallingUid();
      final long ident = Binder.clearCallingIdentity();
      try {
          goToSleepInternal(eventTime, reason, flags, uid);
      } finally {
          Binder.restoreCallingIdentity(ident);
      }
  }
  ......
}

private void goToSleepInternal(long eventTime, int reason, int flags, int uid) {
    synchronized (mLock) {
        if (goToSleepNoUpdateLocked(eventTime, reason, flags, uid)) {
            updatePowerStateLocked();
        }
    }
}
```
goToSleepInternal中将执行goToSleepNoUpdateLocked更新内部状态，同样在updatePowerStateLocked中更新PowerState的操作。
`frameworks/base/services/core/java/com/android/server/power/PowerManagerService.java`
```
private boolean goToSleepNoUpdateLocked(long eventTime, int reason, int flags, int uid) {
    // 当不处于awake状态或未开机systemReady，不处理
    if (eventTime < mLastWakeTime
            || mWakefulness == WAKEFULNESS_ASLEEP
            || mWakefulness == WAKEFULNESS_DOZING
            || !mBootCompleted || !mSystemReady) {
        return false;
    }

    try {
        ......
        // 更新mLastSleepTime时间，设置DIRTY_WAKEFULNESS标志位
        mLastSleepTime = eventTime;
        mSandmanSummoned = true;
        setWakefulnessLocked(WAKEFULNESS_DOZING, reason);

        // Report the number of wake locks that will be cleared by going to sleep.
        int numWakeLocksCleared = 0;
        final int numWakeLocks = mWakeLocks.size();
        for (int i = 0; i < numWakeLocks; i++) {
            final WakeLock wakeLock = mWakeLocks.get(i);
            switch (wakeLock.mFlags & PowerManager.WAKE_LOCK_LEVEL_MASK) {
                case PowerManager.FULL_WAKE_LOCK:
                case PowerManager.SCREEN_BRIGHT_WAKE_LOCK:
                case PowerManager.SCREEN_DIM_WAKE_LOCK:
                    numWakeLocksCleared += 1;
                    break;
            }
        }

        // Skip dozing if requested.
        if ((flags & PowerManager.GO_TO_SLEEP_FLAG_NO_DOZE) != 0) {
            reallyGoToSleepNoUpdateLocked(eventTime, uid);
        }
    } finally {
        Trace.traceEnd(Trace.TRACE_TAG_POWER);
    }
    return true;
}

```
goToSleepNoUpdateLocked中更新mLastSleepTime，mWakefulness，mDirty状态。



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

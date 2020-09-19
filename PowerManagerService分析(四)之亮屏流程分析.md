
 

###### 和您一起终身学习，这里是程序员Android 

**PowerManagerService** 之前系列文章请参考如下
1.[PowerManagerService分析(一)之PMS启动](https://mp.weixin.qq.com/s/Mgi1W9mmrCUPkASTp3LPrA)
2.[PowerManagerService分析(二)之updatePowerStateLocked()核心](https://mp.weixin.qq.com/s/P3IvBrYt7afEa4XyEd3BQg)
3.[PowerManagerService分析(三)之WakeLock机制](https://mp.weixin.qq.com/s/q3NLI_o3db7wRZ8ThKW3cQ)


>注： 本文转自网络，[原文地址](https://blog.csdn.net/FightFightFight/article/details/79808100)




![](https://upload-images.jianshu.io/upload_images/5851256-600dca83084c7424.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



本篇分析PMS中涉及到亮屏的部分，以及PMS相关的两个类：**PowerManager**和**Notifier**。

###1.亮屏流程

####1.1.Power键亮屏

这里直接从`PhoneWindowManager`开始分析。按`power`键后，会触发`PhoneWindowManager`的`interceptKeyBeforeQueueing()`方法：
```
@Override
    public int interceptKeyBeforeQueueing(KeyEvent event, int policyFlags) {
           .......
            case KeyEvent.KEYCODE_POWER: {
                cancelPendingAccessibilityShortcutAction();
                result &= ~ACTION_PASS_TO_USER;
                isWakeKey = false; // wake-up will be handled separately
                if (down) {
                    interceptPowerKeyDown(event, interactive);
                } else {
                    interceptPowerKeyUp(event, interactive, canceled);
                }
                break;
            }
            .......

```
在这个方法中，对`Power`键的按下和抬起做了处理，按下时，调用`interceptPowerKeyDown():`
```
private void interceptPowerKeyDown(KeyEvent event, boolean interactive) {
    // Hold a wake lock until the power key is released.
    if (!mPowerKeyWakeLock.isHeld()) {
       //1.申请一个唤醒锁，使CPU保持唤醒
        mPowerKeyWakeLock.acquire();
    }
   ......
if (!mPowerKeyHandled) {
       if (interactive) {
       ......
       } else {
    //2.进行亮屏处理
        wakeUpFromPowerKey(event.getDownTime());
    ........
        }
    }
}

```
在这个方法中，首先是申请了一个唤醒锁，然后会对一些特定功能进行处理，如截屏、结束通话，等等，然后如果此时处于非交互状态`(interactive=false)`，进行亮屏操作。该锁实例化如下：
```
mPowerKeyWakeLock = mPowerManager.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK,
            "PhoneWindowManager.mPowerKeyWakeLock");

```

申请锁流程在`PowerManagerService`分析第三篇[PowerManagerService分析(三)之WakeLock机制](https://mp.weixin.qq.com/s/q3NLI_o3db7wRZ8ThKW3cQ)已经分析过了。继续看`wakeUpFromPowerKey()`方法：
```
private void wakeUpFromPowerKey(long eventTime) {
    //第三个参数为亮屏原因，因此如果是power键亮屏，则log中会出现android.policy:POWER
    wakeUp(eventTime, mAllowTheaterModeWakeFromPowerKey, "android.policy:POWER");
}

private boolean wakeUp(long wakeTime, boolean wakeInTheaterMode, String reason) {
    final boolean theaterModeEnabled = isTheaterModeEnabled();
    if (!wakeInTheaterMode && theaterModeEnabled) {
        return false;
    }
    if (theaterModeEnabled) {
        Settings.Global.putInt(mContext.getContentResolver(),
                Settings.Global.THEATER_MODE_ON, 0);
    }
    mPowerManager.wakeUp(wakeTime, reason);
    return true;
}

```
在这个方法中，首先判断是否允许在剧院模式下点亮屏幕(这个模式不常用，未进行详细分析),之后通过`PowerManager`在`PMS`进行屏幕的唤醒，先来看看`PowerManager`的`wakeup（）`方法：
```
public void wakeUp(long time) {
    try {
        mService.wakeUp(time, "wakeUp", mContext.getOpPackageName());
    } catch (RemoteException e) {
        throw e.rethrowFromSystemServer();
    }
}

```
现在到`PMS`中继续分析流程：
```
@Override // Binder call
public void wakeUp(long eventTime, String reason, String opPackageName) {
    if (eventTime > SystemClock.uptimeMillis()) {
        throw new IllegalArgumentException("event time must not be in the future");
    }
    //权限检查
    mContext.enforceCallingOrSelfPermission(
            android.Manifest.permission.DEVICE_POWER, null);
    final int uid = Binder.getCallingUid();
    //清除IPC标志
    final long ident = Binder.clearCallingIdentity();
    try {
        //调用内部方法
        wakeUpInternal(eventTime, reason, uid, opPackageName, uid);
    } finally {
        //重置IPC标志
        Binder.restoreCallingIdentity(ident);
    }
}

```
在`PMS`中暴露给`Binder`客户端的方法中，进行了权限的检查，然后调用`wakeUpInternal()`方法,该方法如下：

```
private void wakeUpInternal(long eventTime, String reason, int uid, String opPackageName,
        int opUid) {
    synchronized (mLock) {
        if (wakeUpNoUpdateLocked(eventTime, reason, uid, opPackageName, opUid)) {
            updatePowerStateLocked();
        }
    }
}

```
这里又调用了`wakeUpNoUpdateLocked()`方法，如果这个方法返回`true`，则会执行`updatePowerStateLocked()`方法，如果返回`false`，则整个过程结束。这个方法在我们分析wakelock申请时提到过，如果申请的`wakelock`锁带有唤醒屏幕的标志，也只执行这个方法，因此，这个方法是唤醒屏幕的主要方法之一，来看看这个方法：
```
private boolean wakeUpNoUpdateLocked(long eventTime, String reason, int reasonUid,
        String opPackageName, int opUid) {
    if (eventTime < mLastSleepTime || mWakefulness == WAKEFULNESS_AWAKE
            || !mBootCompleted || !mSystemReady) {
        return false;
    }
    try {
	//根据当前wakefulness状态打印log，这些log很有用
        switch (mWakefulness) {
            case WAKEFULNESS_ASLEEP:
                Slog.i(TAG, "Waking up from sleep (uid " + reasonUid +")...");
                break;
            case WAKEFULNESS_DREAMING:
                Slog.i(TAG, "Waking up from dream (uid " + reasonUid +")...");
                break;
            case WAKEFULNESS_DOZING:
                Slog.i(TAG, "Waking up from dozing (uid " + reasonUid +")...");
                break;
        }
        //设置最后一次亮屏时间，即该次的时间
        mLastWakeTime = eventTime;
        //设置wakefulness为WAKEFULNESS_AWAKE
        setWakefulnessLocked(WAKEFULNESS_AWAKE, 0);
        //Notifier中通知BatteryStatsService统计亮屏
        mNotifier.onWakeUp(reason, reasonUid, opPackageName, opUid);
        //更新用户活动时间
        userActivityNoUpdateLocked(
                eventTime, PowerManager.USER_ACTIVITY_EVENT_OTHER, 0, reasonUid);
    } finally {
    }
    return true;
}

```
在这个方法中，`Log`中的`reason`需要注意一下：

- `Power`键亮屏:
则reason是PWM中传入的android.policy:POWER；
- 来电亮屏:
android.server.am:TURN_ON;
- USB插拔时:
android.server.power:POWER

所以不管是哪种亮屏方式，最终都会在这里汇合的。之后通过`setWakefulnessLocked()`方法设置`wakefulness`，再通过`Notifier`进行处理和通知其他系统服务`wakefulness`的改变，最后更新用户活动的时间，重置下次超时灭屏时间。继续看看`setWakefulnessLocked():`
```
void setWakefulnessLocked(int wakefulness, int reason) {
       if (mWakefulness != wakefulness) {
       //改变Wakefulness
        mWakefulness = wakefulness;
        mWakefulnessChanging = true;
        //置位操作
        mDirty |= DIRTY_WAKEFULNESS;
        if (mNotifier != null) {
            //处理wakefulness改变前的操作
            mNotifier.onWakefulnessChangeStarted(wakefulness, reason);
        }
    }
}

```
首先，改变当前`mWakefulness`值，将`mWakefulnessChanging`标记为`true`，将`mWakefulness`值标志为`DIRTY_WAKEFULNESS`，然后通过`Notifier`进行改变`wakefulness`之前的一些处理，`Notifier`负责`PMS`和其他系统服务的交互。而`Notifier`中的`onWakefulnessChangeStarted()`方法，就是亮屏的主要方法之一，如发送亮屏或者灭屏的广播等。

为了不显得凌乱，将关于`Notifier`的具体细节的分析放在了3小节中。这里对其大概的进行下阐述。`onWakefulnessChangeStarted()`方法部分如下：
```
public void onWakefulnessChangeStarted(final int wakefulness, int reason) {
        final boolean interactive = PowerManagerInternal.isInteractive(wakefulness);
        if (DEBUG) {
            Slog.d(TAG, "onWakefulnessChangeStarted: wakefulness=" + wakefulness
                    + ", reason=" + reason + ", interactive=" + interactive);
        }
        // Handle any early interactive state changes.
        // Finish pending incomplete ones from a previous cycle.
        if (mInteractive != interactive) {
            // Finish up late behaviors if needed.
            if (mInteractiveChanging) {
                handleLateInteractiveChange();
            }
            // Handle early behaviors.
            mInteractive = interactive;
            mInteractiveChangeReason = reason;
            mInteractiveChanging = true;
            //做亮屏早期工作
            handleEarlyInteractiveChange();
        }
    }

```
**亮屏操作**，此时部分变量值为：
`interactive为true;
mInteractive为false；
mInteractiveChanging = true；`
因此会开始执行`handleEarlyInteractiveChange()`方法，做一些亮屏前期的工作，该方法亮屏部分如下：
```
private void handleEarlyInteractiveChange() {
        synchronized (mLock) {
            if (mInteractive) {
                // Waking up...
                mHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        //回调PhoneWindowManager
                        mPolicy.startedWakingUp();
                    }
                });
                // Send interactive broadcast.
                mPendingInteractiveState = INTERACTIVE_STATE_AWAKE;
                mPendingWakeUpBroadcast = true;
                //发送亮/灭屏广播
                updatePendingBroadcastLocked();
            } else {
                // Going to sleep...
            }
        }
    }

```
首先，会回调`PhoneWindowManager`中的`startedWakingUp()`，然后发送亮屏广播。在`startedWakingUp()`中的工作如下：
```
    @Override
    public void startedWakingUp() {
        if (DEBUG_WAKEUP) Slog.i(TAG, "Started waking up...");
        synchronized (mLock) {
            mAwake = true;
            updateWakeGestureListenerLp();
            updateOrientationListenerLp();
            updateLockScreenTimeout();
        }
        if (mKeyguardDelegate != null) {
            mKeyguardDelegate.onStartedWakingUp();
        }
    }

```
继续分析`PMS`中的剩余流程，`setWakefulnessLocked()`执行完毕后，接下来执行的是`Notifier`的`onWakeUp()`方法,这个方法负责和`BatteryStatsService`、`AppService`进行交互，将`wakeup`信息传递给它们，如下：
```
public void onWakeUp(String reason, int reasonUid, String opPackageName, int opUid) {
    try {
        //开始统计亮屏时间
        mBatteryStats.noteWakeUp(reason, reasonUid);
        if (opPackageName != null) {
            mAppOps.noteOperation(AppOpsManager.OP_TURN_SCREEN_ON, 
                opUid, opPackageName);
        }
    } catch (RemoteException ex) {
    }
}

```
接下来，执行`userActivityNoUpdateLocked()`方法，这个方法任务只有一个,负责更新系统和用户最后交互时间，计算的时间在`updateUserActivitySummary()`方法中会用于判断何时灭屏,该方法如下：

```
private boolean userActivityNoUpdateLocked(long eventTime, int event, int flags, int uid) {

        // ......
       
        /**
         * USER_ACTIVITY_FLAG_NO_CHANGE_LIGHTS标识
         * 只有带有PowerManager.ON_AFTER_RELEASE类型的锁在释放时才会有该flag，在亮屏流程中没有该标识，因此不满足该条
         * 件，如果满足条件，改变mLastUserActivityTimeNoChangeLights的值，否则进入else语句，改变
         * mLastUserActivityTime的值
         */
        if ((flags & PowerManager.USER_ACTIVITY_FLAG_NO
                  _CHANGE_LIGHTS) != 0) {
            if (eventTime > mLastUserActivityTimeNoChangeLights
                    && eventTime > mLastUserActivityTime) {
                mLastUserActivityTimeNoChangeLights = eventTime;
                mDirty |= DIRTY_USER_ACTIVITY;
                if (event == PowerManager.USER_ACTIVITY_EVENT_BUTTON) {
                    mDirty |= DIRTY_QUIESCENT;
                }
                return true;
            }
        } else {
            if (eventTime > mLastUserActivityTime) {
                mLastUserActivityTime = eventTime;
                mDirty |= DIRTY_USER_ACTIVITY;
                if (event == PowerManager.USER_ACTIVITY_EVENT_BUTTON) {
                    mDirty |= DIRTY_QUIESCENT;
                }
                return true;
            }
        }
    // ......
    return false;
}

```
在这个方法中来看下`PowerManager.USER_ACTIVITY_FLAG_NO_CHANGE_LIGHT`这个值，这个值和释放`wakelock`有关系，在分析`WakeLock`释放流程时分析到，如果带有`PowerManager.ON_AFTER_RELEASE`标记，则在释放该`WakeLock`时会先亮一小会之后才会灭屏，这里正是为何会亮一小会才会灭屏的关键。我们可以在释放`WakeLock`锁的流程方法中看到：
```
private void applyWakeLockFlagsOnReleaseLocked(WakeLock wakeLock) {
    if ((wakeLock.mFlags & PowerManager.ON_AFTER_RELEASE) != 0
            && isScreenLock(wakeLock)) {
        userActivityNoUpdateLocked(SystemClock.uptimeMillis(),
                PowerManager.USER_ACTIVITY_EVENT_OTHER,
                PowerManager.USER_ACTIVITY_FLAG_NO_CHANGE_LIGHTS,
                wakeLock.mOwnerUid);
    }
}

```
因此，如果是释放带有`ON_AFTER_REALESE`标记的锁，则会给该方法传入`USER_ACTIVITY_FLAG_NO_CHANGE_LIGHT`标记，其他场景下不会带有该标识。

这些方法执行完后,执行`updatePowerStateLocked()`方法更新所有信息，这个方法作为PMS的核心方法，在`PowerManagerService`分析第二篇[PowerManagerService分析(二)之updatePowerStateLocked()核心](https://mp.weixin.qq.com/s/P3IvBrYt7afEa4XyEd3BQg)中分析过了，同时在这个方法中会进行亮屏的最关键的操作:`updateDisplayPowerStateLocked()`，这里对`updateDisplayPowerStateLocked()`方法进行分析：

```
  private boolean updateDisplayPowerStateLocked(int dirty) {
        final boolean oldDisplayReady = mDisplayReady;
	       //策略值，请求Display的重要属性
            mDisplayPowerRequest.policy = getDesiredScreenPolicyLocked();
            //确定亮度值部分省略......
            
            //亮度值
            mDisplayPowerRequest.screenBrightness = screenBrightness;
            //自动调节亮度比例值
            mDisplayPowerRequest.screenAutoBrightnessAdjustment =
                    screenAutoBrightnessAdjustment;
            //是否是用户设置亮度
            mDisplayPowerRequest.brightnessSetByUser = brightnessSetByUser;
            //是否使用自动调节亮度
            mDisplayPowerRequest.useAutoBrightness = autoBrightness;
            //是否使用PSensor
            mDisplayPowerRequest.useProximitySensor = shouldUseProximitySensorLocked();
            //是否进行亮度增强
            mDisplayPowerRequest.boostScreenBrightness = shouldBoostScreenBrightness();
            //低电量模式一些值的设置
            updatePowerRequestFromBatterySaverPolicy(mDisplayPowerRequest);
            //Doze状态下相关设置值
            if (mDisplayPowerRequest.policy == DisplayPowerRequest.POLICY_DOZE) {
                mDisplayPowerRequest.dozeScreenState = mDozeScreenStateOverrideFromDreamManager;
                if (mDisplayPowerRequest.dozeScreenState == Display.STATE_DOZE_SUSPEND
                        && (mWakeLockSummary & WAKE_LOCK_DRAW) != 0) {
                    mDisplayPowerRequest.dozeScreenState = Display.STATE_DOZE;
                }
                mDisplayPowerRequest.dozeScreenBrightness =
                        mDozeScreenBrightnessOverrideFromDreamManager;
            } else {
                mDisplayPowerRequest.dozeScreenState = Display.STATE_UNKNOWN;
                mDisplayPowerRequest.dozeScreenBrightness = PowerManager.BRIGHTNESS_DEFAULT;
            }
            //通过DMS本地服务DisplayManagerInternal请求DisplayManagerService
            mDisplayReady = mDisplayManagerInternal.requestPowerState(mDisplayPowerRequest,
                    mRequestWaitForNegativeProximity);
            mRequestWaitForNegativeProximity = false;

            if ((dirty & DIRTY_QUIESCENT) != 0) {
                sQuiescent = false;
            }
           
        }
        //mDisplayReady表示请求的新的显示是否完成
        return mDisplayReady && !oldDisplayReady;
    }

```
这个方法中，将`PMS`中和`Display`相关的值都封装在了`DisplayPowerRequest`中，然后向`DisplayManagerService`请求新的状态，`DisplayManagerService`会交给`DisplayPowerController`去处理请求。
在请求时，`DisplayPowerRequest.policy`作为`DisplayPowerRequset`的属性，有四种值，分别为`off、doze、dim、bright、vr`。在向`DisplayManagerService`请求时，会根据当前PMS中的唤醒状态和统计的`wakelock`来决定`policy`值，从而确定要请求的`Display`状态，这部分源码如下：

```
@VisibleForTesting
int getDesiredScreenPolicyLocked() {
    //asleep时，policy值为0
    if (mWakefulness == WAKEFULNESS_ASLEEP || sQuiescent) {
        return DisplayPowerRequest.POLICY_OFF;//0
    }
    if (mWakefulness == WAKEFULNESS_DOZING) {
        if ((mWakeLockSummary & WAKE_LOCK_DOZE) != 0) {
            return DisplayPowerRequest.POLICY_DOZE;//1
        }
        if (mDozeAfterScreenOffConfig) {
            return DisplayPowerRequest.POLICY_OFF;
        }
    }    
    if ((mWakeLockSummary & WAKE_LOCK_SCREEN_BRIGHT) != 0
            || (mUserActivitySummary & USER_ACTIVITY_SCREEN_BRIGHT) != 0
            || !mBootCompleted
            || mScreenBrightnessBoostInProgress) {
        return DisplayPowerRequest.POLICY_BRIGHT;//3
    }

    return DisplayPowerRequest.POLICY_DIM;
}

```
这里对`DisplayPowerRequest.policy`进行下说明，在该方法中,由于是亮屏操作，所以此时`mWakefulness=WAKEFULNESS_AWAKE(1)`，综合判断条件，所以policy的值为`POLICY_BRIGHT`。同样地，如果是灭屏操作的话，`policy`值就是`POLICY_OFF`，还有一种情况，如果系统配置了`dozeComponent`组件，那么在按`Power`键灭屏时，不会立即进入`Sleep`装填，而是会先进入`Doze`状态，所以这种情况下`policy`值为`POLICY_DOZE`。

现在回到`updateDisplayPowerStateLocked()`方法中，再来看看当“请求体”确定后，是如何进行请求的。`PMS`中向`DisplayManagerService`发起请求后，`DisplayManagerService`交给了`DisplayController`中进行处理，这个过程图示如下：

![](https://upload-images.jianshu.io/upload_images/5851256-4dd6b37a18fd07fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
对应相关代码如下(有删减)：
`DisplayController.java:`
```

public boolean requestPowerState(DisplayPowerRequest request,
        boolean waitForNegativeProximity) {
    synchronized (mLock) {
        boolean changed = false;
        //开机后第一次进入
        if (mPendingRequestLocked == null) {
            mPendingRequestLocked = new DisplayPowerRequest(request);
            changed = true;
         //如果该次请求和上次请求不同，说明已经改变，需要更新Display
        } else if (!mPendingRequestLocked.equals(request)) {
            mPendingRequestLocked.copyFrom(request);
            changed = true;
        }
        /**
         * changed为true,说明有改变发生,这个改变交给Handler异步去处理，此时说
         * 明显示没有准备好，mDisplayReadyLocked=false
         * 直到改变处理成功，mDisplayReadyLocked又被置为true，
         */
        if (changed) {
            mDisplayReadyLocked = false;
        }
        //mPendingRequestChangedLocked：用于标识电源请求状态或者PSensor标签是
        //否改变
        if (changed && !mPendingRequestChangedLocked) {
            mPendingRequestChangedLocked = true;
            sendUpdatePowerStateLocked();
        }
        return mDisplayReadyLocked;
    }
}

```
在这个方法中，会判断请求时携带的`DisplayPowerRequest`对象是否和上一次发生请求的`DisplayRequest`对象相同，如果相同，则返回值`mDisplayReadyLocked为true`，如果不同，则表示发生了改变，此时会异步去请求新的Display状态，并向PMS返回false，表示Display状态正在更新，没有准备完成，直到更新完成后，会将`mDisplayReadyLocked`值置为true，表示更新完成。同时回调PMS中的`onStateChanged（）`方法通知PMS更新完成。这个返回值会在PMS中作为下一步的执行条件。

处理完成后回调PMS中的`onStateChanged()`方法通知PMS，最终完成Display的更新。关于`DisplayPowerController`和`DisplayManagerService`以及其他模块中如何处理的，这里暂不分析。只需要知道当`DisplayPowerController`处理完请求后，回调`DisplayManagerInternal.DisplayPowerCallbacks`的`onStateChanged()`方法，再来看看这个方法：
```
private final DisplayManagerInternal.DisplayPowerCallbacks mDisplayPowerCallbacks =
            new DisplayManagerInternal.DisplayPowerCallbacks() {
        private int mDisplayState = Display.STATE_UNKNOWN;

        @Override
        public void onStateChanged() {
            synchronized (mLock) {
                mDirty |= DIRTY_ACTUAL_DISPLAY_POWER_STATE_UPDATED;
                updatePowerStateLocked();
            }
        }
        .........
   }

```
在这个方法中，对`mDirty`进行了置位操作，然后由调用了`updatePowerState()`方法，这次调用时状态已经没有发生过改变，因此会根据执行条件而在某一阶段跳出。
当请求完毕后，对于`updatePowerStateLocked()`方法中剩余的其他三个方法主要作用是和屏保、`wakefulness`改变的收尾工作和申请/释放`SuspendBlocker`锁，这里略去。
关于按power键亮屏PMS相关部分就分析完了，当然，亮屏完整流程涉及模块较多，绝非几句就可以说完，这里仅仅将PMS相关的进行分析，至于其他模块如`DMS、LightsService`中的，会在下篇文章中进行分析。整个Power键亮屏时序图如下：
![](https://upload-images.jianshu.io/upload_images/5851256-70f49c19c047a9da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
PMS发起请求后其他模块的处理时序图：
![](https://upload-images.jianshu.io/upload_images/5851256-312db203aa5321cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###1.2.插拔USB亮屏

当插拔USB时，会发送`BATTERY_CHANGED`广播，`PMS`中对该广播进行监听，如果收到广播后，配置了插播USB时亮屏，则会进行亮屏操作。
在`BatteryService`中，如果电池状态发生改变，则会发送一个`ACTION_Battery_CAHNGED`广播：
```
private void sendIntentLocked() {
    //  Pack up the values and broadcast them to everyone
    final Intent intent = new Intent(Intent.ACTION_BATTERY_CHANGED);
    intent.addFlags(Intent.FLAG_RECEIVER_REGISTERED_ONLY
            | Intent.FLAG_RECEIVER_REPLACE_PENDING);
    ....
    mHandler.post(new Runnable() {
        @Override
        public void run() {
            ActivityManager.broadcastStickyIntent(intent, UserHandle.USER_ALL);
        }
    });
}

```
在PMS中，注册了广播接受者，会接收该广播：
```
public void systemReady(IAppOpsService appOps) {
     ....
IntentFilter filter = new IntentFilter();
    filter.addAction(Intent.ACTION_BATTERY_CHANGED);
    filter.setPriority(IntentFilter.SYSTEM_HIGH_PRIORITY);
    mContext.registerReceiver(new BatteryReceiver(), filter, null, mHandler);
    ......
}

```
因此当`BatteryService`中检测到底层电池状态发生变化后，会发送该广播，PMS中的`BatteryReceiver`用于接受该广播并进行处理，如下：
```
private final class BatteryReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        synchronized (mLock) {
            handleBatteryStateChangedLocked();
        }
    }
}

```
继续看看：
```
private void handleBatteryStateChangedLocked() {
    mDirty |= DIRTY_BATTERY_STATE;
    updatePowerStateLocked();
}

```
在这里对`mDirty`进行了置位，之后开始调用`updatePowerStateLocked()`方法。在之前已经分析过该方法了，其中调用的`updateIsPowerLocked()`方法之前分析过了，是插播USB亮屏的入口方法，所有和电池相关的都是在这里处理。这里看看具体是如何进行亮屏的：
```
private void updateIsPoweredLocked(int dirty) {
    if ((dirty & DIRTY_BATTERY_STATE) != 0) {
           ........
            if (shouldWakeUpWhenPluggedOrUnpluggedLocked(wasPowered, 
           oldPlugType, dockedOnWirelessCharger)) {
                wakeUpNoUpdateLocked(now, "android.server.power:POWER", 
                         Process.SYSTEM_UID,mContext.getOpPackageName(), 
                         Process.SYSTEM_UID);
            }
        }
    }
}

```
因此，如果`shouldWakeUpWhenPluggedOrUnpluggedLocked()`方法返回true，则会开始亮屏，否则不会亮屏。该方法如下：
```
private boolean shouldWakeUpWhenPluggedOrUnpluggedLocked(
        boolean wasPowered, int oldPlugType, boolean dockedOnWirelessCharger) {
    // 如果配置config_unplugTurnsOnScreen为false,则不会亮屏
    if (!mWakeUpWhenPluggedOrUnpluggedConfig) {
        return false;
    }
    //断开无线充电不会亮屏
    if (wasPowered && !mIsPowered
            && oldPlugType == BatteryManager.BATTERY_PLUGGED_WIRELESS) {
        return false;
    }
    //连接无线充电不会亮屏
    if (!wasPowered && mIsPowered
            && mPlugType == BatteryManager.BATTERY_PLUGGED_WIRELESS
            && !dockedOnWirelessCharger) {
        return false;
    }
    // 插入充电时屏保状态下不会亮屏
    if (mIsPowered && mWakefulness == WAKEFULNESS_DREAMING) {
        return false;
    }
    //剧院模式下，且配置了剧院模式下充电是否亮屏为false，则不会亮屏
    if (mTheaterModeEnabled && !mWakeUpWhenPluggedOrUnpluggedInTheaterModeConfig) {
        return false;
    }
    //如果屏幕保持常亮且当前wakefulness为Doze状态
    if (mAlwaysOnEnabled && mWakefulness == WAKEFULNESS_DOZING) {
        return false;
    }
    // 其他情况下则亮屏
    return true;
}

```
当该方法返回true后，开始调用`wakeUpNoUpdateLocked()`方法开始唤醒屏幕，该方法是唤醒屏幕的入口方法，在前面有详细的分析。

#### 2.PowerManager类

`PowerManager`可以说是`PMS`向`Application`层提供信息的一个接口。`PowerManager`用来控制设备电源状态。在上面分析了`PMS`，是一个系统服务，由`SystemServer`启动并运行，并没有提供上层调用的接口，因此呢，`PowerManager`作为PMS的一个代理类，向上层应用层提供开放接口，供Application层调用，实现对电源的管理，其实现原理和上文谈到的`Binider`注册有关。
`PowerManager`作为系统级别服务，在获取其实例时，通过以下方式进行获取：
```
PowerManager pm = 
          (PowerManager) mContext.getSystemService(Context.POWER_SERVICE);

```
通过`Context.POWER_SERVICE`，获取了`PowerManager`实例，而这个字段在`PMS`进行`Binder`注册的时候使用了，因此，实际上`PowerManager`对象中包含了一个`PMS.BindService`对象，当应用层调用`PowerManager`开放接口后，`PowerManager`再通过了`PMS.BindService`向下调用到了`PMS`中。这点可以在`PowerManager`的构造方法中看出：

```
public PowerManager(Context context, IPowerManager service, Handler handler) {
    mContext = context;
    mService = service;
    mHandler = handler;
}

```
在`PowerManager`中，提供了许多`public`方法，当应用层调用这些方法时，`PowerManager`将向下调用`PMS`。
####3.Notifier类
`Notifier`类好比`PMS`和其他系统服务交互的”中介“，`Notifier`和`PMS`在结构上可以说是组合关系，`PMS`中需要和其他组件交互的大部分都由`Notifier`处理，如亮灭屏通知其他服务等,亮灭屏广播也是在该类中发出。这里介绍其中的部分方法，有些可能已经在上面内容的分析中涉及到了。(注：如果是第一次看到这部分内容，那么应该先看看之前的内容。)
####3.1.onWakefulnessChangeStarted()
该方法用于亮屏或者灭屏时逻辑的处理，和`onWakefulnessChangeFinished()`方法对应，分别负责操作开始和结束的逻辑处理，当`wakefulness`改变时进行回调，因此当亮屏、灭屏、进入`Doze`模式时都会调用这个方法，看看这个方法：
```
public void onWakefulnessChangeStarted(final int wakefulness, int reason) {

//是否可以和用户进行交互
//既wakefulness == WAKEFULNESS_AWAKE || WAKEFULNESS_DREAM
    final boolean interactive = PowerManagerInternal.isInteractive(wakefulness);
    mHandler.post(new Runnable() {
        @Override
        public void run() {
        //和AMS交互，通知AMS wakefulness发生改变
            mActivityManagerInternal.onWakefulnessChanged(wakefulness);
        }
    });
  //如果为false，表示交互状态发生改变，即从亮屏到灭屏或者从灭屏到亮屏
      if (mInteractive != interactive) {
        // Finish up late behaviors if needed.
    //交互状态发生了改变
        if (mInteractiveChanging) {
            handleLateInteractiveChange();//处理交互改变后的任务
        }
        // Start input as soon as we start waking up or going to sleep.
    //和IMS交互
        mInputManagerInternal.setInteractive(interactive);
        mInputMethodManagerInternal.setInteractive(interactive);
        //和BatteryStatsService交互
        try {
            mBatteryStats.noteInteractive(interactive);
        } catch (RemoteException ex) { }
        //处理交互完成前的操作
        mInteractive = interactive;
        mInteractiveChangeReason = reason;
        mInteractiveChanging = true;
        handleEarlyInteractiveChange();
    }
}

```
首先判断系统是否可以进行交互，如果处于`Dream`或者`Awake`状态，表示可以进行交互，`interactive为true`;在这个方法中有两个关键方法，`handleLateInteractiveChange()`和`handleEarlyInteractiveChange(),`分别表示处理交互状态改变后的操作和改变前的操作，如果是亮屏场景，则在执行到该方法时，在`setWakeFulnessLocked()`方法中将`wakefulness`设置为了`WAKEFULNESS_AWAKE`,所以`interactive为true，mInteractive是false`，因此会先执行`handleEarlyInteractiveChange()`,继续看看`handleEarlyInteractiveChange()`方法：
```
//mScreenOnIntent = new Intent(Intent.ACTION_SCREEN_ON);
private void sendWakeUpBroadcast() {
    if (DEBUG) {
        Slog.d(TAG, "Sending wake up broadcast.");
    }

    if (mActivityManagerInternal.isSystemReady()) {
        mContext.sendOrderedBroadcastAsUser(mScreenOnIntent, 
                UserHandle.ALL, null,
                mWakeUpBroadcastDone, mHandler, 0, null, null);
    } else {
        EventLog.writeEvent(EventLogTags.POWER_SCREEN_BROADCAST_STOP, 2, 
                                          1);
        sendNextBroadcast();
    }
}

```
####3.2.onWakefulnessChangeFinished()

该方法负责`wakefulness`状态改变完成后的工作，和3.1方法相对应。这个方法较简单，当`PMS`中调用它后，它会调用`handleLaterInteractiveChanged()`方法,如下：
```
/**
 * Notifies that the device has finished changing wakefulness.
 */
public void onWakefulnessChangeFinished() {
    if (mInteractiveChanging) {
        mInteractiveChanging = false;
        handleLateInteractiveChange();
    }
}

```
继续：
```
private void handleLateInteractiveChange() {
    synchronized (mLock) {
        //mInteractive在onWakefulnessCHangeStated()中进行了改变，以用来确定是否
        //是亮屏或者灭屏
        if (mInteractive) {
           //亮屏...
            mHandler.post(new Runnable() {
                @Override
                public void run() {
                    mPolicy.finishedWakingUp();
                }
            });
        } else {
            //灭屏...
            if (mUserActivityPending) {
                mUserActivityPending = false;
                mHandler.removeMessages(MSG_USER_ACTIVITY);
            }
            final int why = translateOffReason(mInteractiveChangeReason);
            mHandler.post(new Runnable() {
                @Override
                public void run() {
                    LogMaker log = new LogMaker(MetricsEvent.SCREEN);
                    log.setType(MetricsEvent.TYPE_CLOSE);
                    log.setSubtype(why);
                    MetricsLogger.action(log);
                    EventLogTags.writePowerScreenState(0, why, 0, 0, 0);
                    mPolicy.finishedGoingToSleep(why);
                }
            });
            // 发送完成后的广播
            mPendingInteractiveState = INTERACTIVE_STATE_ASLEEP;
            mPendingGoToSleepBroadcast = true;
            updatePendingBroadcastLocked();
        }
    }
}

```
在这个方法中，如果是亮屏，则调用`PWM`的`finishedWakingUp()`表示亮屏处理成功，如果是灭屏，则调用`PWM`的`finishedGoingToSleep()`
####3.3.updatePendingBroadcaseLocked()
这个方法用于交互状态改变时发送广播，最常见的就是由亮屏-灭屏之间的改变了，都会发送这个广播。亮屏时，在`handlerEarlyInteractiveChang()`方法中调用该方法发送广播，灭屏时，在`handlerLateInteractiveChang()`中调用方法发送广播。接下来会分两种情况进行分析。
当系统由不可交互变成可交互时，如由灭屏-亮屏，首先做了如下处理：
```
// Send interactive broadcast.
mPendingInteractiveState = INTERACTIVE_STATE_AWAKE;
mPendingWakeUpBroadcast = true;
updatePendingBroadcastLocked();

```
现在进入这个方法进行分析：
```
private void updatePendingBroadcastLocked() {
    /**
     * 广播没有进行中&&要发送的广播状态！= UNKNOW
     * && (发送亮屏广播||发送灭屏广播||发送广播状态！=当前广播交互状态）
     * mBroadcastedInteractiveState值实际上是上次发送广播交互状态的值
     */
    if (!mBroadcastInProgress
            && mPendingInteractiveState != INTERACTIVE_STATE_UNKNOWN
            && (mPendingWakeUpBroadcast || mPendingGoToSleepBroadcast
                    || mPendingInteractiveState != mBroadcastedInteractiveState)) {
        mBroadcastInProgress = true;
        //申请一个Suspend锁，以防广播发送未完成系统休眠而失败
        mSuspendBlocker.acquire();
        Message msg = mHandler.obtainMessage(MSG_BROADCAST);
        msg.setAsynchronous(true);
        mHandler.sendMessage(msg);
    }
}

```
在这个方法中，使用到的几个属性值意义如下：
```
//要广播的交互状态
private int mPendingInteractiveState;
//是否广播亮屏
private boolean mPendingWakeUpBroadcast;
//是否广播灭屏
private boolean mPendingGoToSleepBroadcast;
//当前要广播的交互状态
private int mBroadcastedInteractiveState;
//是否广播正在进行中
private boolean mBroadcastInProgress;

```
在这个方法中，首先申请了一个`suspend`锁，这个锁是通过在`PMS`中创建`Notifier`对象时创建传入的，`name`为`PowerManagerService.Broadcast`在广播发送完成后又进行了释放，这样作的目的是避免在发送广播过程中系统休眠而导致广播未发送完成；可以通过如下命令查看该锁：
```
adb root
adb remount
adb shell
cat /sys/power/wake_lock

```
之后通过`Handler`中调用`sendNextBroadcaset()`方法发送广播，看看这个方法：
```
private void sendNextBroadcast() {
    final int powerState;
    synchronized (mLock) {
        //当前广播的交互状态=0（成员变量默认0）
        if (mBroadcastedInteractiveState == INTERACTIVE_STATE_UNKNOWN) {
            // Broadcasted power state is unknown.  Send wake up.
            mPendingWakeUpBroadcast = false;
            mBroadcastedInteractiveState = INTERACTIVE_STATE_AWAKE;
            //当前广播的交互状态为亮屏
        } else if (mBroadcastedInteractiveState == INTERACTIVE_STATE_AWAKE) {
            // Broadcasted power state is awake.  Send asleep if needed.
            //广播亮屏||广播灭屏||最终要广播的交互状态为灭屏
            if (mPendingWakeUpBroadcast || mPendingGoToSleepBroadcast
                    || mPendingInteractiveState == INTERACTIVE_STATE_ASLEEP) {
                mPendingGoToSleepBroadcast = false;
                mBroadcastedInteractiveState = INTERACTIVE_STATE_ASLEEP;
            } else {
                finishPendingBroadcastLocked();
                return;
            }
            //当前广播的交互状态为灭屏
        } else {
            if (mPendingWakeUpBroadcast || mPendingGoToSleepBroadcast
                    || mPendingInteractiveState == INTERACTIVE_STATE_AWAKE) {
                mPendingWakeUpBroadcast = false;
                mBroadcastedInteractiveState = INTERACTIVE_STATE_AWAKE;
            } else {
                finishPendingBroadcastLocked();
                return;
            }
        }
        mBroadcastStartTime = SystemClock.uptimeMillis();
        powerState = mBroadcastedInteractiveState;
    }
    if (powerState == INTERACTIVE_STATE_AWAKE) {
        //发送亮屏广播
        sendWakeUpBroadcast();
    } else {
        //发送灭屏广播
        sendGoToSleepBroadcast();
    }
}

```
在这里总结下该方法的判断条件：

- 1.如果`mBroadcastedInteractiveState`为`INTERACTIVE_STATE_UNKNOWN`，即0，由于是成员变量，因此初始值就等于0.开机后灭屏满足这个情景，此时会将`mBroadcastedInteractiveState`置为`INTERACTIVE_STATE_AWAKE`；
- 2.如果`mBroadcastedInteractiveState`为`INTERACTIVE_STATE_AWAKE`，表示当前广播的交互状态为可交互状态，即此时处于亮屏状态；（下同）
- 3.如果`mBroadcastedInteractiveState`为`INTERACTIVE_STATE_ASLEEP`，表示此时处于灭屏状态。
到这里为止，我们理清了发送广播的大概流程，可以确定交互状态改变时对应的操作了，现在分别对亮屏和灭屏的流程进行分析。

#### 3.3.1.由灭屏-亮屏：

`onWakefulnessChangeStarted()`方法中部分变量初始值和调用流程:
```
wakefulness=1, 即WAKEFULNESS_AWAKE；
reason=0, PMS中传入；
interactive=true，即处于AWAKE或者DREAM状态
mInteractive=false,即前一次不出于交互状态，
mInteractiveChanging=false，刚进入该函数该值初始值；
handleEarlyInteractiveChange()被调用；  --->
handleEarlyInteractiveChange()方法中相关变量赋值：
mPendingInteractiveState = INTERACTIVE_STATE_AWAKE,即为1,最终	要广播的交互状态；
mPendingWakeUpBroadcast = true;表示是否最终发送的为wakeup广播；
updatePendingBroadcastLocked()被调用；   --->
updatePendingBroadcastLocked()中的变量值：
mBroadcastInProgress=false, 表示当前没有正在进行的广播；
mPendingInteractiveState=1, handleEarlyInteractiveChange()中赋值；
mPendingWakeUpBroadcast=true,handleEarlyInteractiveChange()中赋值
mPendingGoToSleepBroadcast=false,是否最终发送的为asleep广播；
mBroadcastedInteractiveState=2，当前广播的状态（还未发送wakeup广播状态），为INTERACTIVE_STATE_ASLEEP，2；
sendNextBroadcast()中根据上述变量值，走else语句：
mPendingWakeUpBroadcast=false，
mBroadcastedInteractiveState=INTERACTIVE_STATE_AWAKE;
sendWakeUpBroadcast()被调用。      --->

```
`sendWakeUpBroadcast()`中发送`Intent_SCREEN_ON`广播：
```
private void sendWakeUpBroadcast() {
    if (mActivityManagerInternal.isSystemReady()) {
        //广播发送完成后最后被mWakeUpBroadcastDone接受
        mContext.sendOrderedBroadcastAsUser(mScreenOnIntent, 
               UserHandle.ALL, null, mWakeUpBroadcastDone, mHandler,
                0, null, null);
    } else {
        sendNextBroadcast();
    }
}

```
`mWakeUpBroadcastDone`会在最后接受触发onReceive()方法，继续看看`MWakeUpBroadcastDone`这个广播接受器：
```
private final BroadcastReceiver mWakeUpBroadcastDone = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        EventLog.writeEvent(EventLogTags.POWER_SCREEN_BROADCAST_DONE, 1,
                SystemClock.uptimeMillis() - mBroadcastStartTime, 1);
        sendNextBroadcast();
    }
};

```
在这里，又调用了`sendNextBroadcast()`方法，并根据条件判断，走`else if `语句，此时相关变量值如下：
```
mPendingWakeUpBroadcast = false;上次else if中设置；
mPendingGoToSleepBoradcast = false;发送灭屏广播时才会设置为true；
mPendingInteractiveState = 1;handleEarlyInteractiveChange()中设置；

```
结合以上变量值，在`else if`中走`else`语句，else中直接调用了`finishPendingBroadcastLocked()`方法，该方法如下：
```
private void finishPendingBroadcastLocked() {
    //表示此时没有正在进行的广播
    mBroadcastInProgress = false;
    //释放suspend锁
    mSuspendBlocker.release();
}

```
在这个方法中，将`mBroadcastInProgress`值设置为false,表示当前没有正在进行中的广播，并释`sendGoToSleepBroadcast`放了避免系统`CPU`休眠的`Suspend`锁。亮屏广播就发送完毕了。

#### 3.3.2.由亮屏-灭屏：

`onWakefulnessChangeStarted()`方法中部分变量初始值:
```
wakefulness=3, WAKE_LOCK_DOZE,即先进入doze模式；
reason=4, PM.GO_TO_SLEEP_POWER_BUTTTON,即由power键灭屏；
interactive=false,由于wakefulness已变为doze，因此不可交互状态；
mInteractive=true,上次状态为可交互状态；
mInteractiveChanging=false，刚进入该函数该值初始值；
handleEarlyInteractiveState()被调用；
handleEarlyInteractiveState()中：只做了通过WindowManagerPolicy开始睡眠的操作，之后返回。
在PMS的handleSandman()中,调用reallyGoToSleepNoUpdateLocke()方法进入睡眠，因此又会调用到onWakefulnessChangeStarted()方法中：
onWakefulnessChangeStarted()中部分变量初始值：
wakefulness=0,WAKEFULNESS_ASLEEP=0;表示进入睡眠状态
 reason=2,PM.GO_TO_SLEEP_REASON_TIMEOUT.
interactive=false,当前为asleep状态，表示不可交互
mInteractive=false,上次为doze状态，也是不可交互
mInteractiveChanging=true，是上次由awake->doze时设置；
handleEarlyInteractiveState()被调用；
handleEarlyInteractiveState()中：只做了通过WindowManagerPolicy开始睡眠的操作，之后返回。

```
最终，在`onWakefulnessChangeFinished()`方法中，调用了`handleLateInteractiveChanged(),`发送灭屏广播前设置值如下：
```
mPendingInteractiveState = INTERACTIVE_STATE_ASLEEP;
mPendingGoToSleepBroadcast = true;
updatePendingBroadcastLocked();

```
进入到`sendNextBroadcast()`中，此时部分变量值如下：
```
mBroadcastedInteractiveState=1,表示当前广播的交互状态为awake;
mPendingWakeUpBroadcast=false,是否最终发送的为wakeup广播；
mPendingGoToSleepBroadcast=true,表示最终发送的为sleep广播；

```
因此，走`sendNextBroadcast()`中的else if->if语句，`sendGoToSleepBroadcast()`被调用；
`sendGoToSleepBroadcast()`中发送`Intent.SCREEN_OFF`广播：
```
private void sendGoToSleepBroadcast() {
    if (mActivityManagerInternal.isSystemReady()) {
        //广播发送后最后一个广播接受器mGoToSleepBroadcastDone
        mContext.sendOrderedBroadcastAsUser(mScreenOffIntent, 
               UserHandle.ALL, null,
                mGoToSleepBroadcastDone, mHandler, 0, null, null);
    } else {
        sendNextBroadcast();
    }
}

```
灭屏广播发出后，`mGoToSleepBroadcastDone`会在最后接受到，这里进行收尾处理：
```
private final BroadcastReceiver mGoToSleepBroadcastDone = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        sendNextBroadcast();
    }
};
```
这里又调用了`sendNextBroadcast()`方法，此时相关变量值如下：
```
mPendingWakeUpBroadcast=false,
mPendingGoToSleepBroadcast=false,
mPendingInteractiveState=true,
mBroadcastInteractiveState=ASLEEP(2);
```
因此，走else->else语句，调用了`finishPendingBroadcastLocked()`,在这个方法中重置了`mBroadcastInPorgress`和释放了`Suspend`。
`Notifier`类比较简单，稍微复杂的就是发送广播这块。
![](https://upload-images.jianshu.io/upload_images/5851256-9942702b8d1a73b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

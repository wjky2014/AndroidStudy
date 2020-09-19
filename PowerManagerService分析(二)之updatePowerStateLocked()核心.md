 
##### 和您一起终身学习，这里是程序员Android 


在[PowerManagerService分析(一) ](https://mp.weixin.qq.com/s/Mgi1W9mmrCUPkASTp3LPrA)中对PMS的启动流程进行了分析，本篇对PMS中的一些核心方法进行分析。

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>1.updatePowerStateLocked方法详解
>2.updateIsPoweredLocker()
>3.updateStayOnLocked()
>4.updateScreenBrightnessBoostLocked()
>5.updateWakeLockSummaryLocked()
>6.updateUserActivitySummaryLocked()
>7.for(;;)循环和updateWakefulnessLocked()
>8.updateDisplayPowerStateLocked()
>9.updateDreamLocked()
>10.finishWakefulnessChangeIfNeededLocked()
>11.updateSuspendBlockerLocked()



> 备注：文章转载于网络，地址如下：[原文地址](https://blog.csdn.net/FightFightFight/article/details/80341728)

![](https://upload-images.jianshu.io/upload_images/5851256-600dca83084c7424.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 1.2.updatePowerStateLocked方法详解

接着上文分析，在`systemReady()`方法的最后，调用了`updatePowerStateLocked()`方法：
```
public void systemReady(IAppOpsService appOps) {
synchronized (mLock) {
    mSystemReady = true;
    ........
    mDirty |= DIRTY_BATTERY_STATE;
    updatePowerStateLocked();
}
```
`updatePowerStateLocked()`方法是整个PMS中的核心方法，也是整个PMS中最重要的一个方法，它用来更新整个电源状态的改变，并进行重新计算。PMS中使用一个int值mDirty作为标志位判断电源状态是否发生变化，当电源状态发生改变时，如亮灭屏、电池状态改变、暗屏…都会调用该方法，在该方法中调用了其他同级方法进行更新，下面逐个进行分析，先看其代码:

```
private void updatePowerStateLocked() {
    if (!mSystemReady || mDirty == 0) {
	   return;
        }
	try {
	// Phase 0: Basic state updates.
	//更新电池信息
	updateIsPoweredLocked(mDirty);
	//更新屏幕保持唤醒标识值mStayOn
	updateStayOnLocked(mDirty);
	//亮度增强相关
	updateScreenBrightnessBoostLocked(mDirty);
	
	final long now = SystemClock.uptimeMillis();
	int dirtyPhase2 = 0;
	for (;;) {
	    int dirtyPhase1 = mDirty;
	    dirtyPhase2 |= dirtyPhase1;
	    mDirty = 0;
	    //更新统计wakelock的标记值mWakeLockSummary
	    updateWakeLockSummaryLocked(dirtyPhase1);
	    //更新统计userActivity的标记值mUserActivitySummary和休眠到达时间
	    updateUserActivitySummaryLocked(now, dirtyPhase1);
	    //用来更新屏幕唤醒状态，状态改变返回true
	    if (!updateWakefulnessLocked(dirtyPhase1)) {
	        break;
	    }
	}
	// Phase 2: Update display power state.
	//和Display交互，请求Display状态
	boolean displayBecameReady = updateDisplayPowerStateLocked(dirtyPhase2);
	// Phase 3: Update dream state (depends on display ready signal).
	//更新屏保
	updateDreamLocked(dirtyPhase2, displayBecameReady);
	// Phase 4: Send notifications, if needed.
	//如果wakefulness改变，做最后的收尾工作
	finishWakefulnessChangeIfNeededLocked();
	// Phase 5: Update suspend blocker.
	// Because we might release the last suspend blocker here, we need to make 
	//surewe finished everything else first!
	//更新Suspend锁
	updateSuspendBlockerLocked();
	} finally {
	    Trace.traceEnd(Trace.TRACE_TAG_POWER);
	}
}
```
如果没有进行特定场景的分析，这块可能很难理解，在后续的分析中会对特定场景进行分析，这样更能理解方法的使用，如果这里还不太理解，不用太担心。

在整个方法中，当系统没有准备就绪或者mDirty没有进行置位时，不会执行后续步骤，直接return；接下来对该方法中的内容进行分析，由于此方法非常重要，因此这里将所有的方法都贴出来。

####1.2.1.updateIsPoweredLocker()

这个方法主要功能有两个：
- 1.USB插播亮屏入口点；
- 2.更新低电量模式；

该方法如下:
```
/**
* Updates the value of mIsPowered.
* Sets DIRTY_IS_POWERED if a change occurred.
*/
private void updateIsPoweredLocked(int dirty) {
if ((dirty & DIRTY_BATTERY_STATE) != 0) {
//是否充电
final boolean wasPowered = mIsPowered;
final int oldPlugType = mPlugType;//充电类型
final boolean oldLevelLow = mBatteryLevelLow;//是否处于低电量
/*---------------------BatteryService交互Begin-----------------------------*/
mIsPowered = mBatteryManagerInternal.isPowered(BatteryManager.
                         BATTERY_PLUGGED_ANY);
mPlugType = mBatteryManagerInternal.getPlugType();
mBatteryLevel = mBatteryManagerInternal.getBatteryLevel();
mBatteryLevelLow = mBatteryManagerInternal.getBatteryLevelLow();
/*---------------------BatteryService交互 End-----------------------------*/
//充电状态改变
if (wasPowered != mIsPowered || oldPlugType != mPlugType) {
    mDirty |= DIRTY_IS_POWERED;
    //是否连接无线充电
    final boolean dockedOnWirelessCharger = 
           mWirelessChargerDetector.update(
            mIsPowered, mPlugType, mBatteryLevel);
    final long now = SystemClock.uptimeMillis();
    //插拔充电线是否唤醒屏幕
    if (shouldWakeUpWhenPluggedOrUnpluggedLocked(wasPowered, 
              oldPlugType, dockedOnWirelessCharger)) {
        //屏幕唤醒
        wakeUpNoUpdateLocked(now, 
              "android.server.power:POWER", Process.SYSTEM_UID,
               mContext.getOpPackageName(), Process.SYSTEM_UID);
    }
    //更新用户活动
    userActivityNoUpdateLocked(
            now, PowerManager.USER_ACTIVITY_EVENT_OTHER, 0, 
            Process.SYSTEM_UID);
    if (dockedOnWirelessCharger) {
        mNotifier.onWirelessChargingStarted();
    }
}
//上次是否充电和当前是否充电不同||上次电量是否处于低电量和当前是否处于
//低电量不同
if (wasPowered != mIsPowered || oldLevelLow != mBatteryLevelLow) {
    //上次处于低电量&&当前不处于低电量
    if (oldLevelLow != mBatteryLevelLow && !mBatteryLevelLow) {
        //当电池处于低电量模式触发值时，用户是否关闭了低电量模式
        mAutoLowPowerModeSnoozing = false;
    }
    //更新低电量模式
    updateLowPowerModeLocked();
      }
    }
}
```
从代码中看出，只有满足`mDirty&DIRTY_BATTERY_STATE!=0`时才会执行这个方法，满足此条件的有两处:
- 第一处：在调用`systemReady()`方法中：
```
public void systemReady(IAppOpsService appOps) {
    .................
    mDirty |= DIRTY_BATTERY_STATE;
    updatePowerStateLocked();
    .................
}
```
- 第二处：在监听电量状态改变的广播中：

```
//监听ACTION_BATTERY_CHANGED
private final class BatteryReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        synchronized (mLock) {
        //设置mDirty |= DIRTY_BATTERY_STAT
        handleBatteryStateChangedLocked();
      }
   }
}
```
因此可以看到，这个方法跟电池有关，只要电池状态发生变化，就能够调用执行到这个方法进行操作。

在这个方法中，通过`BatteryService`的本地服务`BatteryManagerInternal`，和`BatteryService`进行交互，刷新电池信息，并记录上次的电池数据；然后判断是插拔USB是否需要唤醒屏幕。我们在插拔USB时，可以唤醒屏幕就是从这里为入口进行唤醒的。然后更新用户活动事件；之后会处理低电量时的操作，在if语句中，如果插拔充电线，此时满足条件`wasPowered != mIsPowered`;或者上次和本次有一次处于低电量，则进入if语句；如果在当前电量未达到低电量值，并且之前处于低电量状态下，此时满足if语句中的if语句，说明此时电量大于低电量值，将`mAutoLowPowerModeSnoozing` 置为false.于是更新低电量相关的设置。在`updateLowPowerModeLocked()`方法中进行低电量模式相关更新，这个方法在低电量模式时会进行分析。


####1.2.2.updateStayOnLocked()

这个方法主要用于判断系统是否在Settings中设置了充电时保持屏幕亮屏后，根据是否充电来决定亮屏与否。方法如下：
```
private void updateStayOnLocked(int dirty) {
    if ((dirty & (DIRTY_BATTERY_STATE | DIRTY_SETTINGS)) != 0) {
    final boolean wasStayOn = mStayOn;
    //充电时亮屏&&DevicePolicyManager中未设置最大关闭时间
       if (mStayOnWhilePluggedInSetting != 0 && 
            !isMaximumScreenOffTimeoutFromDeviceAdminEnforcedLocked()) {
            //保持亮屏取决于是否充电
            mStayOn = 
                mBatteryManagerInternal.isPowered(mStayOnWhilePluggedInSetting);
        } else {
            mStayOn = false;
        }
        if (mStayOn != wasStayOn) {
            //如果mStayOn值改变，mDirty置位
            mDirty |= DIRTY_STAY_ON;
        }
    }
}

```

在该方法中，用bool值mStayOn作为标志，`mStayOnWhilePluggedInSetting`是从`SettingsProvider`中读取的值，表示是否设置了充电时保持屏幕常亮，若要使mStayOn为true,其先决条件是`mStayOnWhilePluggedInSetting`为true，同时`DevicePolicyManager`没有进行最大超时时间的约束，如果符合这个条件，则当设备在充电时`mStayOn`为true.其他情况下都为false.mStayOn在进入Dream相关从操作时作为判断条件用到。
在方法的最后，如果mStayOn值改变，则更新mDirty标志位。

####1.2.3.updateScreenBrightnessBoostLocked()

这部分代码在`PhoneWindowManager`中触发的，具体没有研究，和亮度增强有关系。
```
    private void updateScreenBrightnessBoostLocked(int dirty) {
        if ((dirty & DIRTY_SCREEN_BRIGHTNESS_BOOST) != 0) {
            if (mScreenBrightnessBoostInProgress) {
                final long now = SystemClock.uptimeMillis();
                mHandler.removeMessages(MSG_SCREEN_BRIGHTNESS_BOOST_TIMEOUT);
                if (mLastScreenBrightnessBoostTime > mLastSleepTime) {
                    final long boostTimeout = mLastScreenBrightnessBoostTime +
                            SCREEN_BRIGHTNESS_BOOST_TIMEOUT;
                    if (boostTimeout > now) {
                        Message msg = mHandler.obtainMessage(MSG_SCREEN_BRIGHTNESS_BOOST_TIMEOUT);
                        msg.setAsynchronous(true);
                        mHandler.sendMessageAtTime(msg, boostTimeout);
                        return;
                    }
                }
                mScreenBrightnessBoostInProgress = false;
                mNotifier.onScreenBrightnessBoostChanged();
                userActivityNoUpdateLocked(now,
                        PowerManager.USER_ACTIVITY_EVENT_OTHER, 0, Process.SYSTEM_UID);
            }
        }
    }

```
####1.2.4.updateWakeLockSummaryLocked()

在这个方法中，会对当前所有的`WakeLock`锁进行统计，过滤所有的wakelock锁状态（wakelock锁机制在后续进行分析），并更新`mWakeLockSummary`的值以汇总所有活动的唤醒锁的状态。`mWakeLockSummary`是一个用来记录所有WakeLock锁状态的标识值，该值在请求Display状时会用到。当系统处于睡眠状态时，大多数唤醒锁都将被忽略，比如系统在处于唤醒状态(awake)时，会忽略`PowerManager.DOZE_WAKE_LOCK`类型的唤醒锁，系统在处于睡眠状态(asleep)或者Doze状态时，会忽略`PowerManager.SCREEN_BRIGHT`类型的锁等等。该方法如下：

```
private void updateWakeLockSummaryLocked(int dirty) {
    if ((dirty & (DIRTY_WAKE_LOCKS | DIRTY_WAKEFULNESS)) != 0) {
    mWakeLockSummary = 0;
    //在waklock集合中遍历wakelock
    final int numWakeLocks = mWakeLocks.size();
    for (int i = 0; i < numWakeLocks; i++) {
        final WakeLock wakeLock = mWakeLocks.get(i);
        switch (wakeLock.mFlags & PowerManager.WAKE_LOCK_LEVEL_MASK) {
            case PowerManager.PARTIAL_WAKE_LOCK:
                if (!wakeLock.mDisabled) {
                    // We only respect this if the wake lock is not disabled.
                    //如果存在PARTIAL_WAKE_LOCK并且该wakelock可用,
                    //通过置位进行记录，下同
                    mWakeLockSummary |= WAKE_LOCK_CPU;
                }
                break;
            case PowerManager.FULL_WAKE_LOCK:
                mWakeLockSummary |= WAKE_LOCK_SCREEN_BRIGHT | 
                WAKE_LOCK_BUTTON_BRIGHT;
                break;
            case PowerManager.SCREEN_BRIGHT_WAKE_LOCK:
                mWakeLockSummary |= WAKE_LOCK_SCREEN_BRIGHT;
                break;
            case PowerManager.SCREEN_DIM_WAKE_LOCK:
                mWakeLockSummary |= WAKE_LOCK_SCREEN_DIM;
                break;
            case PowerManager.PROXIMITY_SCREEN_OFF_WAKE_LOCK:
                mWakeLockSummary |= 
                WAKE_LOCK_PROXIMITY_SCREEN_OFF;
                break;
            case PowerManager.DOZE_WAKE_LOCK:
                mWakeLockSummary |= WAKE_LOCK_DOZE;
                break;
            case PowerManager.DRAW_WAKE_LOCK:
                mWakeLockSummary |= WAKE_LOCK_DRAW;
                break;
        }
    }
    // Cancel wake locks that make no sense based on the current state.
    /**
     * 设备不处于DOZE状态时，通过置位操作忽略相关类型wakelock
     * PowerManager.DOZE_WAKE_LOCK和WAKE_LOCK_DRAW锁仅仅
     * 在Doze状态下有效
     */
    if (mWakefulness != WAKEFULNESS_DOZING) {
        mWakeLockSummary &= ~(WAKE_LOCK_DOZE | WAKE_LOCK_DRAW);
    }
    /**
     * 如果处于Doze状态，忽略三类Wakelock.
     * 如果处于睡眠状态，忽略四类wakelock.
     */
    if (mWakefulness == WAKEFULNESS_ASLEEP
            || (mWakeLockSummary & WAKE_LOCK_DOZE) != 0) {
        mWakeLockSummary &= ~(WAKE_LOCK_SCREEN_BRIGHT | 
               WAKE_LOCK_SCREEN_DIM| WAKE_LOCK_BUTTON_BRIGHT);
        if (mWakefulness == WAKEFULNESS_ASLEEP) {
            mWakeLockSummary &= ~WAKE_LOCK_PROXIMITY_SCREEN_OFF;
        }
    }
    //根据当前状态推断必要的wakelock
    //处于awake或dream(不处于asleep/doze)
    if ((mWakeLockSummary & (WAKE_LOCK_SCREEN_BRIGHT | 
            WAKE_LOCK_SCREEN_DIM)) != 0) {
        //处于awake状态，WAKE_LOCK_STAY_AWAKE只用于awake状态时
        if (mWakefulness == WAKEFULNESS_AWAKE) {
            mWakeLockSummary |= WAKE_LOCK_CPU | 
            WAKE_LOCK_STAY_AWAKE;
            //处于屏保状态(dream)
        } else if (mWakefulness == WAKEFULNESS_DREAMING) {
            mWakeLockSummary |= WAKE_LOCK_CPU;
        }
    }
    if ((mWakeLockSummary & WAKE_LOCK_DRAW) != 0) {
        mWakeLockSummary |= WAKE_LOCK_CPU;
    }
  }
}

```
当这个方法执行后，将未忽略的wakelocks类型都记录在`mWakeLockSummary`这个标志位中。在平时分析处理不灭屏相关Bug时，通过该值可确定当前系统持有哪些类型的锁。

####1.2.5.updateUserActivitySummaryLocked()

该方法用来更新用户活动时间，当设备和用户有交互时，都会根据当前时间和休眠时长、Dim时长、所处状态而计算下次休眠的时间，从而完成用户活动超时时的操作。如由亮屏进入Dim的时长、Dim到灭屏的时长、亮屏到屏保的时长，就是在这里计算的。进行关键代码如下（有删减）：
```
private void updateUserActivitySummaryLocked(long now, int dirty) {
    // Update the status of the user activity timeout timer.
    if ((dirty & (DIRTY_WAKE_LOCKS | DIRTY_USER_ACTIVITY
        | DIRTY_WAKEFULNESS | DIRTY_SETTINGS)) != 0) {
    mHandler.removeMessages(MSG_USER_ACTIVITY_TIMEOUT);
    long nextTimeout = 0;
    //如果处于休眠状态，则不会执行该方法
    if (mWakefulness == WAKEFULNESS_AWAKE
            || mWakefulness == WAKEFULNESS_DREAMING
            || mWakefulness == WAKEFULNESS_DOZING) {
        //设备完全进入休眠所需时间，该值为-1表示禁用此值，默认-1
        final int sleepTimeout = getSleepTimeoutLocked();
        //用户超时时间，既经过一段时间不活动进入休眠或屏保的时间，特殊情况外，该值为Settings中的休眠时长
        final int screenOffTimeout = getScreenOffTimeoutLocked(sleepTimeout);
        //Dim时长，即亮屏不操作，变暗多久休眠
        final int screenDimDuration =  
          getScreenDimDurationLocked(screenOffTimeout);
        //通过WindowManager的用户交互
        final boolean userInactiveOverride = 
            mUserInactiveOverrideFromWindowManager;
        mUserActivitySummary = 0;
        //1.亮屏；2.亮屏后进行用户活动
        if (mLastUserActivityTime >= mLastWakeTime) {
            //下次睡眠时间=上次用户活动时间+休眠时间-Dim时间
            nextTimeout = mLastUserActivityTime
                + screenOffTimeout - screenDimDuration;
            //如果满足当前时间<下次屏幕超时时间，说明此时设备为亮屏状态，则将用户活动状态置为表示亮屏的USER_ACTIVITY_SCREEN_BRIGHT
            if (now < nextTimeout) {
                mUserActivitySummary = USER_ACTIVITY_SCREEN_BRIGHT;
            } else {
                //如果当前时间>下次活动时间，此时应有两种情况：已经休眠和Dim
                nextTimeout = mLastUserActivityTime + screenOffTimeout;
                //如果当前时间<上次活动时间+屏幕超时时间，这个值约为3s,说明此时设备为Dim状态，则将用户活动状态置为表示Dim的USER_ACTIVITY_SCREEN_DIM
                if (now < nextTimeout) {
                    mUserActivitySummary = USER_ACTIVITY_SCREEN_DIM;
                }
            }
        ...........................................................
        //发送定时Handler，到达时间后再次进行updatePowerStateLocked()
        if (mUserActivitySummary != 0 && nextTimeout >= 0) {
            Message msg = 
                 mHandler.obtainMessage(MSG_USER_ACTIVITY_TIMEOUT);
                 msg.setAsynchronous(true);
                 mHandler.sendMessageAtTime(msg, nextTimeout);
             }
         } else {
          mUserActivitySummary = 0;
         }
    }
}

```
在获取用户活动超时时长时，不仅仅是由用户在设置中设置的休眠时长所决定，还有比如带有`PowerManager.ON_AFTER_RELEASE`标记的wakelock锁在释放时也会影响用户超时时间。上述代码中只列出了最常见的一种，即由用户亮屏到到达时间休眠的代码逻辑。这里举个例子，假设设备设定休眠时间为15s，Dim时长为3s，我在9:20:01时按power键唤醒设备，因此，执行到该方法时有：

>mLastUserActivity=mLastWakeTime=9:20:01,now=9:20:01+0.02ms,screenOffTimeout=15s,screenDimDuration=3s,所以nextTimeout为9:20:01+15s-3s=9:20:13

因此会通过Handler发送一个定时消息，13秒后会进入Dim…
现在时间到9:20:13，执行Handler中的消息，这个消息中又调用了一次updatePowerState()方法，所以又会执行到这个方法，此时：只有now发生改变为9:20:13+0.02ms(调用方法消耗的时间),因此now>nextTimeout,进入else语句，进入else后：

>nextTimeout = mLastUserActivity + screenOffTimeout =9:20:01+15s=9:20:16 > now

因此判断当前因为Dim状态，同时nextTimeout发生改变，并且再次通过Handler设置定时消息，…，3s后，又回到了该方法中进行了处理，这次处理，会通过判断将nextTimeout设为-1，从而不再发送Handler，通过updatePowerStateLocked()中的其他方法进行休眠。

在计算完成nextTimeout后，会通过Handler发送一个延时消息，到达nextTimeout后，再次更新整个电源状态：
```
private final class PowerManagerHandler extends Handler {
        public PowerManagerHandler(Looper looper) {
            super(looper, null, true /*async*/);
        }
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_USER_ACTIVITY_TIMEOUT:
                    handleUserActivityTimeout();
                    break;
             ......
 
 private void handleUserActivityTimeout() { // runs on handler thread
    synchronized (mLock) {
        if (DEBUG_SPEW) {
            Slog.d(TAG, "handleUserActivityTimeout");
        }

        mDirty |= DIRTY_USER_ACTIVITY;
        updatePowerStateLocked();
    }
}

```
此外，这里还有一点需要注意的地方，在计算灭屏超时时间时，有两个值：
```
final int sleepTimeout = getSleepTimeoutLocked();
final int screenOffTimeout = getScreenOffTimeoutLocked(sleepTimeout);
```
这两个方法和休眠时间相关，在PMS中，定义了两个相关值：
```
mSleepTimeoutSetting = Settings.Secure.getIntForUser(resolver,
        Settings.Secure.SLEEP_TIMEOUT, DEFAULT_SLEEP_TIMEOUT,
        UserHandle.USER_CURRENT);

mScreenOffTimeoutSetting = Settings.System.getIntForUser(resolver,
        Settings.System.SCREEN_OFF_TIMEOUT, DEFAULT_SCREEN_OFF_TIMEOUT,
        UserHandle.USER_CURRENT);

```
其中 `Settings.Secure.SLEEP_TIMEOUT`表示设备在经过一段不活动后完全进入睡眠后屏保的时间，该值可以理解为保持唤醒或屏保的最大值或上限，并且该值要大于`Settings.System.SCREEN_OFF_TIMEOUT`,默认为-1，表示禁用此项功能。
`Settings.System.SCREEN_OFF_TIMEOUT`表示设备在经过一段不活动后进入睡眠或屏保的时间，也称为用户活动超时时间，但屏幕到期时不一定关闭。该值可以在设置-休眠中设置。

####1.2.6.for(;;)循环和updateWakefulnessLocked()

在`updatePowerStateLocked()`方法中，设置了一个死循环，并且上述分析的两个方法都在死循环中执行，为何设计一个死循环与它循环内部的实现有关系。接下来逐一分析其内容。for循环代码如下：

```
for (;;) {
    int dirtyPhase1 = mDirty;
    dirtyPhase2 |= dirtyPhase1;
    mDirty = 0;
    //汇总wakelock
    updateWakeLockSummaryLocked(dirtyPhase1);
    //汇总useractivity
    updateUserActivitySummaryLocked(now, dirtyPhase1);
    if (!updateWakefulnessLocked(dirtyPhase1)) {
        break;
    }
}

```
其中汇总`wakelock`和`useractivity`方法在前面已经进行了分析，因此这里主要有一个关键方法:`updateWakefulnessLocked()`,这个方法是退出循环的关键。如果这个方法返回false,则循环结束，如果返回true，则进行下一次循环，那么这个方法返回值有何含义呢？我们继续看看这个方法：
```
private boolean updateWakefulnessLocked(int dirty) {
    boolean changed = false;
    if ((dirty & (DIRTY_WAKE_LOCKS | DIRTY_USER_ACTIVITY | 
        DIRTY_BOOT_COMPLETED
        | DIRTY_WAKEFULNESS | DIRTY_STAY_ON | 
        DIRTY_PROXIMITY_POSITIVE
        | DIRTY_DOCK_STATE)) != 0) {
           //当前屏幕保持唤醒&&设备将要退出唤醒状态(睡眠or屏保)
           if (mWakefulness == WAKEFULNESS_AWAKE && isItBedTimeYetLocked()) {
               Slog.d(TAG, "updateWakefulnessLocked: Bed time...");
               final long time = SystemClock.uptimeMillis();
               //是否在休眠时启用屏保
               if (shouldNapAtBedTimeLocked()) {
                  //进入屏保，返回true
                  changed = napNoUpdateLocked(time, Process.SYSTEM_UID);
                  } else {
                     //进入睡眠，返回true
                     changed = goToSleepNoUpdateLocked(time,
                     PowerManager.GO_TO_SLEEP_REASON_TIMEOUT, 0, 
                     Process.SYSTEM_UID);
                 }
             }
       }
    return changed;
}

```
这个方法用于更新设备的`wakefulness`，同时，这个方法是`亮屏->屏保/睡眠`的决策点。`wakefulness`是用来表示当前设备状态的一个值，系统定义的wakefulness值共有四种，分别表示不同的状态：

```
//睡眠状态，此时灭屏
public static final int WAKEFULNESS_ASLEEP = 0;
//屏幕亮
public static final int WAKEFULNESS_AWAKE = 1;
//屏保
public static final int WAKEFULNESS_DREAMING = 2;
//处于DOZE模式时
public static final int WAKEFULNESS_DOZING = 3;

```
如果当前设备处于唤醒状态(awake)，并且将要退出唤醒状态，也就是进入睡眠状态(sleep)或者屏保状态(dreaming),如果设备开启了屏保，进入屏保状态，否则直接进入睡眠状态。这种情况下，wakefulness发生改变，因此返回值为true,需要通过下一次循环重新统计`wakelockSummary`和`userActivitySummary`。如果不是以上情况，则不会进入if语句，说明不需要改变wakefulness值，返回false，则循环体只执行一次便退出。因此，该循环只有在一段时间不活动到达用户超时时间后进入屏保或者进入睡眠时，会执行两次，其他情况下只执行一次便退出，比如按power键灭屏等只会执行一次，因为当power键灭屏时，wakefulness值已经由唤醒状态变为SLEEP状态，因此不满足执行条件。

知道`updatefulnessLocked()`方法的主要功能后，现在来看看其中和休眠、屏保相关的方法。首先来看`isItBedTimeYetLocked()`方法，该方法判断当前设备是否将要进入睡眠状态，返回值为对`isBeKeptAwakeLocke()`方法返回值取反，由`mStayOn(是否屏幕常亮)、wakelockSummary、userActivitySummary、mProximityPositive`等决定，只要满足其中之一为ture,则说明无法进入睡眠，也就说，要满足进入睡眠，相关属性值都为false。`isBeKeptAwakeLocke()`如下：

```
private boolean isBeingKeptAwakeLocked() {
return mStayOn//屏幕是否保持常亮
    || mProximityPositive//接近传感器接近屏幕时为true
    //处于awake状态
    || (mWakeLockSummary & WAKE_LOCK_STAY_AWAKE) != 0
    //屏幕处于亮屏或者dim状态
    || (mUserActivitySummary & (USER_ACTIVITY_SCREEN_BRIGHT
            | USER_ACTIVITY_SCREEN_DIM)) != 0            
    || mScreenBrightnessBoostInProgress;//处于亮度增强中
}

```
接下来看看`shoudNapAtBedTimeLocked()`方法。这个方法用来判断设备是否进入屏保模式：
```
private boolean shouldNapAtBedTimeLocked() {
//屏保是否开启
return mDreamsActivateOnSleepSetting
    || (mDreamsActivateOnDockSetting //插入基座时是否开启屏保
            && mDockState != Intent.EXTRA_DOCK_STATE_UNDOCKED);
}

```
除了以上方法外，还有`napNoUpdateLocked()`和`goToSleepNoUpdateLocked()`方法，这两个方法分别用于控制设备进入屏保或者休眠，将在特定场景下进行分析。

结合上述三个方法的分析，之所以把`updateWakeLockSummaryLocked()、updateUserActivitySummaryLocked()、updateWakefulnessLocked()`这三个方法放在for(;;)循环中调用，是因为它们共同决定了设备的状态，前两个方法是汇总状态，后一个方法是根据前两个方法汇总的值而进行判断是否要改变当前的设备唤醒状态,汇总状态会受mWakefulness的影响，因此会进行循环处理。同时，也仅仅会在超时灭屏进入睡眠或屏保时，for循环会执行两次，其他情况下，只会执行一次。

####1.2.7.updateDisplayPowerStateLocked()

该方法用于更新设备显示状态，在这个方法中，会计算出最终需要显示的亮度值和其他值，然后将这些值封装到`DisplayPowerRequest`对象中，向`DisplayMangerService`请求Display状态，完成屏幕亮度显示等。
首先看一下亮度值，系统的亮度值并不是简单的在Settings中设置了就可以了，决定亮度值的一些相关值如下：
```
//WindowManager覆盖的亮度值，如播放视频时调节亮度
//-1表示禁止使用(未发现使用到)
private int mScreenBrightnessOverrideFromWindowManager = -1;
//SystemUI中设置的临时亮度值，自动亮度时无效
//该值之所以是临时的，是因为当调节亮度进度条时，会调用到updateDisplayPowerLocked(),这里给它赋值；
// 当手指放开时，调用updateSettingLocked(),这里又将它置为-1
private int mTemporaryScreenBrightnessSettingOverride = -1;
//Settings.System.SCREEN_BRIGHTNESS中的值，即反映给用户的值
private int mScreenBrightnessSetting;
//限定值,config.xml中配置
private int mScreenBrightnessSettingMinimum;//0
private int mScreenBrightnessSettingMaximum;//255
private int mScreenBrightnessSettingDefault;//102
private int mScreenBrightnessForVrSettingDefault;//86
//自动调节亮度调整值,-1～1
private float mScreenAutoBrightnessAdjustmentSetting;
//Settings中的默认值，一般和config.xml中的默认值相同，也可能不同
Settings.System.SCREEN_BRIGHTNESS

```
最终显示的亮度值和不同的状态有关，如在自动亮度调节打开时、VR模式时、或者从一个视频播放窗口中调节亮度时，都对应有不同的值。
在这个方法中，用到了一个`DisplayPowerRequest`类的对象，这个类是`DisplayManageInternal`类中的一个内部类，专门用于`PowerManagerService`和`DisplayPowerController`交互。PMS中会将当前亮度、屏幕状态等多个值封装在这个对象中，然后调用`requestPowerState()`请求`DisplayPowerController`,从而完成显示更新。

`DisplayPowerRequest`中定义的属性如下：

```
// 可以理解为Display的‘策略‘，该值非常重要，决定了请求后屏幕的状态，有四个值:off，doze,dim,bright.
public int policy;
// 如果为true，则PSensor会覆盖屏幕状态，当物体靠近时将其临时关闭直到物体移开。
public boolean useProximitySensor;
// 屏幕亮度
public int screenBrightness;
// 自动调节亮度值，-1(dimmer)至1(brighter)之间
public float screenAutoBrightnessAdjustment;
// 如果screenBrightness和screenAutoBrightnessAdjustment 由用户设置，则为true
public boolean brightnessSetByUser;
// 是否使用了自动调节亮度
public boolean useAutoBrightness;
// 是否使用了低电量模式，该模式下亮度会减半
public boolean lowPowerMode;
// 在低电量模式下调整屏幕亮度的系数，0（screen off）至1(no change)之间
public float screenLowPowerBrightnessFactor;
// 是否启动了亮度增强
public boolean boostScreenBrightness;
// 如果为true，则会在屏幕亮起时阻止屏幕完全亮起，窗口管理器策略在准备键盘保护程// 时阻止屏幕，以防止用户看到中间更新。
public boolean blockScreenOn;
// 设备处于Doze状态下时覆盖的屏幕亮度和屏幕状态
public int dozeScreenBrightness;
public int dozeScreenState;

```
至于详细的`DisplayPowerRequest`，在Display分析时进行，这里只需要知道亮屏灭屏、亮度调节时会进行请求,交给`DisplayManagerService`去处理。由于亮度调节这块涉及到的模块有`WindowManager、DisplayManagerService`等模块，和这个方法一起分析内容略显臃肿，因此这里仅仅总结该方法主要逻辑，具体亮度调整流程做了单独分析。整个请求过程如下图所示：

![](https://upload-images.jianshu.io/upload_images/5851256-92f1cc7e1315dd3c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
该方法中主要代码如下：
```
private boolean updateDisplayPowerStateLocked(int dirty) {
    final boolean oldDisplayReady = mDisplayReady;
    mDisplayPowerRequest.policy = getDesiredScreenPolicyLocked();

        //------------亮度值计算 -----------
        ......
        // 封装到DisplayPowerRequest中
        mDisplayPowerRequest.screenBrightness = screenBrightness;
        ......
        // 传给DisplayManagerService中处理
        mDisplayManagerInternal.requestPowerState(mDisplayPowerRequest,
                mRequestWaitForNegativeProximity);
        ....
    return mDisplayReady && !oldDisplayReady;
}

```
在请求`DisplayManagerService`时，会将所有的信息封装到`DisplayPowerRequest`对象中，其中需要注意policy值。policy作为DisplayPowerRequset的属性，有四种值，分别为off、doze、dim、bright、vr。在向DisplayManagerService请求时，会根据当前PMS中的唤醒状态和统计的wakelock来决定要请求的Display状态，这部分源码如下：

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
比如，如果当前`mWakefulness`为`ASLEEP`，则表示要进入睡眠状态，去请求`Display`。最终会在`DisplayPowerController`中处理这次请求，如果发现请求时携带的`DisplayPowerRequest`对象和上一次请求时相同，则直接返回true，表示Display已经准备就绪了，如果请求时携带的`DisplayPowerRequest`对象和上一次请求时的不同，说明状态有所改变，此时会返回false，表示Display状态未准备就绪，同时异步请求新的状态。

这里再次总结下这个方法，首先封装一个`DisplayPowerRequest`对象，并且请求`DisplayPowerController`，交给`DisplayPowerController`去完成剩余任务。该方法有个返回值，表示Display是否准备就绪，并且接下来的屏保也依赖于这个返回值。

####1.2.8.updateDreamLocked()

该方法用来更新设备Dream状态，比如是否继续屏保、Doze或者开始休眠，这个方法中异步处理该过程(因此，这里在唤醒或休眠时有风险)。代码如下：
```
/**
 * Determines whether to post a message to the sandman to update the dream state.
 */
private void updateDreamLocked(int dirty, boolean displayBecameReady) {
    if ((dirty & (DIRTY_WAKEFULNESS
            | DIRTY_USER_ACTIVITY
            | DIRTY_WAKE_LOCKS
            | DIRTY_BOOT_COMPLETED
            | DIRTY_SETTINGS
            | DIRTY_IS_POWERED
            | DIRTY_STAY_ON
            | DIRTY_PROXIMITY_POSITIVE
            | DIRTY_BATTERY_STATE)) != 0 || displayBecameReady) {
        if (mDisplayReady) {
            //通过Handler异步发送一个消息
            scheduleSandmanLocked();
        }
    }
}

```
从这里可以看到，该方法依赖于mDisplayReady值，这个值是上个方法在请求Display时的返回值，表示Display是否准备就绪，因此，只有在准备就绪的情况下才会进一步调用该方法的方法体。在`scheduleSandmanLocked()`方法中，通过Handler发送了一个异步消息，代码如下：
```
private void scheduleSandmanLocked() {
    if (!mSandmanScheduled) {
        //由于是异步处理，因此表示是否已经调用该方法且没有被handler处理
        //，如果为true就不会进入该方法了
        mSandmanScheduled = true;
        Message msg = mHandler.obtainMessage(MSG_SANDMAN);
        msg.setAsynchronous(true);
        mHandler.sendMessage(msg);
    }
}

```
再来看看`handleMessage()`中对接受消息的处理：
```
case MSG_SANDMAN:
    handleSandman();
    break;

```
因此，当`updateDreamLocked()`方法调用后，最终会异步执行这个方法，在这个方法中进行屏保相关处理，继续看看这个方法：
```
private void handleSandman() { // runs on handler thread
    //是否开始进入屏保
    final boolean startDreaming;
    final int wakefulness;
    synchronized (mLock) {
        //为false后下次updateDreamLocked()可处理
        mSandmanScheduled = false;
        wakefulness = mWakefulness;
        //在进入asleep状态后该值为true,用于判断是否处于Dream状态
        if (mSandmanSummoned && mDisplayReady) {
            //当前状态能否进入Dream || 当前wakefulness状态为Doze
            startDreaming = canDreamLocked() || canDozeLocked();
            mSandmanSummoned = false;
        } else {
            startDreaming = false;
        }
    }
    //表示是否正在屏保
    final boolean isDreaming;
    if (mDreamManager != null) {
        //重启屏保
        if (startDreaming) {
            mDreamManager.stopDream(false /*immediate*/);
            mDreamManager.startDream(wakefulness == WAKEFULNESS_DOZING);
        }
        isDreaming = mDreamManager.isDreaming();
    } else {
        isDreaming = false;
    }
    synchronized (mLock) {
        //记录进入屏保时的电池电量
        if (startDreaming && isDreaming) {
            mBatteryLevelWhenDreamStarted = mBatteryLevel;
            if (wakefulness == WAKEFULNESS_DOZING) {
                Slog.i(TAG, "Dozing...");
            } else {
                Slog.i(TAG, "Dreaming...");
            }
        }
        //如果mSandmanSummoned改变或者wakefulness状态改变，则return等待下
        //次处理
        if (mSandmanSummoned || mWakefulness != wakefulness) {
            return; // wait for next cycle
        }
        //决定是否继续Dream
        if (wakefulness == WAKEFULNESS_DREAMING) {
            if (isDreaming && canDreamLocked()) {
                //表示从开启屏保开始电池电量下降这个值就退出屏保，-1表示禁用该值
                if (mDreamsBatteryLevelDrainCutoffConfig >= 0
                        && mBatteryLevel < mBatteryLevelWhenDreamStarted
                                - mDreamsBatteryLevelDrainCutoffConfig
                        && !isBeingKeptAwakeLocked()) {
                } else {
                    return; // continue dreaming
                }
            }
            //退出屏保，进入Doze状态
            if (isItBedTimeYetLocked()) {
                goToSleepNoUpdateLocked(SystemClock.uptimeMillis(),
                        PowerManager.GO_TO_SLEEP_REASON_TIMEOUT, 0, 
                        Process.SYSTEM_UID);
                updatePowerStateLocked();
            } else {
                //唤醒设备，reason为android.server.power:DREAM
                wakeUpNoUpdateLocked(SystemClock.uptimeMillis(), 
                        "android.server.power:DREAM",
                        Process.SYSTEM_UID, mContext.getOpPackageName(), 
                        Process.SYSTEM_UID);
                updatePowerStateLocked();
            }
        //如果处于Doze状态,在power键灭屏时，首次会将wakefulness设置为该值
        } else if (wakefulness == WAKEFULNESS_DOZING) {
            if (isDreaming) {
                return; // continue dozing
            }
            //进入asleep状态
            reallyGoToSleepNoUpdateLocked(SystemClock.uptimeMillis(), 
                 Process.SYSTEM_UID);
            updatePowerStateLocked();
        }
    }
    //如果正处在Dream，则只要触发updatePowerStateLocked(),立即退出Dream
    if (isDreaming) {
        mDreamManager.stopDream(false /*immediate*/);
    }
}

```
在PMS中，进入或退出屏保是通过`DreamManager`进行的，在PMS中相当于给DMS下达命令，具体的实现在`DreamManagerService`中。
这里还需要注意，一般在灭屏时，会首先进入Doze状态随后立即进入Sleep状态，但是如果在config.xml文件中配置了`config_dozeComponent`值，则会进入Doze状态，该值表示当系统进入休眠状态时所启动的组件名称，在该组件中调用startDozing()进入Doze Dream状态。如：
`<string name="config_dozeComponent" translatable="false">com.android.systemui/com.android.systemui.doze.DozeService</string>
`
则在灭屏后，首先会进入Doze状态，`DozeService`会申请一个`PowerManager.DOZE_WAKE_LOCK`类型的锁，直到DOZE状态退出后才会进入Sleep状态。
####1.2.9.finishWakefulnessChangeIfNeededLocked()

该方法主要做`updateWakefulnessLocked()`方法的结束工作，可以说`updateWakefulnessLocked()`方法中做了屏幕改变的前半部分工作，而这个方法中做后半部分工作。当屏幕状态改变后，才会执行该方法。我们已经分析了，**屏幕状态有四种**：`唤醒状态(awake)、休眠状态(asleep)、屏保状态(dream)、打盹状态(doze)`，当前屏幕状态由wakefulness表示，当wakefulness发生改变，布尔值`mWakefulnessChanging`变为true。该方法涉及wakefulness收尾相关内容，用来处理wakefulness改变完成后的工作，相关部分如下：
```
private void finishWakefulnessChangeIfNeededLocked() {
    if (mWakefulnessChanging && mDisplayReady) {
        //如果当前处于Doze状态，不进行处理
        if (mWakefulness == WAKEFULNESS_DOZING
                && (mWakeLockSummary & WAKE_LOCK_DOZE) == 0) {
            return; // wait until dream has enabled dozing
        }
        if (mWakefulness == WAKEFULNESS_DOZING || mWakefulness == 
                WAKEFULNESS_ASLEEP) {
            logSleepTimeoutRecapturedLocked();
        }
        if (mWakefulness == WAKEFULNESS_AWAKE) {
            logScreenOn();
        }
        mWakefulnessChanging = false;
        //通过Notifier进行wakefulness改变后的处理
        mNotifier.onWakefulnessChangeFinished();
    }
}

```
可以看到，如果当前屏幕状态处于Doze模式，则不作处理直接return。如果是其他模式，则通过调用Notifier的方法去处理了，Notifier好比PMS的一个喇叭，用来发送广播，和其他组件交互等，都是通过Notifier进行处理的，这个类也会进行单独的分析。
此外，该方法中的`logScreenOn()`方法将打印出整个亮屏流程的耗时，在平时处理问题时也很有帮助。

####1.2.10.updateSuspendBlockerLocked()

在分析这个方法前，先来了解下什么是`Suspend锁`。Suspend锁机制是Android电源管理框架中的一种机制，在前面还提到的wakelock锁也是，不过**wakelock锁**是`上层向framwork层申请`，而**Suspend锁**是`framework层中对wakelock锁的表现`，也就是说，上层应用申请了wakelock锁后，在PMS中最终都会表现为Suspend锁，通过Suspend锁向Hal层写入节点，Kernal层会读取节点，从而进入唤醒或者休眠。这个方法就是用来申请Suspend锁操作，因此，该方法在分析wakelock锁申请流程时进行分析，此处暂且不进行分析。

至此，PMS中核心`updatePowerStateLocked()`以及其中涉及的部分方法分析完毕，文中遗留的Notifier类和Wakelock机制会在后续文章中进行分析，



![](https://upload-images.jianshu.io/upload_images/5851256-9942702b8d1a73b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

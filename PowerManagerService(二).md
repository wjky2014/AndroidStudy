 

##### 和您一起终身学习，这里是程序员Android 

### WakeLock机制

**PowerManager.WakeLock**

为了延长电池的使用寿命，Android设备会在一段时间后使屏幕变暗，然后关闭屏幕显示，直至停止CPU进入休眠。WakeLock是Android提供的唤醒锁机制，用来保持CPU运行或避免屏幕变暗/关闭以及避免键盘背光灯熄灭

**唤醒锁的类型：**


Flag|	CPU|	Screen|	Keyboard
----|----|----|----
PARTIAL_WAKE_LOCK|	on|	off|	off
SCREEN_DIM_WAKE_LOCK|	on|	Dim|	off
SCREEN_BRIGHT_WAKE_LOCK|	on|	Bright|	off
FULL_WAKE_LOCK	|on	|Bright|	on

如果是PARTIAL_WAKE_LOCK,无论屏幕的状态或是按下电源键, CPU都将正常工作。如果是其它的唤醒锁,设备会在用户按下电源钮后停止工作进入休眠状态。以上四种锁，除了PARTIAL_WAKE_LOCK，其余的锁在API level 17已经被deprecated了。


**唤醒锁的使用方法**
代码使用：
```
PowerManager powerManager = (PowerManager) getSystemService(Context.POWER_SERVICE);
PowerManager.WakeLock wl =  powerManager.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK, "My Tag");
wl.acquire();　//acquire时尽量申明timeout时间
// ... do work...
wl.release();
```
权限申明：
```
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.DEVICE_POWER"/>
```
在应用程序中使用WakeLock时必须申明权限，acquire请求唤醒锁时尽量设置timeout时间释放WakeLock，以避免长时间持有WakeLock导致系统无法休眠。

唤醒锁的实现：
`frameworks/base/core/java/android/os/PowerManager.java
`
```
public final class WakeLock {
    private int mFlags;
    private String mTag;
    private final String mPackageName;
    private final IBinder mToken;
    private int mCount;
    private boolean mRefCounted = true;
    private boolean mHeld;
    private WorkSource mWorkSource;
    private String mHistoryTag;
    private final String mTraceName;
    ......
    WakeLock(int flags, String tag, String packageName) {
        mFlags = flags;
        mTag = tag;
        mPackageName = packageName;
        mToken = new Binder();
        mTraceName = "WakeLock (" + mTag + ")";
    }
    ......
}
```
updatePowerStateLocked

`frameworks/base/services/core/java/com/android/server/power/PowerManagerService.java`
```
protected void updatePowerStateLocked() {
    // 服务没有ready,mDirty值没有设置情况下不做更新操作
    if (!mSystemReady || mDirty == 0) {
        return;
    }
    if (!Thread.holdsLock(mLock)) {
        Slog.wtf(TAG, "Power manager lock was not held when calling updatePowerStateLocked");
    }

    try {
        // Phase 0: 更新基本状态
        updateIsPoweredLocked(mDirty);
        updateStayOnLocked(mDirty);
        updateScreenBrightnessBoostLocked(mDirty);

        // Phase 1: Update wakefulness.
        // Loop because the wake lock and user activity computations are influenced
        // by changes in wakefulness.
        final long now = SystemClock.uptimeMillis();
        int dirtyPhase2 = 0;
        for (;;) {
            int dirtyPhase1 = mDirty;
            dirtyPhase2 |= dirtyPhase1;
            mDirty = 0;

            updateWakeLockSummaryLocked(dirtyPhase1);
            updateUserActivitySummaryLocked(now, dirtyPhase1);
            if (!updateWakefulnessLocked(dirtyPhase1)) {
                break;
            }
        }

        // Phase 2: Update display power state.
        boolean displayBecameReady = updateDisplayPowerStateLocked(dirtyPhase2);

        // Phase 3: Update dream state (depends on display ready signal).
        updateDreamLocked(dirtyPhase2, displayBecameReady);

        // Phase 4: Send notifications, if needed.
        finishWakefulnessChangeIfNeededLocked();

        // Phase 5: Update suspend blocker.
        // Because we might release the last suspend blocker here, we need to make sure
        // we finished everything else first!
        updateSuspendBlockerLocked();
    } finally {
        Trace.traceEnd(Trace.TRACE_TAG_POWER);
    }
}
```

首先来看updateIsPoweredLocked(mDirty);

```
private void updateIsPoweredLocked(int dirty) {
    if ((dirty & DIRTY_BATTERY_STATE) != 0) {
        final boolean wasPowered = mIsPowered;
        final int oldPlugType = mPlugType;
        final boolean oldLevelLow = mBatteryLevelLow;
        mIsPowered = mBatteryManagerInternal.isPowered(BatteryManager.BATTERY_PLUGGED_ANY);
        mPlugType = mBatteryManagerInternal.getPlugType();
        mBatteryLevel = mBatteryManagerInternal.getBatteryLevel();
        mBatteryLevelLow = mBatteryManagerInternal.getBatteryLevelLow();

        if (wasPowered != mIsPowered || oldPlugType != mPlugType) {
            mDirty |= DIRTY_IS_POWERED;

            // Update wireless dock detection state.
            final boolean dockedOnWirelessCharger = mWirelessChargerDetector.update(
                    mIsPowered, mPlugType, mBatteryLevel);

            // Treat plugging and unplugging the devices as a user activity.
            // Users find it disconcerting when they plug or unplug the device
            // and it shuts off right away.
            // Some devices also wake the device when plugged or unplugged because
            // they don't have a charging LED.
            final long now = SystemClock.uptimeMillis();
            if (shouldWakeUpWhenPluggedOrUnpluggedLocked(wasPowered, oldPlugType,
                    dockedOnWirelessCharger)) {
                wakeUpNoUpdateLocked(now, "android.server.power:POWER", Process.SYSTEM_UID,
                        mContext.getOpPackageName(), Process.SYSTEM_UID);
            }
            userActivityNoUpdateLocked(
                    now, PowerManager.USER_ACTIVITY_EVENT_OTHER, 0, Process.SYSTEM_UID);

            // Tell the notifier whether wireless charging has started so that
            // it can provide feedback to the user.
            //Bug293654, zhanghong.wt, modify, 20170914, modify no notificatin ring while plugging USB
            if (dockedOnWirelessCharger || (mIsPowered && oldPlugType != mPlugType&&("1".equals(SystemProperties.get("sys.boot_completed"))))) {
                mNotifier.onWirelessChargingStarted();
            }
        }

        if (wasPowered != mIsPowered || oldLevelLow != mBatteryLevelLow) {
            if (oldLevelLow != mBatteryLevelLow && !mBatteryLevelLow) {
                if (DEBUG_SPEW) {
                    Slog.d(TAG, "updateIsPoweredLocked: resetting low power snooze");
                }
                mAutoLowPowerModeSnoozing = false;
            }
            updateLowPowerModeLocked();
        }
    }
}
```
最后附上总结图
![PowerManagerService 总结图](https://upload-images.jianshu.io/upload_images/5851256-4b6bfafaf862adc9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

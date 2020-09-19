 

##### 和您一起终身学习，这里是程序员Android 

>注：文章转于网络，[点击查看原文](https://blog.csdn.net/FightFightFight/article/details/79733559)

![](https://upload-images.jianshu.io/upload_images/5851256-600dca83084c7424.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**PowerManagerService** 之前系列文章请参考如下
1.[PowerManagerService分析(一)之PMS启动](https://mp.weixin.qq.com/s/Mgi1W9mmrCUPkASTp3LPrA)
2.[PowerManagerService分析(二)之updatePowerStateLocked()核心](https://mp.weixin.qq.com/s/P3IvBrYt7afEa4XyEd3BQg)

####WakeLock机制概述

** WakeLock**  是android系统中一种锁的机制，只要有进程持有这个锁，系统就无法进入休眠状态。应用程序要`申请WakeLock`时，需要在清单文件中配置**android.Manifest.permission.WAKE_LOCK** 权限。

根据**作用时间**，`WakeLock`可以分为**永久锁**和**超时锁**，**永久锁**表示只要获取了`WakeLock`锁，**必须显式的进行释放**，否则系统会一直持有该锁；**超时锁**表示在到达给定时间后，自动释放`WakeLock`锁，其实现原理为方法内部维护了一个`Handler`。

根据**释放原则**，WakeLock可以分为**计数锁**和**非计数锁**，默认为计数锁，如果一个WakeLock对象为计数锁，则一次申请必须对应一次释放；如果为非计数锁，则不管申请多少次，一次就可以释放该WakeLock。以下代码为WakeLock申请释放示例，要申请WakeLock，必须有`PowerManager`实例，如下：

```
 PowerManager pm = (PowerManager) getSystemService(Context.POWER_SERVICE);
 PowerManager.WakeLock wl = pm.newWakeLock(PowerManager.SCREEN_DIM_WAKE_LOCK, "My Tag");
 wl.acquire();
 Wl.acquire(int timeout);//超时锁
 // ... do work...
 //释放锁
 wl.release();
```
**Wakelock**申请和释放流程如下：


![](https://upload-images.jianshu.io/upload_images/5851256-a4d11ee2f1255443.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在整个`WakeLock`机制中，对应不同的范围，有三种表现形式：

- 1.PowerManger.WakeLock：
PowerManagerService和其他应用、服务交互的接口；
- 2.PowerManagerService.WakeLock：
 PowerManager.WakeLock在PMS中的表现形式；
- 3.SuspendBlocker：
PowerManagerService.WakeLock在向`底层节点操作`时的表现形式。


下面开始对wakelock的详细分析。

####1.PowerManager中的WakeLock

要获取、申请Wakelock时，直接通过`PowerManager`的WakeLock进行。它作为系统服务的接口来供应用调用。
1.1.获取WakeLock对象
获取WakeLock实例在PowerManager中进行。
在应用中获取WakeLock对象，方式如下：
```
 PowerManager.WakeLock mWakeLock = 
                mPowerManager.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK, TAG);
```
应用中获取wakelock对象，获取的是位于PowerManager中的内部类——WakeLock的实例，在PowerManager中看看相关方法：
```
public WakeLock newWakeLock(int levelAndFlags, String tag) {
    validateWakeLockParameters(levelAndFlags, tag);
    return new WakeLock(levelAndFlags, tag, mContext.getOpPackageName());
}

```
在PowerManager的`newWakeLock()`方法中，首先进行了参数的校验，然后调用WakeLock构造方法获取实例，构造方法如下：
```
WakeLock(int flags, String tag, String packageName) {
//表示wakelock类型或等级
    mFlags = flags;
//一个tag，一般为当前类名
    mTag = tag;
//获取wakelock的包名
    mPackageName = packageName;
//一个Binder标记
    mToken = new Binder();
    mTraceName = "WakeLock (" + mTag + ")";
}

```
除了构造方法中必须要传入的参数之外，还有如下几个属性：
```
//表示内部计数
private int mInternalCount;
//表示内部计数
private int mExternalCount;
//表示是否是计数锁
private boolean mRefCounted = true;
//表示是否已经持有该锁
private boolean mHeld;
//表示和该wakelock相关联的工作源，这在当一个服务获取wakelock执行工作时很有用，以便计算工作成本
private WorkSource mWorkSource;
//表示一个历史标签
private String mHistoryTag;

```
#####1.2.WakeLock等级（类别）

Wakelock共有以下几种等级：
```
//如果持有该类型的wakelock锁，则按Power键灭屏后，即使允许屏幕、按键灯灭，也不会释放该锁，CPU不会进入休眠状态
public static final int PARTIAL_WAKE_LOCK;
//Deprecated，如果持有该类型的wakelock锁，则使屏幕保持亮/Dim的状态，键盘灯允许灭，按Power键灭屏后，会立即释放
public static final int SCREEN_DIM_WAKE_LOCK;
//Deprecated，如果持有该类型的wakelock锁，则使屏幕保持亮的状态，键盘灯允许灭，按Power键灭屏后，会立即释放
public static final int SCREEN_BRIGHT_WAKE_LOCK
//Deprecated，如果持有该类型的wakelock锁，则使屏幕、键盘灯都保持亮，按Power键灭屏后，会立即释放
public static final int FULL_WAKE_LOCK
//如果持有该锁，则当PSensor检测到有物体靠近时关闭屏幕，远离时又亮屏，该类型锁不会阻止系统进入睡眠状态，比如
//当到达休眠时间后会进入睡眠状态，但是如果当前屏幕由该wakelock关闭，则不会进入睡眠状态。
public static final int PROXIMITY_SCREEN_OFF_WAKE_LOCK
//如果持有该锁，则会使屏幕处于DOZE状态，同时允许CPU挂起，该锁用于DreamManager实现Doze模式，如SystemUI的DozeService
public static final int DOZE_WAKE_LOCK
//如果持有该锁,则会时设备保持唤醒状态，以进行绘制屏幕，该锁常用于WindowManager中，允许应用在系统处于Doze状态下时进行绘制
public static final int DRAW_WAKE_LOCK

```
这些值会在下面以图标的形式总结。除了等级之外，还有几个标记：
```
//该值为0x0000FFFF，用于根据flag判断Wakelock的级别，如：
//if((wakeLock.mFlags & PowerManager.WAKE_LOCK_LEVEL_MASK) == PowerManager.PARTIAL_WAKE_LOCK){}
public static final int WAKE_LOCK_LEVEL_MASK
//用于在申请锁时唤醒设备，一般情况下，申请wakelock锁时不会唤醒设备，它只会导致屏幕保持打开状态，如果带有这个flag，则会在申
//请wakelock时就点亮屏幕,如常见通知来时屏幕亮，该flag不能和PowerManager.PARTIAL_WAKE_LOCE一起使用。
public static final int ACQUIRE_CAUSES_WAKEUP
//在释放锁时，如果wakelock带有该标志，则会小亮一会再灭屏，该flag不能和PowerManager.PARTIAL_WAKE_LOCE一起使用。
public static final int ON_AFTER_RELEASE
//和其他标记不同，该标记是作为release()方法的参数，且仅仅用于释放PowerManager.PROXIMITY_SCREEN_OFF_WAKE_LOCK类型的
//锁，如果带有该参数，则会延迟释放锁，直到传感器不再感到对象接近
public static final int RELEASE_FLAG_WAIT_FOR_NO_PROXIMITY

```
####  申请WakeLock

当获取到WakeLock实例后，就可以申请WakeLock了。前面说过了，根据作用时间，WakeLock锁可以分为`永久锁`和`超时锁`，根据释放原则，WakeLock可以分为`计数锁`和`非计数锁`。申请方式如下：
```
 PowerManager pm = (PowerManager) getSystemService(Context.POWER_SERVICE);
 PowerManager.WakeLock wl = pm.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK, "My Tag");
 wl.acquire();//申请一个永久锁
 Wl.acquire(int timeout);//申请一个超时锁

```
`acquire()`方法源码如下：
```
public void acquire() {
    synchronized (mToken) {
        acquireLocked();
    }
}
public void acquire(long timeout) {
    synchronized (mToken) {
        acquireLocked();
        //申请锁之后，内部会维护一个Handler去完成自动释放锁
        mHandler.postDelayed(mReleaser, timeout);
    }
}

```
可以看到这两种方式申请方式完全一样，只不过如果是申请一个超时锁，则会通过`Handler`延时发送一个消息，到达时间后去自动释放锁。
到这一步，对于申请`wakelock`的应用或系统服务来说就完成了，具体的申请在`PowerManager`中进行，继续看看`acquireLocked()`方法：
```
private void acquireLocked() {
    //应用每次申请wakelock，内部计数和外部计数加1
    mInternalCount++;
    mExternalCount++;
    //如果是非计数锁或者内部计数值为1,即第一次申请该锁，才会真正去申请
    if (!mRefCounted || mInternalCount == 1) {
        mHandler.removeCallbacks(mReleaser);
        Trace.asyncTraceBegin(Trace.TRACE_TAG_POWER, mTraceName, 0);
        try {
            //向PowerManagerService申请锁
            mService.acquireWakeLock(mToken, mFlags, mTag, mPackageName, mWorkSource,
                    mHistoryTag);
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
        //表示此时持有该锁
        mHeld = true;
    }
}

```
是否是计数锁可以通过`setReferenceCount()`来设置，默认为计数锁：
```
public void setReferenceCounted(boolean value) {
    synchronized (mToken) {
        mRefCounted = value;
    }
}

```
从`acquire()`方法可以看出，对于计数锁来说，只会在第一次申请时向`PowerManagerService`去申请锁，当该`wakelock`实例第二次、第三次去申请时，如果没有进行过释放，则只会对计数引用加1，不会向`PowerManagerService`去申请。如果是非计数锁，则每次申请，都会调到`PowerManagerService`中去。

释放WakeLock锁

如果是通过`acquire(long timeout)`方法申请的超时锁，则会在到达时间后自动去释放，如果是通过`acquire()`方法申请的永久锁，则必须进行显式的释放，否则由于系统一直持有`wakelock`锁，将导致无法进入休眠状态，从而导致耗电过快等功耗问题。

在前面分析申请锁时已经说了，如果是超时锁，通过`Handler.post(Runnable)`的方式进行释放，该`Runnable`定义如下：

```
private final Runnable mReleaser = new Runnable() {
    public void run() {
        release(RELEASE_FLAG_TIMEOUT);
    }
};

```
`RELEASE_FLAG_TIMEOUT`是一个用于`release()`方法的`flag`，表示释放的为超时锁。
如果是永久锁，则必须通过调用`release()`方法进行释放了，该方法如下：

```
public void release() {
    release(0);
}
```
因此，不管是哪种锁的释放，其实都是在`release(int)`中进行的，只不过参数不同，该方法如下：

```
public void release(int flags) {
    synchronized (mToken) {
        //内部计数-1
        mInternalCount--;
        //如果释放超时锁，外部计数-1
        if ((flags & RELEASE_FLAG_TIMEOUT) == 0) {
            mExternalCount--;
        }
        //如果释放非计数锁或内部计数为0,并且该锁还在持有,则通过PowerManagerService去释放
        if (!mRefCounted || mInternalCount == 0) {
            mHandler.removeCallbacks(mReleaser);
            if (mHeld) {
                Trace.asyncTraceEnd(Trace.TRACE_TAG_POWER, mTraceName, 0);
                try {
                    mService.releaseWakeLock(mToken, flags);
                } catch (RemoteException e) {
                    throw e.rethrowFromSystemServer();
                }
                //表示不持有该锁
                mHeld = false;
            }
        }
        //如果时计数锁，并且外部计数小于0,则抛出异常
        if (mRefCounted && mExternalCount < 0) {
            throw new RuntimeException("WakeLock under-locked " + mTag);
        }
    }
}

```
对于计数锁的释放，每次都会对内部计数值减一，只有当你内部计数值减为0时，才会去调用`PowerManagerService`去真正的释放锁；如果释放非计数锁，则每次都会调用`PowerManagerService`进行释放。

WakeLock类型及其特点见下表：

Level	|CPU	|Screen	|Keyboard	|Power Button|	备注
----|----|----|----|----|----
PARTIAL_WAKE_LOCK|	On	|On/Off	|On/Off	|No Influence|	Cpu不受power键影响，一直运行直到所有锁被释放
FULL_WAKE_LOCK|	On	|Bright	|Bright|	release|	API17中已启用，改用WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON,当用户在不同的应用程序间切换时可以正确的管理，同时不需要权限
SCREEN_DIM_WAKE_LOCK|	On|	Dim/Bright|	Off	|release	|同上
SCREEN_BRIGHT_WAKE_LOCK	|On|	Bright	|Off	|release	|同上
PROXIMITY_SCREEN_OFF_WAKE_LOCK|	On/Off	|Bright/off	|Off	|relesase	|不能和ACQUIRE_CAUSES_WAKEUP一起使用
DOZE_WAKE_LOCK	|On/Off	|Off	|Off	|release|	@hide，允许在doze状态下使cpu进入suspend状态，仅在doze状态下有效，需要android.Manifest.permission.DEVICE_POWER权限
DRAW_WAKE_LOCK|	On/Off	|Off	|Off|	No|	@hide,允许在doze状态下进行屏幕绘制，仅在doze状态下有效，需要DEVICE_POWER权限
ACQUIRE_CAUSES_WAKEUP|||||	Wakelock 标记，一般情况下，获取wakelock并不能唤醒设备，加上这个标志后，申请wakelock后也会唤醒屏幕。如通知、闹钟…				不能和PARTIAL_WAKE_LOCK一起使用
ON_AFTER_RELEASE|||||	Wakelock 标记，当释放该标记的锁时，会亮一小会再灭屏				同上

源码注释如下：
```
/**
 * The following wake lock levels are defined, with varying effects on system power.
 * <i>These levels are mutually exclusive - you may only specify one of them.</i>
 *
 * <table>
 *     <tr><th>Flag Value</th>
 *     <th>CPU</th> <th>Screen</th> <th>Keyboard</th></tr>
 *
 *     <tr><td>{@link #PARTIAL_WAKE_LOCK}</td>
 *         <td>On*</td> <td>Off</td> <td>Off</td>
 *     </tr>
 *
 *     <tr><td>{@link #SCREEN_DIM_WAKE_LOCK}</td>
 *         <td>On</td> <td>Dim</td> <td>Off</td>
 *     </tr>
 *
 *     <tr><td>{@link #SCREEN_BRIGHT_WAKE_LOCK}</td>
 *         <td>On</td> <td>Bright</td> <td>Off</td>
 *     </tr>
 *
 *     <tr><td>{@link #FULL_WAKE_LOCK}</td>
 *         <td>On</td> <td>Bright</td> <td>Bright</td>
 *     </tr>
 * </table>
 * </p><p>
 * *<i>If you hold a partial wake lock, the CPU will continue to run, regardless of any
 * display timeouts or the state of the screen and even after the user presses the power button.
 * In all other wake locks, the CPU will run, but the user can still put the device to sleep
 * using the power button.</i>
***/
```
对于开发者来说，只能申请非@hide的锁，即`PARTIAL_WAKE_LOCK`、`SCREEN_DIM_WAKE_LOCK`、`SCREEN_BRIGHT_WAKE_LOCK`、`FULL_WAKE_LOCK`四类。

比如，我要获取一个`wakelock`类型为`PARTIAL_WAKE_LOCK`的`WakeLock`锁，则在申请这个锁后，虽然屏幕、键盘灯可以关闭，但CPU将一直处于活动状态，不受power键的控制。获得WakeLock对象后，可以根据自己的需求来申请不同形式的锁。接下来我们继续分析在申请、释放锁时`PowerManagerService`中的流程。

#### PowerManagerService中的WakeLock

`WakeLock`申请

在申请`WakeLock`时，当应用层调用完`acquire()`方法后，由`PowerManager`去处理了。对于两种申请方式，最终都调用了`acquireLocked()`进行申请，`acquireLocked()`又向下调用，让`mService`去处理，我们通过上面的分析知道，这个`mService`就是`PMS.BinderService`,该方法如下：
```
private void acquireLocked() {
    if (!mRefCounted || mCount++ == 0) {
        mHandler.removeCallbacks(mReleaser);
        try {
        	//向下调用PMS去处理
            mService.acquireWakeLock(mToken, mFlags, mTag, 
                         mPackageName, mWorkSource,mHistoryTag);
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
        mHeld = true;//持有锁标记置为true
    }

```
其中`mHeld`是一个判断是否持有锁的标记，在应用中可以通过调用`WakeLock`的`isHeld()`来判断是否持有`WakeLock`。

`PMS`中的`acquireWakeLock()`方法如下：
```
@Override // Binder call
public void acquireWakeLock(IBinder lock, int flags, String tag, 
                     String packageName,WorkSource ws, String historyTag) {
	......
    //检查wakelock级别
    PowerManager.validateWakeLockParameters(flags, tag);
	//检查WAKE_LOCK权限
mContext.enforceCallingOrSelfPermission(android.Manifest.permission.WAKE_LO   
                        CK, null);
    //如果是DOZE_WAKE_LOCK级别wakelock,还要检查DEVICE_POWER权限
    if ((flags & PowerManager.DOZE_WAKE_LOCK) != 0) {
        mContext.enforceCallingOrSelfPermission(
                android.Manifest.permission.DEVICE_POWER, null);
    }
    //ws = null
	......
    //重置当前线程上传入的IPC标志
    final long ident = Binder.clearCallingIdentity();
    try {
        acquireWakeLockInternal(lock, flags, tag, packageName, ws, historyTag,
                  uid, pid);
    } finally {
        Binder.restoreCallingIdentity(ident);
    }
}

```
在这个方法中，首先进行了`WakeLock`类型检查，避免无效的`WakeLock`类型，然后进行权限的检查，`WakeLock`需要`android.Manifest.permission.WAKE_LOCK`权限，如果申请的`WakeLock`类型是`DOZE_WAKE_LOCK`，则还需要`android.Manifest.permission.DEVICE_POWER`权限(见上表)，检查完毕后重置`Binder`的`IPC`标志，然后调用下一个方法`acquireWakeLockInternal()`：
```
private void acquireWakeLockInternal(IBinder lock, int flags, String tag, String packageName,
        WorkSource ws, String historyTag, int uid, int pid) {
    synchronized (mLock) {
        //PMS中的WakeLock类
        WakeLock wakeLock;
        //查找是否已存在该PM.WakeLock实例
        int index = findWakeLockIndexLocked(lock);
        boolean notifyAcquire;
        //是否存在wakelock
        if (index >= 0) {
            wakeLock = mWakeLocks.get(index);
            if (!wakeLock.hasSameProperties(flags, tag, ws, uid, pid)) {
            	//更新wakelock
                notifyWakeLockChangingLocked(wakeLock, flags, tag, packageName,
                        uid, pid, ws, historyTag);
                wakeLock.updateProperties(flags, tag, packageName, 
                                ws, historyTag, uid, pid);
            }
            notifyAcquire = false;
        } else {
              //从SpareArray<UidState>中查找是否存在该uid
              UidState state = mUidState.get(uid);
              if (state == null) {
                  state = new UidState(uid);
                  //设置该Uid的进程状态
                  state.mProcState = ActivityManager.PROCESS_STATE_NONEXISTENT;
                  mUidState.put(uid, state);
              }
            //将该uid申请的WakeLock计数加1
            //创建新的PMS.WakeLock实例
            wakeLock = new WakeLock(lock, flags, tag, packageName, ws, 
                              historyTag, uid, pid);
            try {
                lock.linkToDeath(wakeLock, 0);
            } catch (RemoteException ex) {
                throw new IllegalArgumentException("Wake lock is already dead.");
            }
            //添加到wakelock集合中
            mWakeLocks.add(wakeLock);
            //用于设置PowerManger.PARTIAL_WAKE_LOCK能否可用
            //1.缓存的不活动进程不能持有wakelock锁               
            //2.如果处于idle模式，则会忽略掉所有未处于白名单中的应用申请的锁
            setWakeLockDisabledStateLocked(wakeLock);
            //表示有新的wakelock申请了
            notifyAcquire = true;
        }
        //判断是否直接点亮屏幕，如果带有点亮屏幕标志值，并且wakelock类型为
        //FULL_WAKE_LOCK,SCREEN_BRIGHT_WAKE_LOCK,SCREEN_DIM_WAKE_LOCK,则进行下 
        //步处理
        applyWakeLockFlagsOnAcquireLocked(wakeLock, uid);
        //更新标志位
        mDirty |= DIRTY_WAKE_LOCKS;
        updatePowerStateLocked();
        if (notifyAcquire) {
           //当申请了锁后，在该方法中进行长时锁的判断，通知BatteryStatsService      
           // 进行统计持锁时间等
            notifyWakeLockAcquiredLocked(wakeLock);
        }
    }
}

```
首先通过传入的第一个参数`IBinder`进行查找`WakeLock`是否已经存在，若存在，则不再进行实例化，在原有的`WakeLock`上更新其属性值；若不存在，则创建一个`WakeLock`对象，同时将该`WakeLock`保存到`List`中。此时已经获取到了`WakeLock`对象，这里需要注意的是，此处的`WakeLock`对象和`PowerManager`中获取的不是同一个`WakeLock`哦！

获取到`WakeLock`实例后，还通过`setWakeLockDisabledStateLocked(wakeLock)`进行了判断该WakeLock是否可用，主要有两种情况：

1.缓存的不活动进程不能持有WakeLock锁；
2.如果处于idle模式，则会忽略掉所有未处于白名单中的应用申请的锁。
根据情况会设置`WakeLock`实例的`disable`属性值表示该`WakeLock`是否不可用。

下一步进行判断是否直接点亮屏幕，如果获得的`WakeLock`带有`ACQUIRE_CAUSES_WAKEUP`标志，并且`WakeLock`类型为`FULL_WAKE_LOCK,SCREEN_BRIGHT_WAKE_LOCK,SCREEN_DIM_WAKE_LOCK`这三种其中之一(`isScreenLock`判断)，则会直接唤醒屏幕，如下代码中的`applyWakeLockFlagsOnAcquireLocked(wakeLock, uid)`方法：

```
private void applyWakeLockFlagsOnAcquireLocked(WakeLock wakeLock, int uid) {
    if ((wakeLock.mFlags & PowerManager.ACQUIRE_CAUSES_WAKEUP) != 0
            && isScreenLock(wakeLock)) {
		......
        wakeUpNoUpdateLocked(SystemClock.uptimeMillis(), wakeLock.mTag, opUid,
                opPackageName, opUid);
    }
}

```
在wakeUpNoUpdateLocked()中：
```
private boolean wakeUpNoUpdateLocked(long eventTime, String reason, int reasonUid,
        String opPackageName, int opUid) {
//如果eventTime<上次休眠时间、设备当前处于唤醒状态、没有启动完成、没有准备
//完成，则不需要更新，返回false
    if (eventTime < mLastSleepTime || mWakefulness == WAKEFULNESS_AWAKE
            || !mBootCompleted || !mSystemReady) {
        return false;
    }
        //更新最后一次唤醒时间值
        mLastWakeTime = eventTime;
   //设置wakefulness
        setWakefulnessLocked(WAKEFULNESS_AWAKE, 0);
        //通知BatteryStatsService/AppService屏幕状态发生改变
        mNotifier.onWakeUp(reason, reasonUid, opPackageName, opUid);
    //更新用户活动事件时间值
        userActivityNoUpdateLocked(
                eventTime, PowerManager.USER_ACTIVITY_EVENT_OTHER, 0, reasonUid);
    } finally {
        Trace.traceEnd(Trace.TRACE_TAG_POWER);
    }
    return true;
}

```
`wakeUpNoUpdateLocked()`方法是唤醒设备的主要方法。在这个方法中，首先更新了`mLastWakeTime`这个值，表示上次唤醒设备的时间，在系统超时休眠时用到这个值进行判断。现在，只需要知道每次亮屏，都走的是这个方法，关于具体是如何唤醒屏幕的，在第5节中进行分析。

现在我们继续回到`acquireWakeLockInternal()`方法的结尾处，当检查完`WakeLock`的`ACQUIRE_CAUSES_WAKEUP`标志后，更新`mDirty`，然后调用`updatePowerStateLocked()`方法，这个方法在第二篇文章中说过了，是整个`PMS的核心方法`，在这个方法中调用了几个关键方法，这些方法已经进行了分析，只剩一个`WakeLock`相关的`updateSuspendBlockerLocked()`没有分析，现在开始分析这个方法，该方法如下：
```
private void updateSuspendBlockerLocked() {
	//是否需要保持CPU活动状态的SuspendBlocker锁，具体表现为持有Partical WakeLock
final boolean needWakeLockSuspendBlocker = 
           ((mWakeLockSummary & WAKE_LOCK_CPU) != 0);
    //是否需要保持CPU活动状态的SuspendBlocker锁，具体表现保持屏幕亮
    final boolean needDisplaySuspendBlocker = needDisplaySuspendBlockerLocked();
    //是否自动挂起，如果不需要屏幕保持唤醒，则说明可以自动挂起CPU
    final boolean autoSuspend = !needDisplaySuspendBlocker;
    //是否处于交互模式，屏幕处于Bright或者Dim状态时为true
    final boolean interactive = mDisplayPowerRequest.isBrightOrDim();
    //mDecoupleHalAutoSuspendModeFromDisplayConfig:自动挂起模式和显示状态解偶
    if (!autoSuspend && mDecoupleHalAutoSuspendModeFromDisplayConfig) {
    	//禁止CPU自动挂起模式
        setHalAutoSuspendModeLocked(false);
    }
    //如果存在PARTIAL_WAKE_LOCK类型的WakeLock,申请mWakeLockSuspendBlocker锁
    if (needWakeLockSuspendBlocker && !mHoldingWakeLockSuspendBlocker) {
        mWakeLockSuspendBlocker.acquire();
        mHoldingWakeLockSuspendBlocker = true;
    }
    //如果当前屏幕需要保持亮屏，申请mDisplaySuspendBlocker锁
    if (needDisplaySuspendBlocker && !mHoldingDisplaySuspendBlocker) {
        mDisplaySuspendBlocker.acquire();
        mHoldingDisplaySuspendBlocker = true;
    }
    //？？？
    if (mDecoupleHalInteractiveModeFromDisplayConfig) {
    	//设置Hal层交互模式?
        if (interactive || mDisplayReady) {
        	//？？？
            setHalInteractiveModeLocked(interactive);
        }
    }
    //如果不再持有PARTIAL_WAKELOCK类型的WakeLock锁，释放mWakeLockSuspendBlocker锁
    if (!needWakeLockSuspendBlocker && mHoldingWakeLockSuspendBlocker) {
        mWakeLockSuspendBlocker.release();
        mHoldingWakeLockSuspendBlocker = false;
    }
    //如果不再需要屏幕保持亮屏，释放mDisplaySuspendBlocker锁
    if (!needDisplaySuspendBlocker && mHoldingDisplaySuspendBlocker) {
        mDisplaySuspendBlocker.release();
        mHoldingDisplaySuspendBlocker = false;
    }
	//启动自动挂起模式
    if (autoSuspend && mDecoupleHalAutoSuspendModeFromDisplayConfig) {
        setHalAutoSuspendModeLocked(true);
    }
}

```
在`updateSuspendBlockerLocked()`方法中，会根据当前系统是否持有`PARTIAL_WAKELOCK`类型的锁，来决定是否要申请或释放`mWakeLockSuspendBlocker`锁，然后会根据当前系统是否要屏幕亮屏来决定是否要申请或释放`mDisplaySuspendBlocker`锁。

我们在分析PMS的启动时提到过，在PMS的构造方法中创建了两个`SuspendBlocker`对象：`mWakeLockSuspendBlocker`和`mDisplaySuspendBlocker`，前者表示获取一个`PARTIAL_WAKELOCK`类型的`WakeLock`使CPU保持活动状态，后者表示当屏幕亮屏、用户活动时使CPU保持活动状态。因此实际上，上层`PowerManager`申请和释放锁，最终在PMS中都交给了`SuspendBlocker`去申请和释放锁。也可以说`SuspendBlocker`类的两个对象是`WakeLock`锁反映到底层的对象。只要持有二者任意锁，都会使得CPU处于活动状态。

NOTE 这里还有两个值含义很重要：
**mDecoupleHalAutoSuspendModeFromDisplayConfig：**
该值表示是否将自动挂起状态和设备的屏幕显示状态(开/关)进行解耦。如果为`false`，则`autosuspend_disable()`函数会在屏幕亮之前调用，`autosuspend_enable()`函数会在屏幕灭之后调用，这种模式为使用传统电源管理功能的设备（如提前暂停/延迟恢复）提供最佳兼容性。如果为`true`，则`autosuspend_display()`和`autosuspend_enable()`将会被单独调用，不管屏幕正在亮或灭，这种模式使电源管理器可以在屏幕打开时挂起应用程序处理器。当指定了`dozeComponent`组件时，应将此资源设置为“true”，最大限度地节省电力，但并非所有设备都支持它。

**mDecoupleHalInteractiveModeFromDisplayConfig:**
表示是否将交互状态和设备的屏幕显示状态进行解耦。
如果为`false`，则当屏幕亮时调用`setInteractive(…, true)`函数，当屏幕灭时调用`setInteractive(…, false)`函数，此模式为希望将交互式状态绑定到显示状态的设备提供最佳兼容性。
如果为`true`，`setInteractive()`函数将会独立地调用，不管屏幕处于亮或灭，这种模式使电源管理器可以在显示器打开时减少时钟并禁用触摸控制器
再来看看`needDisplaySuspendBlockerLocked()`的实现：
```
    private boolean needDisplaySuspendBlockerLocked() {
        //mDisplayReady表示是否显示器准备完毕
        if (!mDisplayReady) {
            return true;
        }
        //请求Display策略状态为Bright或DIM，这个if语句用来判断当PSensor灭屏时是否需要Display锁
        if (mDisplayPowerRequest.isBrightOrDim()) {
            // If we asked for the screen to be on but it is off due to the proximity
            // sensor then we may suspend but only if the configuration allows it.
            // On some hardware it may not be safe to suspend because the proximity
            // sensor may not be correctly configured as a wake-up source.
            //如果没有PROXIMITY_SCREEN_OFF_WAKE_LOCK类型的WakeLock锁||PSensor正在处于远离状态
            //或在PSensor灭屏后不允许进入Suspend状态，满足之一，则申请misplaySuspendBlocker锁
            if (!mDisplayPowerRequest.useProximitySensor || !mProximityPositive
                    || !mSuspendWhenScreenOffDueToProximityConfig) {
                return true;
            }
        }
        if (mScreenBrightnessBoostInProgress) {
            return true;
        }
        // Let the system suspend if the screen is off or dozing.
        return false;
    }

```
**SuspendBlocker**是一个接口，并且只有`acquire()`和`release()`两个方法，`PMS.SuspendBlockerImpl`实现了该接口，因此，最终申请流程执行到了`PMS.SuspendBlockerImpl`的`acquire()`中。

在`PMS.SuspendBlockerImpl.acquire()`中进行申请时，首先将成员变量计数加1，然后调用到JNI层去进行申请，对应的JNI层文件路径为：`frameworks\base\services\core\jni\com_android_server_power_PowerManagerService.cpp`,具体代码如下：

```
@Override
public void acquire() {
    synchronized (this) {
        //引用计数
        mReferenceCount += 1;  
        if (mReferenceCount == 1) {
            nativeAcquireSuspendBlocker(mName);
        }
    }
}

```
这里使用了引用计数法，如果`mReferenceCount >1`,则不会进行锁的申请，而是仅仅将`mReferenceCount +1`，只有当没有申请的锁时，才会其正真执行申请锁操作，之后不管申请几次，都是`mReferenceCount` 加1.
在JNI层中可以明确的看到有一个申请锁的`acquire_wake_lock()`方法，代码如下：
```
static void nativeAcquireSuspendBlocker(JNIEnv *env, jclass /* clazz */, jstring nameStr) {
    ScopedUtfChars name(env, nameStr);
    acquire_wake_lock(PARTIAL_WAKE_LOCK, name.c_str());
}

```
这个方法位于HAL层，实现在`/hardware/libhardware_legacy/power/power.c` 中，先看看具体代码：
```
int acquire_wake_lock(int lock, const char* id)
{
    initialize_fds();
    ALOGI("acquire_wake_lock lock=%d id='%s'\n", lock, id);
    if (g_error) return g_error;
    int fd;
    size_t len;
    ssize_t ret;
    if (lock != PARTIAL_WAKE_LOCK) {
        return -EINVAL;
    }
    fd = g_fds[ACQUIRE_PARTIAL_WAKE_LOCK];
    ret = write(fd, id, strlen(id));
    if (ret < 0) {
        return -errno;
    }
    return ret;
}

```
在这里，向`/sys/power/wake_lock`文件写入了`id`,这个id就是我们上层中实例化`SuspendBlocker`时传入的`String`类型的`name`，这里在这个节点写入文件以后，就说明获得了`wakelock`。可通过adb命令查看该文件：
```
$ adb root
adbd is already running as root
$ adb remount
remount succeeded
$ adb shell cat /sys/power/wake_lock
PowerManagerService.Display
$ 

```
到这里，整个`WakeLock`的申请流程就结束了。

现在还有一点剩余代码需要看看，继续回到`PMS`的`acquireWakeLockInternal()`方法中，当执行完`updatePowerStateLocked()`方法后，如果有新的`WakeLock`实例创建，则`notifyAcquire`值为`true`，通过以下这个方法通知`Notifier`  ，`Notifier`中则会根据该锁申请的时间开始计时，并以此来判断是否是一个长时间持有的锁：
```
private void notifyWakeLockAcquiredLocked(WakeLock wakeLock) {
    if (mSystemReady && !wakeLock.mDisabled) {
        wakeLock.mNotifiedAcquired = true;
        wakeLock.mStartTimeStamp = SystemClock.elapsedRealtime();
        //Called when a wake lock is acquired.
        mNotifier.onWakeLockAcquired(wakeLock.mFlags, 
                        wakeLock.mTag, wakeLock.mPackageName,
                wakeLock.mOwnerUid, wakeLock.mOwnerPid, wakeLock.mWorkSource,
                wakeLock.mHistoryTag);
        ......
        //重新开始检查持锁时间
        restartNofifyLongTimerLocked(wakeLock);
    }
}

```
申请`WakeLock`时序图如下：
![](https://upload-images.jianshu.io/upload_images/5851256-029f2c6c2dc702be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
WakeLock的释放

当应用持有`WakeLock`锁、更准确地说是`PARTIAL_WAKELOCK`类型的`WakeLock`锁，执行完相应的任务后，要及时对其进行释放，否则会导致设备一直处于唤醒状态，CPU无法休眠，造成电量消耗过快。`WakeLock`的释放流程和申请流程很类似，我们还是从应用层开始逐步分析.

应用中执行释放锁操作的前提是该应用当前持有一个锁，通过`PowerManager.WakeLock`调用`release()`进行释放，在对应用层开放的`API`中，有两种释放形式，除了最常用的直接调用`release()`方法，还有一种是调用`release(int flag)`方法，这个方法允许用户传入一个标志值，以达到修改释放行为的目的，目前而言（API 27），只支持一个值：
```
public static final int RELEASE_FLAG_WAIT_FOR_NO_PROXIMITY = 1;
```
表示延迟释放 `PROXIMITY_SCREEN_OFF_WAKE_LOCK`类型的`WakeLock`(该类型的`WakeLock`最常见于通话过程中，为了有更好的用户体验，在脸靠近屏幕时防止误触，屏幕会熄灭，远离屏幕时屏幕又会开启，就是使用了该`WakeLock`)，传入`0`相当于调用`release()`,具体代码如下：

```
public void release() {
    release(0);
}
public void release(int flags) {
    synchronized (mToken) {
        if (!mRefCounted || --mCount == 0) {
            mHandler.removeCallbacks(mReleaser);
            if (mHeld) {
                try {
                    //PMS中进行处理
                    mService.releaseWakeLock(mToken, flags);
                } catch (RemoteException e) {
                }
                mHeld = false;
            }
        }
        if (mCount < 0) {
            throw new RuntimeException("WakeLock under-locked " + mTag);
        }
    }
}

```
在`release()`中，向下调用了`PMS.BinderService`的`releaseWakeLock()`方法：
```
@Override // Binder call
public void releaseWakeLock(IBinder lock, int flags) {
    if (lock == null) {
        throw new IllegalArgumentException("lock must not be null");
    }
    //检查权限
    mContext.enforceCallingOrSelfPermission(android.Manifest.permission.
         WAKE_LOCK, null);
 	//重置当前线程的IPC标志
    final long ident = Binder.clearCallingIdentity();
    try {
        //去释放锁
        releaseWakeLockInternal(lock, flags);
    } finally {
    	//设置新的IPC标志
        Binder.restoreCallingIdentity(ident);
    }
}

```
在这个方法中，进行了权限检查后，就交给下一个方法去处理了，具体代码如下：
```
private void releaseWakeLockInternal(IBinder lock, int flags) {
    synchronized (mLock) {
    	//查找WakeLock是否存在
        int index = findWakeLockIndexLocked(lock);
        if (index < 0) {
            return;
        }
        WakeLock wakeLock = mWakeLocks.get(index);
            //该flag用来推迟释放PowerManager.PROXIMITY_SCREEN_OFF_WAKE_LOCK类型的锁，它会在传感器感觉不在靠近的时候才释放该锁
            if ((flags & PowerManager.RELEASE_FLAG_WAIT_FOR_NO_PROXIMITY) != 0) {
                //表示在点亮屏幕前需要等待PSensor返回负值
                mRequestWaitForNegativeProximity = true;
            }
        if ((flags & PowerManager.RELEASE_FLAG_WAIT_FOR_NO_PROXIMITY) != 0) {
            mRequestWaitForNegativeProximity = true;
        }
        //取消Binder讣告
        wakeLock.mLock.unlinkToDeath(wakeLock, 0);
        //释放锁
        removeWakeLockLocked(wakeLock, index);
    }
}

```
在`releaseWakeLockInternal()`中处理时，首先查找WakeLock是否存在，若不存在，直接返回；然后检查是否带有影响释放行为的标志值，上面已经提到过，目前只有一个值，之后取消了`Binder`的死亡代理，最后调用了`removeWakeLockLocked()`方法：
```
private void removeWakeLockLocked(WakeLock wakeLock, int index) {
    //从List中移除
    mWakeLocks.remove(index);
    //得到该wakelock中的UidState属性
    UidState state = wakeLock.mUidState;
    state.mNumWakeLocks--;
    if (state.mNumWakeLocks <= 0 &&
            state.mProcState == ActivityManager.PROCESS_STATE_NONEXISTENT) {
        //从SpareArray<UidState>中移除该wakelock的UidState
        //注意,下面的mUidState是SpareArray<UidState>，而上面的mUidState是wakeLock.mUidState
        mUidState.remove(state.mUid);
    }
    //使用Notifier通知其他应用
    notifyWakeLockReleasedLocked(wakeLock);
    //对带有ON_AFTER_RELEASE标志的wakelock进行处理
    applyWakeLockFlagsOnReleaseLocked(wakeLock);
    mDirty |= DIRTY_WAKE_LOCKS;
    //更新电源状态信息
    updatePowerStateLocked();
}

```
在`removeWakeLockLocked()`中，对带有`ON_AFTER_RELEASE`标志的`wakelock`进行处理，前面分析过了，该标志和用户体验相关，当有该标志时，释放锁后会亮一段时间后灭屏，这里来看看`applyWakeLockFlagsOnReleaseLocked(wakeLock)`方法：
```
    /**
     *如果当前释放的wakelock带有PowerManager.ON_AFTER_RELEASE标志，则会屏幕在灭屏时小亮一会儿才会熄灭
     */
    private void applyWakeLockFlagsOnReleaseLocked(WakeLock wakeLock) {
        if ((wakeLock.mFlags & PowerManager.ON_AFTER_RELEASE) != 0
                && isScreenLock(wakeLock)) {
            //更新用户活动时间，并带有PowerManager.USER_ACTIVITY_FLAG_NO_CHANGE_LIGHTS标志,用于延缓灭屏时间
            userActivityNoUpdateLocked(SystemClock.uptimeMillis(),
                    PowerManager.USER_ACTIVITY_EVENT_OTHER,
                    PowerManager.USER_ACTIVITY_FLAG_NO_CHANGE_LIGHTS,
                    wakeLock.mOwnerUid);
        }
    }

```
最后，又将调用`updatePowerStateLocked()`，其中和`WakeLock`申请和释放相关的都`updateSuspendBlockerLocked()`中，释放相关代码如下：
```
    //满足两个条件，释放"PowerManagerService.WakeLocks"锁
    if (!needWakeLockSuspendBlocker && mHoldingWakeLockSuspendBlocker) {
        mWakeLockSuspendBlocker.release();
        mHoldingWakeLockSuspendBlocker = false;
    }
    //释放"PowerManagerService.Display"锁
    if (!needDisplaySuspendBlocker && mHoldingDisplaySuspendBlocker) {
        mDisplaySuspendBlocker.release();
        mHoldingDisplaySuspendBlocker = false;
    }

```
如果满足条件，则释放`SuspendBlocker`锁。申请`SuspendBlocker`流程已经分析过了，接下来我们分析释放`SuspendBlocker`流程。在`SuspendBlocker`中释放锁如下：
```
@Override
public void release() {
    synchronized (this) {
    	//计数-1
        mReferenceCount -= 1;
        if (mReferenceCount == 0) {
        	//调用JNI层进行释放
            nativeReleaseSuspendBlocker(mName);
        } else if (mReferenceCount < 0) {
            mReferenceCount = 0;
        }
    }
}

```
在释放锁时，如果有多个锁，实际上是对锁计数的属性减1，直到剩余一个时才会调用JNI层执行释放操作。JNI层对应的文件也在`com_android_server_power_PowerManagerService.cpp`中，具体代码如下：
```
static void nativeReleaseSuspendBlocker(JNIEnv *env, jclass /* clazz */, jstring nameStr) {
    ScopedUtfChars name(env, nameStr);
    release_wake_lock(name.c_str());
}

```
在JNI层方法中，调用了HAL层的方法，通过文件描述符向`/sys/power/wake_unlock`中写值完成释放：
```
int release_wake_lock(const char* id)
{
    initialize_fds();
    //    ALOGI("release_wake_lock id='%s'\n", id);
    if (g_error) return g_error;
    ssize_t len = write(g_fds[RELEASE_WAKE_LOCK], id, strlen(id));
    if (len < 0) {
        return -errno;
    }
    return len;
}

```
到这里为止，`WakeLock`的释放流程也就分析完毕了。

通过对`WakeLock`锁的申请和释放流程分析，知道实际上通过操作`/sys/power/wake_lock`和 `/sys/power/wake_unlock`节点来控制设备的唤醒和休眠，当应用需要唤醒设备时，申请一个`WakeLock`锁，最终会在`/sys/power/wake_lock` 中写入`SuspendBlocker`锁名，从而保持了设备的唤醒；当应用执行完操作后，则释放`WakeLock`锁，最终会在`/sys/power/wake_unlock` 中写入`SuspendBlocker`锁名。
整个`wakelock`锁释放的时序图如下：
![](https://upload-images.jianshu.io/upload_images/5851256-4992fcfff4c08284.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
再谈`WakeLock`锁和`SuspendBlocker`锁

通过对`Wakelock`锁的申请和释放，可以知道底层最终的锁都是`SuspendBlocker`锁，`SuspendBlocker`锁官方的解释是：**SuspendBlocker**相当于持有部分唤醒锁，该接口在内部使用，以避免在高级别唤醒锁机制上引入内部依赖关系。
当上层使用申请了`wakelock`锁后，最终反映在底层的都是`SuspendBlocker`锁，从`PowerManager到PowerManagerService再到HAL层`，其锁之间的关系如下图：
![](https://upload-images.jianshu.io/upload_images/5851256-8889f179bf0e49c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
再来看下在PMS中实例化的SuspendBloker：
```
//通过PARTIAL_WAKE_LOCK类型WakeLock锁使CPU激活
mWakeLockSuspendBlocker = createSuspendBlockerLocked("PowerManagerService.WakeLocks");
//通过屏幕亮屏使得CPU激活
mDisplaySuspendBlocker = createSuspendBlockerLocked("PowerManagerService.Display");
//防止在发送广播时CPU休眠
mNotifier = new Notifier(Looper.getMainLooper(), mContext, mBatteryStats,
        mAppOps, createSuspendBlockerLocked("PowerManagerService.Broadcasts"),
        mPolicy);

```
除了这三类锁，在通过adb命令查看`/sys/power/wake_unlock `还可以看得出其他两种`SuspendBlocker`锁：
```
$ adb shell cat /sys/power/wake_unlock
KeyEvents PowerManagerService.Broadcasts PowerManagerService.Display PowerManagerService.WakeLocks radio-interface
$

```
`mWakeLockSuspendBlocker`锁在`updateSuspendBlockerLocked()`方法中看出，`mWakeLockSuspendBlocker`申请的条件是：
```
final boolean needWakeLockSuspendBlocker = ((mWakeLockSummary & WAKE_LOCK_CPU) != 0);

```
`mWakeLockSummary`在第二篇文章中分析了，是一个汇总了所有`WakeLock`锁的二进制标志位，同时在第二篇文章的`updateWakeLockSummaryLocked（） `方法中分析了，满足以下条件时，会将`mWakeLockSummary`置位为`WAKE_LOCK_CPU`：
```
//updateWakeLockSummaryLocked（）中：
case PowerManager.PARTIAL_WAKE_LOCK:
case PowerManager.DRAW_WAKE_LOCK:
    mWakeLockSummary |= WAKE_LOCK_DRAW;
if (mWakefulness == WAKEFULNESS_AWAKE) {
    mWakeLockSummary |= WAKE_LOCK_CPU | WAKE_LOCK_STAY_AWAKE;
} else if (mWakefulness == WAKEFULNESS_DREAMING) {
    mWakeLockSummary |= WAKE_LOCK_CPU;
}
if ((mWakeLockSummary & WAKE_LOCK_DRAW) != 0) {
    mWakeLockSummary |= WAKE_LOCK_CPU;
}

```
因此，当申请了`PARTIAL_WAKE_LOCK` 类型的`WakeLock`锁、`DRAW_WAKE_LOCK`类型的WakeLock锁(前提是处于Doze模式)、屏幕处于唤醒、屏保时，都会持有一个`mWakeLockSuspendBlocker`锁，会在`/sys/power/wake_lock `节点中写入`”PowerManager.WakeLocks“`，从而保持设备处于唤醒状态。

##### mDisplaySuspendBlocker锁

在`updateSuspendBlockerLocked()`方法中，`mDisplaySuspendBlocker`申请的条件是：
```
final boolean needDisplaySuspendBlocker = needDisplaySuspendBlockerLocked();

private boolean needDisplaySuspendBlockerLocked() {
if (mDisplayPowerRequest.isBrightOrDim()) {//屏幕处于亮或Dim
    //没有使用PSensor || PSensor值为负值（靠近） || 配置PSensor灭屏时挂起CPU为false 
    if (!mDisplayPowerRequest.useProximitySensor || !mProximityPositive
            || !mSuspendWhenScreenOffDueToProximityConfig) {
        return true;
    }
}
......
return false;
}

```
因此，只要在屏幕处于亮或Dim状态时，满足其中之一项，就可以申请`mDisplaySuspendBlocker`锁，会向`/sys/power/wake_lock `中写入`”PowerManagerService.Display“`。
当屏幕处于亮屏或Dim状态时，一定持有`mDisplaySuspendBlocker`锁

#####`PowerManagerService.Broadcasts`锁

这个类型的`SuspendBlocker`并没有在PMS中进行实例化，它以构造方法的形式传入了`Notifier`中，Notifier类相当于是PMS的”中介“，PMS中和其他服务的部分交互通过Notifier进行，还有比如亮屏广播、灭屏广播等，都是由PMS交给Notifier来发送，这点在下篇文章中进行分析。因此，如果CPU在广播发送过程中进入休眠，则广播无法发送完成，因此，需要一个锁来保证Notifier中广播的成功发送，这就是`PowerManagerService.Broadcasts` 锁的作用，当广播发送完毕后，该锁立即就释放了。



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

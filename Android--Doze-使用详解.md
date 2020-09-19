

 

##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 Android 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、Doze 模式
>二、空闲状态下，优化app耗电
>三、Doze 模式下的限制措施
>四、Doze 模式概要
>五、Doze 模式涉及的类如下:
>六、Doze 模式状态
>七、Doze 白名单
>八、Doze 模式测试方法
>九、开启Doze dubug 调试开关





  # 一、Doze 模式

当设备处于非充电、灭屏状态下静止一段时间，设备将进入睡眠状态，进入`Doze`模式，延长电池使用时间。`Doze `模式下系统会定期恢复正常操作，异步执行`ap`p的一些同步数据等操作。比如很长时间不使用，系统会允许设备一天访问一次网络等。当设备处于充电状态下，系统将进入标准模式，`app`执行操作将不被限制。

#二、空闲状态下，优化app耗电

在用户没有使用`app`的情况下，系统会使`app`处于`idle` 状态,
在空闲状态下，系统将会禁止`app `网络访问以及数据同步

#三、Doze 模式下的限制措施
	   
 1.禁止网络访问
 2.忽略`Wake lock`
 3.忽略`Alarms(setAlarmClock() 、AlarmManager.setAndAllowwhileIdle()` 这两个方法除外)
 4.忽略` WIFI` 扫描
 5.同步作业调度程序将不被执行
   
#四、Doze 模式概要
![Doze模式概要](http://upload-images.jianshu.io/upload_images/5851256-639d51c59af329bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#五、Doze 模式涉及的类如下:
`frameworks/base/services/core/java/com/android/server/DeviceIdleController.java`
```
   /**
  * Keeps track of device idleness and drives low power mode based on that.
  */
   public class DeviceIdleController extends SystemService
        implements AnyMotionDetector.DeviceIdleCallback {
```
#六、Doze 模式状态
+ ACTIVE：手机设备处于激活活动状态
+ INACTIVE：屏幕关闭进入非活动状态
+ IDLE_PENDING：每隔30分钟让App进入等待空闲预备状态
+ IDLE：空闲状态
+ IDLE_MAINTENANCE：处理挂起任务

![ Doze 模式状态](http://upload-images.jianshu.io/upload_images/5851256-7f870c91782415cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对应的 Doze 模式状态如下： 
 ![Doze模式状态图](http://upload-images.jianshu.io/upload_images/5851256-44645870b94b2a0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## active--->     inactive --->    idle_pending

运动模式检测
```
  void handleMotionDetectedLocked(long timeout, String type) {
        // The device is not yet active, so we want to go back to the pending idle
        // state to wait again for no motion.  Note that we only monitor for motion
        // after moving out of the inactive state, so no need to worry about that.
        boolean becomeInactive = false;
        if (mState != STATE_ACTIVE) {
            scheduleReportActiveLocked(type, Process.myUid());
            mState = STATE_ACTIVE;
            mInactiveTimeout = timeout;
            mCurIdleBudget = 0;
            mMaintenanceStartTime = 0;
            EventLogTags.writeDeviceIdle(mState, type);
            addEvent(EVENT_NORMAL);
            becomeInactive = true;
        }
        if (mLightState == LIGHT_STATE_OVERRIDE) {
            // We went out of light idle mode because we had started deep idle mode...  let's
            // now go back and reset things so we resume light idling if appropriate.
            mLightState = STATE_ACTIVE;
            EventLogTags.writeDeviceIdleLight(mLightState, type);
            becomeInactive = true;
        }
        if (becomeInactive) {
            becomeInactiveIfAppropriateLocked();
        }
    }
```
## idle_pending    ————>sensing
```

    @Override
    public void onAnyMotionResult(int result) {
        if (DEBUG) Slog.d(TAG, "onAnyMotionResult(" + result + ")");
        if (result != AnyMotionDetector.RESULT_UNKNOWN) {
            synchronized (this) {
                cancelSensingTimeoutAlarmLocked();
            }
        }
        if (result == AnyMotionDetector.RESULT_MOVED) {
            if (DEBUG) Slog.d(TAG, "RESULT_MOVED received.");
            synchronized (this) {
                handleMotionDetectedLocked(mConstants.INACTIVE_TIMEOUT, "sense_motion");
            }
        } else if (result == AnyMotionDetector.RESULT_STATIONARY) {
            if (DEBUG) Slog.d(TAG, "RESULT_STATIONARY received.");
            if (mState == STATE_SENSING) {
                // If we are currently sensing, it is time to move to locating.
                synchronized (this) {
                    mNotMoving = true;
                    stepIdleStateLocked("s:stationary");
                }
            } else if (mState == STATE_LOCATING) {
                // If we are currently locating, note that we are not moving and step
                // if we have located the position.
                synchronized (this) {
                    mNotMoving = true;
                    if (mLocated) {
                        stepIdleStateLocked("s:stationary");
                    }
                }
            }
        }
    }
```
# 七、Doze 白名单
电量优化白名单
设置 --电池 --电量优化（menu菜单）
会设置查看app 电池优化使用情况 
白名单是以xml形式存储(`deviceidle.xml)`
查看白名单命令
```
//主要存放app包名
adb shell dumpsys deviceidle whitelist
```
白名单代码保存部分代码如下
```

    /**
     * Package names the system has white-listed to opt out of power save restrictions,
     * except for device idle mode.
     */
    private final ArrayMap<String, Integer> mPowerSaveWhitelistAppsExceptIdle = new ArrayMap<>();

    /**
     * Package names the system has white-listed to opt out of power save restrictions for
     * all modes.
     */
    private final ArrayMap<String, Integer> mPowerSaveWhitelistApps = new ArrayMap<>();

    /**
     * Package names the user has white-listed to opt out of power save restrictions.
     */
    private final ArrayMap<String, Integer> mPowerSaveWhitelistUserApps = new ArrayMap<>();

    /**
     * App IDs of built-in system apps that have been white-listed except for idle modes.
     */
    private final SparseBooleanArray mPowerSaveWhitelistSystemAppIdsExceptIdle
            = new SparseBooleanArray();

    /**
     * App IDs of built-in system apps that have been white-listed.
     */
    private final SparseBooleanArray mPowerSaveWhitelistSystemAppIds = new SparseBooleanArray();

    /**
     * App IDs that have been white-listed to opt out of power save restrictions, except
     * for device idle modes.
     */
    private final SparseBooleanArray mPowerSaveWhitelistExceptIdleAppIds = new SparseBooleanArray();

    /**
     * Current app IDs that are in the complete power save white list, but shouldn't be
     * excluded from idle modes.  This array can be shared with others because it will not be
     * modified once set.
     */
    private int[] mPowerSaveWhitelistExceptIdleAppIdArray = new int[0];

    /**
     * App IDs that have been white-listed to opt out of power save restrictions.
     */
    private final SparseBooleanArray mPowerSaveWhitelistAllAppIds = new SparseBooleanArray();

    /**
     * Current app IDs that are in the complete power save white list.  This array can
     * be shared with others because it will not be modified once set.
     */
    private int[] mPowerSaveWhitelistAllAppIdArray = new int[0];

    /**
     * App IDs that have been white-listed by the user to opt out of power save restrictions.
     */
    private final SparseBooleanArray mPowerSaveWhitelistUserAppIds = new SparseBooleanArray();

    /**
     * Current app IDs that are in the user power save white list.  This array can
     * be shared with others because it will not be modified once set.
     */
    private int[] mPowerSaveWhitelistUserAppIdArray = new int[0];
```
#八、Doze 模式测试方法
1.开启Doze
`
adb shell dumpsys deviceidle enable`
`// or  MTK addadb shell setprop persist.config.AutoPowerModes 1
`
2.拔掉电池
`
adb shell dumpsys battery unplug 
`
3.调试Doze状态

` Active ---idle_pending---sensing--location---idle --idle_mantenance`

`
adb shell dumpsys deviceidle step
`
4.Dump Doze 状态分析

`Doze `模式下的信息，包括电池电量优化白名单等
`
adb shell dumpsys deviceidle
`
#九、开启Doze dubug 调试开关

 默认`false` 关闭，设置为`true ` 开启`DeviceIdleController.java`
  `  private static final boolean DEBUG = false;`



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

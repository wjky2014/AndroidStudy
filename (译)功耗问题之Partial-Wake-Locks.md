#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、发现问题
>二、解决问题
>三、验证问题
>四、最佳实践


**Partial wake locks** 是 [PowerManager](https://developer.android.com/reference/android/os/PowerManager.html) API 中的一种机制。可让开发人员在设备显示屏关闭（无论是由于系统超时还是用户按下电源按钮）之后，继续让`CPU`保持运行状态。

您的应用通过 [acquire()](https://developer.android.com/reference/android/os/PowerManager.WakeLock.html#acquire()) 使用 [PARTIAL_WAKE_LOCK](https://developer.android.com/reference/android/os/PowerManager.html#PARTIAL_WAKE_LOCK)  标志调用来获取部分唤醒锁。

如果部分唤醒锁 在您的应用程序在后台运行时被长时间*Hold，*则会 *stuck*（用户看不到应用程序的任何部分）。

这种情况会耗尽设备的电池电量，因为它会阻止设备进入低功耗模式。建议` Partial wake locks `应仅在必要时使用，并且再不需要时立即释放。

如果您的应用有 `Partial wake locks `卡住，则可以使用下文中的指导来诊断和解决问题。

# 一、发现问题

您可能并不总是知道`App`被 `Partial wake locks `卡住了。如果您已经发布了应用程序，则`Android vitals`可帮助您了解问题。

## Android vitals

当您的应用程序显示卡住的部分唤醒锁时 ，`Android vitals `可以通过[Play Console](https://play.google.com/apps/publish/signup/)提醒您，从而帮助提高应用程序的性能 。`Android vitals `主要报告 `partial wake locks `卡住至少一个小时以上的问题，比如：

- 至少占整个电池消耗的`0.70％`
或者
- 仅在后台运行时，至少有`0.10％`的电池电量消耗

电池电量是指两次充满电的间隔，显示的电池续航次数是该应用程序所有测量用户的汇总。有关`Google Play`如何收集`Android`生命数据的信息，请参阅 [Play Console](https://support.google.com/googleplay/android-developer/answer/7385505)文档。


一旦知道您的应用程序卡住了部分唤醒锁，下一步就是解决该问题。

# 二、解决问题

唤醒锁是在`Android`平台的早期版本中引入的，但是随着时间的推移，以前需要唤醒锁的许多用例现在可以通过诸如[WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager)更新的`API`更好地解决问题。

本节包含修复唤醒锁的技巧，但从长远来看，请考虑迁移您的应用程序，以遵循 **第四部分最佳实践** 中的建议。

在代码中标识并修复需要唤醒锁的位置，例如`newWakeLock(int ,String)`或`WakefulBroadcastReceiver(Android O 已弃用)`

- newWakeLock(int ,String)

```
 PowerManager pm = (PowerManager)mContext.getSystemService(
                                          Context.POWER_SERVICE);
 PowerManager.WakeLock wl = pm.newWakeLock(
                                      PowerManager.SCREEN_DIM_WAKE_LOCK
                                      | PowerManager.ON_AFTER_RELEASE,
                                      TAG);
 wl.acquire();
 // ... do work...
 wl.release();
 
```

## 1.建议在唤醒锁标记包名、类名、方法名

这样方便您可以轻松地在源码中标识创建唤醒锁的位置。以下是一些其他提示：
*   忽略名称中的任何个人身份信息（PII），例如电子邮件地址。否则，设备将记录日志，`_UNKNOWN` 而不是唤醒锁名称。

*   不要以编程方式获取类或方法的名称，例如通过调用 [getName()](https://developer.android.com/reference/java/lang/Class.html#getName()) ，因为`Proguard`可能会混淆它们。而是使用硬编码的字符串。

*   不要添加计数器或唯一标识符来唤醒锁定标签。系统将无法聚合通过相同方法创建的唤醒锁，因为它们都具有唯一的标识符。

## 2.确保您的代码释放了它获取的所有唤醒锁
准确的使用 [acquire()](https://developer.android.com/reference/android/os/PowerManager.WakeLock.html#acquire())及[release()](https://developer.android.com/reference/android/os/PowerManager.WakeLock.html#release())，避免异常导致无法是否唤醒锁。
* 举例由于未捕获的异常导致未释放的唤醒锁

```
    void doSomethingAndRelease() throws MyException {
        wakeLock.acquire();
        doSomethingThatThrows();
        wakeLock.release();  // does not run if an exception is thrown
    }
```
正确的写法应该如下
```
    void doSomethingAndRelease() throws MyException {
        try {
            wakeLock.acquire();
            doSomethingThatThrows();
        } finally {
            wakeLock.release();
        }
    }
```
## 3.确保唤醒锁在不再需要时被释放

如果您使用唤醒锁来允许后台任务完成，请确保该任务完成时及时释放唤醒锁。如果唤醒锁保持的时间比预期的要长而不被释放，则可能意味着您的后台任务花费的时间比预期的要多。

# 三、验证问题
解决了代码中的问题后，请使用以下Android工具验证您的应用正确释放了唤醒锁：
## 1.dumpsys
`dumpsys `提供设备上系统服务状态的信息。可以通过使用`adb `命令看电源服务的状态（包括唤醒锁列表）`adb shell dumpsys power`

```
C:\Users\Administrator>adb shell dumpsys power 
POWER MANAGER (dumpsys power)

Power Manager State:
  ... ...
  mBatteryLevel=46   // 电量信息
  mBatteryLevelWhenDreamStarted=0
  mDockState=0
  mStayOn=false
  mProximityPositive=false //Psensor
  mBootCompleted=true
  mSystemReady=true
  ... ... 

Settings and Configuration:
  
  ... ...
  mScreenBrightnessSettingMinimum=30  //屏幕连读参数
  mScreenBrightnessSettingMaximum=255
  mScreenBrightnessSettingDefault=102
  mScreenBrightnessForVrSettingDefault=86
  mScreenBrightnessForVrSetting=86
  mDoubleTapWakeEnabled=false
  mIsVrModeEnabled=false
  ... ...

Suspend Blockers: size=4  //唤醒中断源
  PowerManagerService.WakeLocks: ref count=0
  PowerManagerService.Display: ref count=0
  PowerManagerService.Broadcasts: ref count=0
  PowerManagerService.WirelessChargerDetector: ref count=0
  ... ...

```

## 2.Battery Historian

[Battery Historian](https://developer.android.com/topic/performance/power/battery-historian.html) 一种将 Android [Bugreport](https://developer.android.com/studio/debug/bug-report.html)的输出解析为与电源相关的事件的可视表示的工具。
比如：遇到功耗问题，我们可以执行以下命令抓取 `bugreport`，并通过`BatteryHistorian `分析唤醒源。
具体操作步骤如下：
```
adb shell dumpsys batterystats --reset
adb shell dumpsys batterystats --enable full-wake-history
adb bugreport bugreport.zip
```
**BatteryHistorian  详细使用方法**，请点击下方链接。
[BatteryHistorian使用详解 ](https://www.jianshu.com/p/3d385b12c544)

#四、最佳实践

通常，应用应避免部分唤醒锁定，因为这样很容易耗尽用户的电池。`Android`为需要部分唤醒锁的用例提供了替代`API`。部分唤醒锁的另一个用例是确保屏幕关闭时音乐应用程序继续播放。如果您使用唤醒锁来运行任务，请考虑在[background processing guide](https://developer.android.com/guide/background).
描述的替代方法。
如果必须使用部分唤醒锁，请遵循以下建议：
 *   确保您的应用程序的某些部分保留在前台。例如，如果您需要运行服务，请[start a foreground service](https://developer.android.com/guide/components/services.html#Foreground)。直观地向用户表明您的应用仍在运行。

 * 确保获取和释放唤醒锁的逻辑尽可能简单。当您的唤醒锁逻辑与混淆多种状态，超时，执行程序池和/或回调事件相关联时，该逻辑中的任何细微错误都可能导致唤醒锁的保存时间超出预期。这些错误很难诊断和调试。
至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！
 


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

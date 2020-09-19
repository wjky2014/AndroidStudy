#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的 **性能** 部分知识点，通过阅读本篇文章，您将收获以下内容:
>1. 过多的唤醒源wakeups
>2. 如何fix 过多唤醒源问题
>3. 最佳实践

# 1.过多的唤醒源wakeups
 **Wakeups** 是 [AlarmManager](https://developer.android.google.cn/reference/android/app/AlarmManager.html) API 中的一种机制 ，它可让开发人员在指定时间设置警报，进而达到唤醒设备的目的。您的应用通过使用  [RTC_WAKEUP](https://developer.android.google.cn/reference/android/app/AlarmManager.html#RTC_WAKEUP) 或[ELAPSED_REALTIME_WAKEUP](https://developer.android.google.cn/reference/android/app/AlarmManager.html#ELAPSED_REALTIME_WAKEUP)  标志调用[AlarmManager](https://developer.android.google.cn/reference/android/app/AlarmManager.html)中的种`set()`方法来设置唤醒警报。当触发唤醒警报后，设备将退出低功耗模式，并在执行警报 [onReceive()](https://developer.android.google.cn/reference/android/content/BroadcastReceiver.html#onReceive(android.content.Context,%20android.content.Intent))或 [onAlarm()](https://developer.android.google.cn/reference/android/app/AlarmManager.OnAlarmListener.html#onAlarm()) 方法的同时**holds**[partial wake lock ](https://developer.android.google.cn/topic/performance/vitals/wakelock.html)。如果唤醒警报触发过多，它们可能会耗尽设备的电池电量。 

为了帮助您提高应用程序质量，Android会自动监视应用程序是否存在过多的唤醒警报，并以Android vitals的形式显示信息。有关如何收集数据的信息，请参阅[Play控制台文档](https://support.google.com/googleplay/android-developer/answer/7385505)。

如果您的应用过度唤醒设备，则可以使用此页面中的指导来诊断和解决问题。

# 2.  如何fix 过多唤醒源问题

[AlarmManager](https://developer.android.google.cn/reference/android/app/AlarmManager.html) 是在Android平台的早期版本中推出的，但随着时间的推移，以前需要很多 [AlarmManager](https://developer.android.google.cn/reference/android/app/AlarmManager.html) 的用例现在更好新功能提供服务(比如： [ WorkManager](https://developer.android.google.cn/topic/libraries/architecture/workmanager))。本部分包含有关减少唤醒警报的提示，但从长远来看，请考虑迁移您的应用以遵循[第三节最佳实践](https://developer.android.google.cn/topic/performance/vitals/wakeup#best_practices)部分中的建议。

确定您在应用中安排唤醒警报的位置，并减少触发这些警报的频率。这里有一些提示：

*   查找对包含[RTC_WAKEUP](https://developer.android.google.cn/reference/android/app/AlarmManager.html#RTC_WAKEUP)   或  [ELAPSED_REALTIME_WAKEUP](https://developer.android.google.cn/reference/android/app/AlarmManager.html#ELAPSED_REALTIME_WAKEUP) 标志的各种  [AlarmManager](https://developer.android.google.cn/reference/android/app/AlarmManager.html) [set()](https://developer.android.google.cn/reference/android/app/AlarmManager.html#set(int,%20long,%20java.lang.String,%20android.app.AlarmManager.OnAlarmListener,%20android.os.Handler))  方法的调用 。

*   我们建议您将包，类或方法的名称包括在警报的标记名称中，以便您可以轻松地在源中识别设置警报的位置。以下是一些其他提示：

    *   忽略名称中的任何个人身份信息（PII），例如电子邮件地址。否则，设备将记录日志`_UNKNOWN`而不是警报名称。
    *   不要以编程方式获取类或方法的名称，例如通过调用  [getName()](https://developer.android.google.cn/reference/java/lang/Class.html#getName()) ，因为Proguard可能会混淆它们。而是使用硬编码的字符串。
    *   不要在警报标签中添加计数器或唯一标识符。系统将无法聚合以这种方式设置的警报，因为它们都具有唯一的标识符。

解决问题后，通过运行以下[ADB](https://developer.android.google.cn/studio/command-line/adb.html) 命令来验证唤醒警报是否按预期工作：

```
adb shell dumpsys alarm
``` 

该命令提供有关设备上警报系统服务状态的信息。有关更多信息，请参见 [dumpsys](https://source.android.google.cn/devices/tech/debug/dumpsys)。

# 3. 最佳实践

仅当您的应用需要执行面向用户的操作（例如发布通知或提醒用户）时，才使用唤醒警报。有关AlarmManager最佳做法的列表，请参阅[Scheduing Repeating Alarms](https://developer.android.google.cn/training/scheduling/alarms.html#tradeoffs)。

不要  [AlarmManager](https://developer.android.google.cn/reference/android/app/AlarmManager.html)
用于安排后台任务，尤其是重复的或网络后台任务。建议使用 [WorkManager](https://developer.android.google.cn/topic/libraries/architecture/workmanager) 执行后台任务，因为它具有以下优点：

*   批处理-合并作业，以减少电池消耗
*   持久性-如果重新启动设备，则在重新启动完成后运行计划的WorkManager作业
*   条件-作业可以根据条件运行，例如设备是否正在充电或WiFi是否可用

有关更多信息，请参阅[《后台处理指南》](https://developer.android.google.cn/guide/background)。

不要 [AlarmManager](https://developer.android.google.cn/reference/android/app/AlarmManager.html)  用于安排仅在应用程序运行时才有效的计时操作（换句话说，当用户退出应用程序时应取消计时操作）。在这种情况下，请使用  [Handler](https://developer.android.google.cn/reference/android/os/Handler.html) 该类，因为它更易于使用且效率更高。




**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

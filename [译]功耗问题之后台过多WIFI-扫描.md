#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
 >1.后台wifi 扫描过多
>2.发现问题
>3.研究 Wifi 扫描
>4.减少Wifi 扫描时间


# 1. 后台wifi 扫描过多

当应用在后台执行Wi-Fi扫描时，它会唤醒CPU，从而导致电池耗电。当扫描次数过多时，设备的电池寿命可能会明显缩短。如果应用处于`PROCESS_STATE_BACKGROUND`或 `PROCESS_STATE_CACHED`状态，则认为该应用在后台运行。

本文档说明了如何检测您的应用何时在后台执行过多的Wi-Fi扫描，并提供了有关诊断和解决问题的提示。

# 2.发现问题

您可能并不总是知道您的应用程序展示了异常数量的Wi-Fi扫描。如果您已经发布了应用程序，则Android vitals可以使您意识到该问题，从而可以对其进行修复。
## Android vitals

当您的应用在后台执行过多的Wi-Fi扫描时 ，Android生命体可以通过[Play控制台](https://play.google.com/apps/publish/signup/)提醒您，从而提高应用的性能 。当应用在后台运行时，每小时在0.10％的电池会话中执行超过4次扫描时，Android vitals认为Wi-Fi扫描过多。

一个 *电池会话* 指的是两个全电池充电之间的时间间隔。有关Google Play如何收集Android生命数据的信息，请参阅 [Play控制台](https://support.google.com/googleplay/android-developer/answer/7385505)文档。

#3.研究 Wifi 扫描

Battery Historian等工具可帮助您更深入地了解应用程序的扫描行为。Battery Historian提供了基于每个应用程序的Wi-Fi扫描行为的可视化信息，可以帮助您更清晰地了解应用程序的状况。有关Battery Historian的更多信息，请参阅 [使用Battery Historian分析电量使用](https://developer.android.google.cn/topic/performance/power/battery-historian.html#asd)。

有关使用Battery Historian的机制的信息，请参阅 [Batterystats 和Battery Historian演练](https://developer.android.google.cn/studio/profile/battery-historian.html)。

# 4.减少Wifi 扫描时间

如果可能，您的应用程序应在前台运行时应执行Wi-Fi扫描。前台服务会自动显示通知；因此，在前台执行Wi-Fi扫描会使用户知道在其设备上进行Wi-Fi扫描的原因和时间。

有关在前台时如何扫描的信息，请参阅该类的文档 。[ `WifiManager`](https://developer.android.google.cn/reference/android/net/wifi/WifiManager.html#startScan())

如果您的应用在后台运行时无法避免执行Wi-Fi扫描，则可以通过应用“ [Lazy First”](https://developer.android.google.cn/topic/performance/power/index.html#lazy)策略来受益。Lazy First包含可用于减少Wi-Fi扫描的三种技术： *减少*，*延迟*和*合并*。有关这些技术的信息，请参阅《 [优化电池寿命》](https://developer.android.google.cn/topic/performance/power/index.html)。




**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

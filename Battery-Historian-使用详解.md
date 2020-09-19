
 
##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中 **电量** 的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、安装Battery Historian
>二、收集Batterystats 数据
>三、使用Battery Historian分析数据
>四、Batterystats 额外数据
>五、使用Battery Historian 分析电量消耗
>六、Battery Historian能干什么




本文主要分析了Batterystats工具和Battery Historian脚本的基本用法和工作流程。

Batterystats是一个包含在Android框架中的工具，用于收集设备上的电池数据。 您可以使用adb将收集的电池数据转储到您的开发机器上，并创建一个报告，您可以使用Battery Historian进行分析。 Battery Historian将来自Batterystats的报告转换为可在浏览器中查看的HTML可视化文件。

它有什么好处：

向您显示过程在何处以及如何从电池中吸取电流。
识别您的应用程序中可延期或甚至删除的任务以延长电池寿命。
#一、安装Battery Historian

安装Battery Historian的最简单方法是使用Docker。 有关备用安装方法（包括从源代码构建）。 要使用Docker进行安装，请执行以下操作：

- 1.按照Docker网站上的说明安装[Docker Community Edition](https://www.docker.com/community-edition)。

- 2.要确认Docker已正确安装，请打开命令行并输入以下命令：
`docker run hello-world`

![Docker 成功安装信息](https://upload-images.jianshu.io/upload_images/5851256-242dc9b21c123ed6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 3.使用以下命令运行Battery Historian图像：

`docker --run -p port_number:9999 gcr.io/android-battery-historian:2.1 --port 9999`

- 4. 在浏览器中导航到Battery Historian以确认它正在运行。 地址因操作系统而异：
**Linux and Mac**
`http://localhost/port_number`
**For Windows**
`http://123.456.78.90:port_number`
您将看到Battery Historian起始页面，您可以在其中上载和查看电池统计信息。

![Battery Historian 首页](https://upload-images.jianshu.io/upload_images/5851256-f7e8df315e467e24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#二、收集Batterystats 数据

连接手机，打开开发者模式，连接adb。
- 1. 重置 batterystats 数据

`adb shell dumpsys batterystats --reset`

设备始终在后台收集batterystats和其他调试信息。 重置会清除旧电池采集数据.

- 2. 断开设备与电脑的连接，以便只从设备的电池中消耗电流。

- 3. 玩你的应用程序，并执行你想要的数据的行为; 例如，断开与WiFi的连接并将数据发送到云端。

- 4. 重新连接手机dump数据

`adb shell dumpsys batterystats > [path/]batterystats.txt`

- 5.  导出原始bugreport数据

**Android 7.0 以上**
`adb bugreport > [path/]bugreport.zip`

**Android 5.0/ 6.0**
`adb bugreport > [path/]bugreport.txt`

- 6.打开并提交分析数据
 [Battery Historian分析网站 需翻墙](http://192.168.11.30:9999/)

#三、使用Battery Historian分析数据

成功提交后，数据分析如下

![Battery Historian](https://upload-images.jianshu.io/upload_images/5851256-bf2f2d19dbfc0bcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 1. 从下拉列表中添加其他指标。
- 2. 将鼠标悬停在信息图标上可查看有关每个指标的更多信息，包括图表中使用的颜色的关键字。
- 3.将鼠标悬停在栏上，即可查看有关该指标的详细信息以及时间线上特定点的电池状态。

#四、Batterystats 额外数据

Battery Historian图表下面的stats部分可以查看batterystats.txt文件中的其他信息。

![Batterystats 额外数据](https://upload-images.jianshu.io/upload_images/5851256-64666189b7cd74eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. System Stats选项卡包含系统范围的统计信息,如单元信号级别和屏幕亮度。 这些信息提供了设备发生情况的整体情况。 这对确保没有外部事件影响您的测试特别有用。

2. App Stats选项卡包含有关特定应用程序的信息。 使用左侧的应用程序选择窗格中（图3）排序应用程序下拉列表对应用程序列表进行排序。 您可以选择一个特定的应用程序来查看使用下面（图4）个应用程序下拉列表的统计信息。
#五、使用 Battery Historian 分析电量消耗

Battery Historian工具提供了一段时间内设备电池消耗的深入分析。 在全系统级别，该工具以HTML表示形式从系统日志中查看与电源相关的事件。 在特定应用程序级别，该工具提供了各种数据，可帮助您确定电池耗尽应用程序的行为。

本文档介绍了使用Battery Historian了解电池消耗模式的一些方法。 该文件首先解释如何读取Battery Historian报告的系统范围数据。 然后，介绍如何使用Battery Historian诊断和解决与电池消耗有关的应用行为。 最后，它提供了有关Battery Historian可能特别有用的场景的几个提示。

Battery Historian工具提供了各种应用程序和系统行为的系统范围可视化，以及随着时间的推移与电池消耗的相关性

![Battery Historian 事例](https://upload-images.jianshu.io/upload_images/5851256-10384f5a77aa4ce7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此图特别感兴趣的是在y轴上测量的代表电池电量的黑色水平下降趋势线。 例如，在电池水平线的最开始时间点，大约早上6点50分，可视化显示电池电量水平相对急剧下降。
![CPU 唤醒锁等信息](https://upload-images.jianshu.io/upload_images/5851256-8eb7ce5d0260be72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在电池电量线开始时，随着电池电量急剧下降，显示屏显示出三件事：CPU正在运行，应用程序已获取唤醒锁，屏幕亮起。 通过这种方式，Battery Historian可帮助您了解电池消耗高时发生的事件。 然后，您可以将这些行为定位到您的应用中，并调查是否可以进行相关的优化。

除了系统范围内提供的宏观数据外，Battery Historian还提供特定于设备上运行的每个应用程序的表格和一些可视化数据。 表格数据包括：

- 该应用在设备上的预计用电量。
- 网络信息。
- Wakelocks。
- 服务。
- 进程信息。

这些表格提供了有关您的应用的两个维度数据。 首先，您可以查看您的应用的用电量排名与其他应用的相比。 为此，请单击表下的设备功率估算表。 这个例子考察了一个名为Pug Power的虚构应用程序。

![调查哪些应用程序消耗的功率最大 ](https://upload-images.jianshu.io/upload_images/5851256-018fe3e447860800.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


上图的表格显示，Pug Power是该设备上第九大电池供电用户，也是不属于操作系统的第三大应用。 这个数据表明，这个应用程序承担更深入的调查。

要查找特定应用程序的数据，请将其包名称输入位于可视化左侧下方应用程序选择下的两个下拉菜单中的较低位置

![](https://upload-images.jianshu.io/upload_images/5851256-ec55f2f6600ccca0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当您选择特定的应用程序时，以下数据可视化类别将更改为显示应用程序特定的数据而不是系统范围的数据：
- SyncManager.
- Foreground process.
- Userspace Wakelock.
- Top app.
- JobScheduler.
- Activity Manager Proc.

如果您的应用程序执行同步并执行作业的次数超过必要，SyncManager和JobScheduler可视化立即显而易见。 在此过程中，他们可以快速发现优化应用行为以改善电池性能的机会。

您还可以获得更多的特定于应用程序的可视化数据，用户空间Wakelock。 要将此信息包含在错误报告中，请在终端窗口中输入以下命令：
`adb shell dumpsys batterystats --enable full-wake-history`

ps:从Android 6.0（API级别23）开始，该平台包含Doze功能，这对应用程序实施了某些优化。 例如，无论JobScheduler如何安排他们，Doze都会在短期维护时段内批处理作业。

![显示特定应用的可视化数据](https://upload-images.jianshu.io/upload_images/5851256-3b0d014ee12e6dd9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![应用数据表格](https://upload-images.jianshu.io/upload_images/5851256-a5f20a276ee91eae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看看可视化并不立即显示任何东西。 JobScheduler行显示该应用程序没有计划任务。 SyncManager行显示应用程序未执行任何同步。

然而，对表格数据的Wakelocks部分的检查表明，帕格电力总共花费了一个多小时的时间。 这种不寻常的和昂贵的行为可以解释应用程序的高功耗水平。 这一信息有助于开发人员瞄准优化可能极大帮助的领域。 在这种情况下，为什么应用程序会获得如此多的唤醒时间，开发人员如何改善这种行为？

#六、Battery Historian能干什么

Battery Historian还可以帮助您诊断改善电池行为的机会。 例如，Battery Historian可以告诉你你的应用程序是否：

- 过度频繁地发出唤醒警报（每10秒钟或更少）。
- 持续持有GPS锁。
- 每30秒或更短时间安排一次作业。
- 计划每30秒或更短时间同步一次。
- 更频繁地使用蜂窝无线电比您期望的更频繁。

 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

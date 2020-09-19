 
##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、飞行模式底电流问题
>二、正常待机功耗简介
>三、最干净的待机电流波形
>四、通过唤醒源理清正常待机问题
>五、Audio Playback 功耗问题
>六、Display 及多媒体功耗问题
>七、通话功耗问题



#一、飞行模式底电流问题

飞行模式底电流正常是所有功耗问题的前置条件，此时`wifi 、Bluetooth、Location、Radio` 都处于关闭状态。

## 1.系统睡眠的条件 

查看`CPU`是否进入`suspend `状态， `suspend `确切的说是 `MCU （ARM）`的`suspend `, 也是`CPU` 进入WFI（Wait  For Interrupt）状态，`CPU` 进入`WFI`后，整个系统就依靠一颗  **SCP：SPM（System Power Manager）** 来控制 **睡眠/唤醒**  的流程

## 2.灭屏到CPU 进入suspend的流程
 
![灭屏到CPU 进入suspend的流程](https://upload-images.jianshu.io/upload_images/5851256-df75bdf0088b0956.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 3.判断系统是否进入suspend的方法

在kernel log中搜索关键字 **Chip_pm_begin**  或者 **suspend entry**

![查看suspend状态](https://upload-images.jianshu.io/upload_images/5851256-6ef18cd2c9ebb6a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 4.查看SPM（System Power Manager）状态 

1.在 `kernel log` 中搜索关键字 **wake up by**, 可以查看唤醒源的情况，
2.在` kernel log`中查看`R13`寄存器跟**debug_flag**的值，如果后面值不是**ff**结尾，此时系统功耗可能会存在异常，需要查看分析。

![查看SPM（System Power Manager）状态](https://upload-images.jianshu.io/upload_images/5851256-2a0d81c72628ab6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 二、 正常待机功耗简介

待机功耗很容易出现问题，并且很难理清，因为其涉及到`APK 、Modem、Wifi、Other `这些不确定因素。

**功耗问题处理原则：**

1.先花时间把现象理清，到底在什么样的环境下复现。
2.多做几个实验，给出清晰的问题描述、问题复现条件、电流波形图。
3.提供`关闭 modem 的log`。

# 3. 最干净的待机电流波形

![最干净的待机电流波形](https://upload-images.jianshu.io/upload_images/5851256-b9a4c5d9bc271a74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 4. 通过唤醒源理清正常待机问题

## 1. 其他唤醒源分析

 kernel Log收缩关键字 **wakeup by**, **wakeup by xxxx** ,其中 **xxxx** 就是唤醒源。

![image.png](https://upload-images.jianshu.io/upload_images/5851256-ef44a8cef2dc6258.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 2. APK 唤醒源分析

`APK` 唤醒系统是通过设置 `type 0` 和`type 2`的alarm 来唤醒系统，这两种`alarm` 会设置到`RTC`寄存器中，而`RTC Module` 其实是在`PMIC` 里面，因此`APK `唤醒实际上是`PMIC`的`EINT `唤醒。

`RTC` 唤醒`sys_log`中搜索关键字 **AlarmManager: sending alarm Alarm**，查看  `type 0`  和 `type 2` 的应用有哪些。

![gms包APK经常唤醒系统](https://upload-images.jianshu.io/upload_images/5851256-f562d5fdf5dd1e48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果`log`没有开启，请使用`adb shell dumpsys alarm log on`


# 5. Audio Playback 功耗问题

`Audio playback` 时候`MTK `低端平台没有专门的`audio DSP（Heilo X20除外）`，故无法在`suspend `状态下完成`audio playback`，故需要`CPU` 做这件事情。
 
通话的时候之所以可以睡眠，是疑问`modem` 充当了`dsp`的角色。

**deep idle 状态**

**Deep idle** 实际上系统还是`Active `状态，因此`CPU `需要快速响应系统请求调度，因此 **GPT唤醒源** 是`Deep idle `的主要唤醒源。

在 `Kernel Log`中搜索关键字 **wake up by** , 这个log是在 **swapper进程** 中打印出来的（代表当前`CPU`在运行`idle task`） ，并且后面可以看到 **DP：**的字样。
 
![播放MPS GPT 唤醒源 log](https://upload-images.jianshu.io/upload_images/5851256-b999d93dd4974f90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


`MP3 `播放时进入`deep idle `状态（`20mA`）举例

![MP3 播放时进入deep idle 状态（20mA）举例](https://upload-images.jianshu.io/upload_images/5851256-67c3c95f8c431a1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**区分suspend 与deep Idle**

1. **suspend** 是跑在 `suspend workqueue` 中，因此log的进程主体是` kwork`
2. **deep idle** 是跑在`idle task` 中，因此log的进程主体是`swapper`
3.  **suspend** 默认不会被 `GPT` 唤醒。


# 6.Display 及多媒体功耗问题

手机所有亮屏的场景都是模块自身的耗电跟`Display` 部分耗电的叠加，所以`Display` 的功耗在整个系统中占比非常高。
`Display` 功耗 = 硬件+平台+内容 

在 `Kernel Log`中搜索关键字 **wake up by** , 这个log是在 **swapper进程** 中打印出来的（代表当前`CPU`在运行`idle task`） ，并且后面可以看到 **SO：**的字样（通）

![Display 及多媒体功耗问题 ](https://upload-images.jianshu.io/upload_images/5851256-f20127b782692853.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 7. 通话电流功耗问题


**通话模式的功耗跟正常模式的功耗区别**

![通话模式的功耗跟正常模式的功耗区别](https://upload-images.jianshu.io/upload_images/5851256-50970b82e15f037d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一般情况下 
GSM 功耗< 3G-TD < 3G-W 功耗

![GSM 3G-TD 功耗图](https://upload-images.jianshu.io/upload_images/5851256-e964a2056d3fd375.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![3G-W功耗图](https://upload-images.jianshu.io/upload_images/5851256-f1c3e7170e2c4b06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


[飞行模式底电流 参考文档](https://pan.baidu.com/s/1rgBAfeimlOKm-DckvD9UMw)
[标准模式功耗 参考文档](https://pan.baidu.com/s/1bLjIBNoIBm5Nd4HOF1uCrw)
[Audio PlayBack功耗 参考文档](https://pan.baidu.com/s/1sSZAthqKGJJYImVd74eMMA)
[通话底功耗 参考文档](https://pan.baidu.com/s/18Ran8YUAxqQssfV-n_fvGw)
[Display 及多媒体功耗 参考文档](https://pan.baidu.com/s/15-a6ZaHoM7iXBTp1av2hLg)
![](https://upload-images.jianshu.io/upload_images/5851256-9942702b8d1a73b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

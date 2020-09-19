#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、Camera 元数据支持
>二、 Camera 原始传感器数据支持
>三、Camera 3A 状态与模式


# 一、Camera 元数据支持
为了支持Android框架保存原始图像文件，需要有关传感器特性的大量元数据。这包括诸如色彩空间和镜头阴影功能之类的信息。

这些信息大部分是相机子系统的静态属性，因此可以在配置任何输出管道或提交任何请求之前进行查询。新的相机API大大扩展了该 getCameraInfo()方法提供的信息，以将该信息提供给应用程序。

另外，对照相机子系统的手动控制需要来自各种设备的有关其当前状态以及用于捕获给定帧的实际参数的反馈。硬件实际使用的控件的实际值（曝光时间，帧持续时间和灵敏度）必须包含在输出元数据中。这是必不可少的，以便应用程序知道何时进行夹紧或舍入，并且应用程序可以补偿用于图像捕获的实际设置。

例如，如果应用程序在请求中将帧持续时间设置为0，则HAL必须将帧持续时间限制为该请求的实际最小帧持续时间，并在输出结果元数据中报告该限制的最小持续时间。

因此，如果应用程序需要实现自定义3A例程（例如，正确计量HDR突发信号），则它需要知道用于捕获已收到的最新结果集的设置，以更新下一个请求的设置。因此，新的相机API向每个捕获的帧添加了大量的动态元数据。这包括用于捕获的请求参数和实际参数，以及其他每帧元数据，例如时间戳和统计信息生成器输出。

# 二、 Camera 原始传感器数据支持
对于大多数设置，期望它们可以在每帧更改，而不会对输出帧流造成明显的卡顿或延迟。理想情况下，输出帧速率应仅由捕获请求的帧持续时间字段控制，并且与处理块配置的任何更改无关。实际上，已知某些特定的控件更改缓慢。其中包括相机管线的输出分辨率和输出格式，以及影响物理设备的控件，例如镜头焦距。每个控件集的确切要求将在后面详细介绍。

除了旧API支持的像素格式外，新API还增加了对原始传感器数据（Bayer RAW）的支持要求，这既适用于高级相机应用程序，又支持原始图像文件。

# 三、Camera 3A 状态与模式

虽然实际的3A算法取决于HAL的实现，但是HAL接口定义了高级状态机描述，以允许HAL设备和框架就3A的当前状态进行通信并触发3A事件。

打开设备时，所有3A状态都必须为STATE_INACTIVE。流配置不会重置3A。例如，必须在整个configure()呼叫过程中保持锁定的焦点。

触发3A动作仅涉及在下一个请求的设置中设置相关的触发条目，以指示触发开始。例如，启动自动聚焦扫描的触发器是针对一个请求将条目ANDROID_CONTROL_AF_TRIGGER设置为ANDROID_CONTROL_AF_TRIGGER_START。并通过将ANDROID_CONTROL_AF_TRIGGER设置为ANDROID_CONTRL_AF_TRIGGER_CANCEL来触发取消自动对焦扫描。否则，该条目将不存在或被设置为ANDROID_CONTROL_AF_TRIGGER_IDLE。将触发条目设置为非空闲值的每个请求将被视为独立的触发事件。

在顶层，3A由ANDROID_CONTROL_MODE设置控制。它在3A（ANDROID_CONTROL_MODE_OFF），正常AUTO模式（ANDROID_CONTROL_MODE_AUTO）和场景模式设置（ANDROID_CONTROL_USE_SCENE_MODE）之间进行选择：

- 在关闭模式下，各个自动对焦（AF），自动曝光（AE）和自动白平衡（AWB）模式均有效关闭，并且3A例程均不能覆盖任何捕获控件。

- 在自动模式下，AF，AE和AWB模式均运行各自独立的算法，并具有各自的模式，状态和触发元数据条目，如下一节中所列。

- 在USE_SCENE_MODE中，必须使用ANDROID_CONTROL_SCENE_MODE条目的值来确定3A例程的行为。在除FACE_PRIORITY以外的SCENE_MODEs中，HAL必须覆盖ANDROID_CONTROL_AE / AWB / AF_MODE的值，以使其成为所选SCENE_MODE的首选模式。例如，HAL可能更喜欢SCENE_MODE_NIGHT使用CONTINUOUS_FOCUS AF模式。对于这些场景模式，必须忽略场景时用户对AE / AWB / AF_MODE的任何选择。

- 对于SCENE_MODE_FACE_PRIORITY，AE / AWB / AFMODE控件的工作方式与ANDROID_CONTROL_MODE_AUTO相同，但是3A例程必须偏重于测光并聚焦于场景中检测到的任何面部。

## 自动对焦设置和结果输入

![主要元数据列表](https://upload-images.jianshu.io/upload_images/5851256-212499be4da71e80.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![其他元数据列表](https://upload-images.jianshu.io/upload_images/5851256-eedba37e9ab4a2c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 自动曝光设置和结果输入
![主要元素据列表](https://upload-images.jianshu.io/upload_images/5851256-46ef5e482cba13c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![其他元数据列表](https://upload-images.jianshu.io/upload_images/5851256-5ae238336dd48595.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 自动白平衡设置和结果输入
![白平衡列表](https://upload-images.jianshu.io/upload_images/5851256-1a2792f945bf25d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 一般状态机转换说明 
在AF，AE或AWB模式之间切换总是将算法的状态重置为INACTIVE。同样，如果CONTROL_MODE == USE_SCENE_MODE，则在CONTROL_MODE或CONTROL_SCENE_MODE之间进行切换会将所有算法状态重置为INACTIVE。

下表是每种模式。
![](https://upload-images.jianshu.io/upload_images/5851256-54f61a46dcd15627.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5851256-faad793bb45108b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![AF_MODE_CONTINOUS_PICTURE](https://upload-images.jianshu.io/upload_images/5851256-496205dba2a46a8a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## AE和AWB状态机

AE和AWB状态机基本相同。AE具有其他FLASH_REQUIRED和PRECAPTURE状态。因此，对于AWB状态机，下面的行引用了这两种状态。
![AE /AWB mode](https://upload-images.jianshu.io/upload_images/5851256-b991803b9f4c330b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##启用手动控制
在配置设备3A块以允许直接应用程序控制时，还涉及一些控件。

用于3A控制的HAL模型是针对每个请求，HAL检查3A控制字段的状态。如果启用了任何3A例程，则该例程将覆盖与该例程相关的控制变量，然后这些覆盖值可在该捕获的结果元数据中使用。因此，例如，如果在请求中启用了自动曝光，则HAL应该覆盖请求的曝光，增益和帧持续时间字段（可能还有闪光字段，具体取决于AE模式）。相关控件列表为：
![Manual Control 模式](https://upload-images.jianshu.io/upload_images/5851256-63435f7e822e364b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

 至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

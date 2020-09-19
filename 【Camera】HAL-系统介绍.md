#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、Camera 请求
>二、HAL & Camera 子系统
>三、Camera启动和预期的操作顺序
>四、Camera 硬件Level
 

# 一、Camera 请求
应用程序框架向摄像头子系统发出对捕获结果的请求。一个请求对应一组结果。请求封装了有关这些结果的捕获和处理的所有配置信息。其中包括分辨率和像素格式等内容；手动传感器，镜头和闪光灯控制；3A工作模式；RAW到YUV处理控制；和统计信息生成。这样可以更好地控制结果的输出和处理。可以一次执行多个请求，并且提交请求是非阻塞的。并且始终按照接收到的顺序处理请求。
![图1.相机模型](https://upload-images.jianshu.io/upload_images/5851256-e0991fb9802ef7ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 二、HAL & Camera 子系统
相机子系统包括相机管道中组件的实现，例如3A算法和处理控件。相机HAL为您提供了实现这些组件版本的接口。为了保持多个设备制造商与图像信号处理器（ISP或相机传感器）供应商之间的跨平台兼容性，相机管道模型是虚拟的，并不直接对应于任何实际的ISP。但是，它与真实的处理管道足够相似，因此您可以将其有效地映射到硬件。此外，它足够抽象，可以支持多种不同的算法和操作顺序，而不会影响质量，效率或跨设备兼容性。

摄像头管道还支持应用程序框架可以启动以打开诸如自动对焦之类的触发器。它还会将通知发送回应用程序框架，通知应用程序诸如自动对焦锁定或错误之类的事件。
![图2.摄像头管道](https://upload-images.jianshu.io/upload_images/5851256-606220cf5cbc94d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
请注意，上图中显示的某些图像处理模块在初始版本中定义不明确。相机管道进行以下假设：

- RAW拜耳输出不在ISP内部进行任何处理。
- 根据原始传感器数据生成统计信息。
- 将原始传感器数据转换为YUV的各种处理块按任意顺序排列。
- 虽然显示了多个缩放和修剪单位，但所有缩放器单位共享输出区域控件（数字缩放）。但是，每个单元可能具有不同的输出分辨率和像素格式。

## API使用摘要
这是使用Android camera API的步骤的简要摘要。有关这些步骤（包括API调用）的详细分解，请参见“启动和预期操作顺序”部分。

- 1.侦听并枚举相机设备。
- 2.打开设备并连接侦听器。
- 3.配置目标用例的输出（例如，静态捕获，记录等）。
- 4.为目标用例创建请求。
- 5.捕获/重复请求和突发。
- 6.接收结果元数据和图像数据。
- 7.切换用例时，请返回步骤3。

## HAL层操作摘要

- 捕获的异步请求来自框架。
- HAL设备必须按顺序处理请求。并为每个请求生成输出结果元数据，以及一个或多个输出图像缓冲区。
- 请求和结果以及后续请求引用的流的先进先出。
- 给定请求的所有输出的时间戳都必须相同，以便框架可以根据需要将它们匹配在一起。
- 所有捕获配置和状态（3A例程除外）都封装在请求和结果中。

![图3.摄像机HAL概述](https://upload-images.jianshu.io/upload_images/5851256-b324053112662b1b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 三、启动和预期的操作顺序

本节包含使用相机API时预期步骤的详细说明。请参阅 [platform / hardware / interfaces / camera /](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/)了解HIDL接口定义。

## 枚举，打开摄像头设备并创建活动会话 

1.  初始化后，框架开始监听实现该 [ ICameraProvider](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/provider/2.4/ICameraProvider.hal) 接口的所有现有摄像机提供程序 。如果存在此类提供者，则框架将尝试建立连接。
2.  该框架通过枚举摄像头设备 `ICameraProvider::getCameraIdList()`。
3.  该框架`ICameraDevice`通过调用各自的实例化一个新的`ICameraProvider::getCameraDeviceInterface_VX_X()`。
4.  框架调用`ICameraDevice::open()`以创建一个新的活动捕获会话ICameraDeviceSession。

## 使用Active 的Camera
1. 该框架ICameraDeviceSession::configureStreams() 使用到HAL设备的输入/输出流列表进行调用。
2. 框架要求通过调用某些用例的默认设置ICameraDeviceSession::constructDefaultRequestSettings()。ICameraDeviceSession由创建后，随时可能会发生这种情况ICameraDevice::open。
3. 框架使用一组默认设置中的一组设置以及至少一个早先由框架注册的输出流，来构造第一个捕获请求并将其发送到HAL。这是通过发送到HAL的ICameraDeviceSession::processCaptureRequest()。HAL必须阻止此调用的返回，直到准备好发送下一个请求为止。
4. 框架会继续提交请求，并根据需要调用 ICameraDeviceSession::constructDefaultRequestSettings()以获取其他用例的默认设置缓冲区。
5. 当开始捕获请求时（传感器开始进行捕获以进行捕获），HAL调用ICameraDeviceCallback::notify()SHUTTER消息，包括帧号和开始曝光的时间戳。此通知回调不必在第一次processCaptureResult()调用请求之前发生，但是在 调用该捕获之前，不会将任何结果传递给应用程序进行捕获 notify()。
6. 经过一段流水线延迟后，HAL开始使用将返回的完整捕获返回给框架ICameraDeviceCallback::processCaptureResult()。它们以与提交请求相同的顺序返回。根据摄像机HAL设备的管线深度，可以一次执行多个请求

一段时间后，将发生以下情况之一：

- 框架可能会停止提交新请求，等待现有捕获完成（所有缓冲区已满，返回所有结果），然后ICameraDeviceSession::configureStreams() 再次调用。这将为新的一组输入/输出流重置摄像机硬件和管线。某些流可以从以前的配置中重用。如果剩余至少一个注册的输出流，则框架将从第一个捕获请求继续到HAL。（否则， ICameraDeviceSession::configureStreams()首先需要。）

 - 框架可以调用ICameraDeviceSession::close() 以结束相机会话。当没有其他来自框架的调用处于活动状态时，可以随时调用此方法，尽管该调用可能会阻塞，直到完成所有正在进行的捕获（返回所有结果，填充所有缓冲区）为止。在之后close()调用返回，没有更多的调用 ICameraDeviceCallback从HAL允许的。一旦 close()调用正在进行，框架可能不会调用任何其他HAL设备功能。

- 如果发生错误或其他异步事件，则HAL必须ICameraDeviceCallback::notify()使用适当的错误/事件消息进行调用 。从致命的设备范围错误通知中返回后，HAL应该像close()在其上被调用一样起作用。但是，HAL必须在调用之前取消或完成所有未完成的捕获notify()，以使一旦 notify()被致命错误调用，框架将不再从设备接收进一步的回调。方法从致命错误消息返回close()后，除方法外 还应返回-ENODEV或NULL notify()。

![图4.相机操作流程](https://upload-images.jianshu.io/upload_images/5851256-1d8aa4e892332710.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 四、硬件Level 
相机设备可以根据其功能实现几种硬件级别。有关更多信息，请参阅 [支持的硬件级别](https://developer.android.google.cn/reference/android/hardware/camera2/CameraCharacteristics#INFO_SUPPORTED_HARDWARE_LEVEL)。

## 应用程序捕获请求，3A控制和处理管道之间的交互
根据3A控制块中的设置，摄像头管道会忽略应用程序捕获请求中的某些参数，而是使用3A控制例程提供的值。例如，当自动曝光处于活动状态时，传感器的曝光时间，帧持续时间和灵敏度参数由平台3A算法控制，而任何应用程序指定的值都将被忽略。3A例程为帧选择的值必须在输出元数据中报告。下表描述了3A控制块的不同模式以及由这些模式控制的属性。有关这些属性的定义，请参见[ platform / system / media / camera / docs / docs.html](https://android.googlesource.com/platform/system/media/+/master/camera/docs/docs.html)文件。

![](https://upload-images.jianshu.io/upload_images/5851256-1b8b650580d37196.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
图2中“图像处理”块中的控件均以相似的原理操作，通常每个块具有三种模式：

- OFF：禁用该处理块。无法禁用去马赛克，色彩校正和色调曲线调整块。

- 快速：在此模式下，与“关闭”模式相比，处理块可能不会减慢输出帧速率，但在有此限制的情况下，应产生最佳质量的输出。通常，它将用于预览或视频记录模式，或用于静态图像的连拍。在某些设备上，这可能等效于OFF模式（在不降低帧速率的情况下无法进行处理），而在某些设备上，这可能等效于HIGH_QUALITY模式（最佳质量仍不会降低帧速率）。

- HIGH_QUALITY：在此模式下，处理模块应产生可能的最佳质量结果，并根据需要降低输出帧速率。通常，这将用于高质量静止图像捕获。某些块包含一个手动控件，可以选择它来代替FAST或HIGH_QUALITY。例如，颜色校正块支持颜色变换矩阵，而色调曲线调整则支持任意全局色调映射曲线。

摄像机子系统可以支持的最大帧速率取决于许多因素：

- 请求的输出图像流分辨率
- 成像仪上合并/跳过模式的可用性
- 成像器接口的带宽
- 各种ISP处理模块的带宽

由于这些因素在不同的ISP和传感器之间可能会有很大差异，因此相机HAL接口试图将带宽限制抽象为尽可能简单的模型。呈现的模型具有以下特征：

- 在给定应用程序要求的输出流大小的情况下，图像传感器始终配置为输出可能的最小分辨率。最小分辨率定义为至少与最大请求输出流大小一样大。

- 由于任何请求都可以使用任何或所有当前配置的输出流，因此必须将传感器和ISP配置为支持将单个捕获同时缩放到所有流。

- JPEG流就像未处理请求的已处理YUV流一样；在直接引用它们的请求中，它们充当JPEG流。

- JPEG处理器可以与相机管道的其余部分同时运行，但一次不能处理多个捕获。

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

 至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

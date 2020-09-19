#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、HAL 3 简介
>二、HAL 3 功能
>三、HAL 1 概述


#一、HAL 3 简介

Android 的相机硬件抽象层 (HAL) 可将 [android.hardware.camera2](https://developer.android.google.cn/reference/android/hardware/camera2/package-summary) 中较高级别的相机框架 API 连接到底层的相机驱动程序和硬件。Android 8.0 引入了 [Treble](https://source.android.google.cn/devices/architecture/treble)，用于将 CameraHal API 切换到由 HAL 接口描述语言 (HIDL) 定义的稳定接口。如果您之前为 Android 7.0 及更低版本开发过相机 HAL 模块和驱动程序，请注意相机管道中发生的重大变化。

# 二、HAL 3 功能

重新设计 Android Camera API 的目的在于大幅提高应用对于 Android 设备上的相机子系统的控制能力，同时重新组织 API，提高其效率和可维护性。借助额外的控制能力，您可以更轻松地在 Android 设备上构建高品质的相机应用，这些应用可在多种产品上稳定运行，同时仍会尽可能使用设备专用算法来最大限度地提升质量和性能。

版本 3 相机子系统将多个运行模式整合为一个统一的视图，您可以使用这种视图实现之前的任何模式以及一些其他模式，例如连拍模式。这样一来，便可以提高用户对聚焦、曝光以及更多后期处理（例如降噪、对比度和锐化）效果的控制能力。此外，这种简化的视图还能够使应用开发者更轻松地使用相机的各种功能。

API 将相机子系统塑造为一个管道，该管道可按照 1:1 的基准将传入的帧捕获请求转化为帧。这些请求会封装有关帧的捕获和处理的所有配置信息，其中包括分辨率和像素格式；手动传感器、镜头和闪光灯控件；3A 运行模式；RAW->YUV 处理控件；统计信息生成等等。

简单来说，应用框架从相机子系统请求帧，然后相机子系统将结果返回到输出流。此外，系统还会针对每组结果生成包含色彩空间和镜头阴影等信息的元数据。您可以将相机版本 3 看作相机版本 1 的单向流管道。它会将每个捕获请求转化为传感器捕获的一张图像，这张图像将被处理成：

- 包含有关捕获的元数据的结果对象。
- 图像数据的 1 到 N 个缓冲区，每个缓冲区会进入自己的目的地 Surface。

可能的输出 Surface 组经过预配置：

- 每个 Surface 都是一个固定分辨率的图像缓冲区流的目标位置。
- 一次只能将少量 Surface 配置为输出（约 3 个）。

 一个请求中包含所需的全部捕获设置，以及要针对该请求将图像缓冲区（从总配置组）推送到其中的输出 Surface 的列表。请求可以只发生一次（使用 capture()），也可以无限重复（使用 setRepeatingRequest()）。捕获的优先级高于重复请求的优先级

![图 1. 相机核心操作模型](https://upload-images.jianshu.io/upload_images/5851256-1fe2c767d8840c16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 3. HAL 1 概述

**由于相机 HAL1 已弃用，建议在搭载 Android 9 或更高版本的设备上使用相机 HAL3.**

相机子系统的第 1 个版本被设计为具有高级控件和以下三种运行模式的黑盒子：

- 预览
 - 视频录制
- 静态拍摄

三种模式具有略有不同又相互重叠的功能。这样就难以实现介于其中两种运行模式之间的新功能，例如连拍模式。

![图 2. 相机组件
](https://upload-images.jianshu.io/upload_images/5851256-f50c6300e3d5c27e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于很多设备仍然依赖相机 HAL1，因此 Android 7.0 继续支持该模块。此外，Android 相机服务还支持同时实现两种 HAL（1 和 3），如果您希望通过相机 HAL1 支持性能略低的前置摄像头，并通过相机 HAL3 支持更为高级的后置摄像头，那么这项支持将非常有用。

有一种单独的相机 HAL 模块（拥有自己的[版本号](https://source.android.google.cn/devices/camera/versioning.html#module_version)），其中列出了多种独立的相机设备，每种都有自己的版本号。要支持设备 2 或更新版本，必须使用相机模块 2 或更新版本，而且此类相机模块可以具有混合的相机设备版本（我们在上文中提到 Android 支持同时实现两种 HAL，就是这个含义）。


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

 至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、Android 架构概览
>二、应用框架
>三、Binder IPC
>四、System Service
>五、HAL 硬件抽象层
>六、Linux Kernel
>七、HAL层接口定义语言
>八、架构资源

# 一、Android 架构概览
Android系统架构包含以下组件：
![图1. Android系统架构](https://upload-images.jianshu.io/upload_images/5851256-c26ec491f42f5db0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#二、应用框架

应用程序框架是应用程序开发人员最常使用的。作为硬件开发人员，您应该了解开发人员API，因为许多API直接映射到基础HAL接口，并且可以提供有关实现驱动程序的有用信息。

#三、Binder IPC
Binder  进程间通信（IPC）机制允许应用程序框架跨进程调用Android系统服务代码。并且高版本的 Framework API可以与Android系统服务进行交互。

#四、 System Service
系统服务是集中的模块化组件，例如窗口管理器，搜索服务或通知管理器。应用程序框架API公开的功能与系统服务进行通信以访问底层硬件。Android包括两组服务：系统（例如Window Manager和Notification Manager）和媒体（涉及播放和录制媒体的服务）。

#五、HAL 硬件抽象层
HAL定义了供硬件供应商实施的标准接口，该接口使Android无需考虑底层驱动程序的实现。使用HAL可使您实现功能而不会影响或修改更高级别的系统。HAL实现被打包到模块中，并由Android系统在适当的时间加载。有关详细信息，请参阅 [硬件抽象层（HAL）](https://source.android.google.cn/devices/architecture/hal.html)。
#六、 Linux Kernel
开发设备驱动程序类似于开发典型的Linux设备驱动程序。Android使用Linux内核的版本，并添加了一些特殊的附加功能，例如Low Memory Killer（内存管理系统，主要优化低内存下的性能），唤醒锁（ [`PowerManager`](https://developer.android.google.cn/reference/android/os/PowerManager.html) 系统服务），Binder IPC驱动程序以及其他重要功能。移动嵌入式平台。这些添加主要用于系统功能，并且不影响驱动程序开发。您可以使用任何版本的内核，只要它支持必需的功能（例如活页夹驱动程序）即可。但是，我们建议使用最新版本的Android内核。有关详细信息，请参见 [构建内核](https://source.android.google.cn/setup/building-kernels.html)

# 七、HIDL  HAL层接口定义语言
Android 8.0重新设计了Android OS框架（在一个名为Treble的项目中 ），以使制造商将设备更新到新版本的Android变得更加轻松，快捷且成本更低。在这种新架构中，HAL接口定义语言（HIDL）指定了HAL及其用户之间的接口，从而可以在不重建HAL的情况下替换Android框架。

**注意：**有关Project Treble的更多详细信息，请参阅开发人员博客文章 [Treble：Android的模块化基础](https://android-developers.googleblog.com/2017/05/here-comes-treble-modular-base-for.html)和 [Project Treble的更快采用](https://android-developers.googleblog.com/2018/05/faster-adoption-with-project-treble.html)。


HIDL通过新的供应商界面将供应商实施（由硅制造商编写的特定于设备的低级软件）与Android OS框架分开。供应商或SOC制造商只构建一次HAL，并将其放置在`/vendor`设备的分区中；然后，可以在不需重新编译HAL的情况下，通过[OTA更新](https://source.android.google.cn/devices/tech/ota/)来替换其自己分区中的框架。
传统Android架构与当前基于HIDL的架构之间的区别在于供应商界面的使用：

1.在Android 7.x和更早版本中，不存在正式的供应商界面，因此设备制造商必须更新大部分Android代码才能将设备移至更新版本的Android：
![图2.旧版Android更新环境](https://upload-images.jianshu.io/upload_images/5851256-367e346215287e7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.在Android 8.0及更高版本中，新的稳定的供应商界面提供对Android硬件特定部分的访问，因此设备制造商只需更新Android OS框架即可交付新的Android版本，而无需硅制造商的额外工作：
![图3.当前的Android更新环境](https://upload-images.jianshu.io/upload_images/5851256-6653318c71fb887f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从Android 8.0及更高版本启动的所有新设备都可以利用新架构。为了确保供应商实现的前向兼容性，供应商接口由[供应商测试套件（VTS）](https://source.android.google.cn/devices/tech/vts/index.html)进行验证 ，这类似于[兼容性测试套件（CTS）](https://source.android.google.cn/compatibility/cts/)。您可以使用VTS在旧版和当前Android架构中自动执行HAL和OS内核测试。

# 八、架构资源

有关Android体系结构的详细信息，请参见以下部分：
*   [HAL类型](https://source.android.google.cn/devices/architecture/hal-types.html)。描述绑定的，直通的，Same-Process（SP）和旧的HAL。
*   [HIDL（常规）](https://source.android.google.cn/devices/architecture/hidl/index.html)。包含有关HAL及其用户之间接口的常规信息。
*   [HIDL（C ++）](https://source.android.google.cn/devices/architecture/hidl-cpp/index.html)。包含有关创建HIDL接口的C ++实现的详细信息。
*   [HIDL（Java）](https://source.android.google.cn/devices/architecture/hidl-java/index.html)。包含有关HIDL接口的Java前端的详细信息。
*   [ConfigStore HAL](https://source.android.google.cn/devices/architecture/configstore/index.html)。描述用于访问用于配置Android框架的只读配置项的API。
*   [设备树覆盖](https://source.android.google.cn/devices/architecture/dto/index.html)。提供有关在Android中使用设备树覆盖（DTO）的详细信息。
*   [供应商本地开发套件（VNDK）](https://source.android.google.cn/devices/architecture/vndk/index.html)。描述用于实施供应商HAL的供应商专有库的集合。
*   [供应商接口对象（VINTF）](https://source.android.google.cn/devices/architecture/vintf/index.html)。描述汇总有关设备的相关信息并使这些信息通过可查询的API可用的对象。
*   [适用于Android 8.0的SELinux](https://source.android.google.cn/security/selinux/images/SELinux_Treble.pdf)。详细介绍SELinux的更改和自定义。


 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

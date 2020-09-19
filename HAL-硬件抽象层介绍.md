#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
 >一、HAL 概述
>二、HAL 模块介绍
>三、HAL 设备介绍
>四、构建HAL模块




# 一、HAL 概述 
HAL定义了供硬件供应商实施的标准接口，该接口使Android无需考虑底层驱动程序的实现。使用HAL可使您实现功能而不会影响或修改更高级别的系统。本页介绍了较旧的架构，该架构从Android 8.0开始不再受支持。对于Android 8.0及更高版本，请参见 [HAL类型](https://source.android.google.cn/devices/architecture/hal-types/)。

![图 1. HAL 组件](https://upload-images.jianshu.io/upload_images/5851256-425e9bc717170610.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

您必须为产品提供的特定硬件实现相应的HAL（和驱动程序）。HAL实现通常内置于共享库模块（.so文件）中，但是由于Android并未强制HAL实现与设备驱动程序之间进行标准交互，因此您可以根据自己的情况采取最佳措施。但是，为了使Android系统能够与您的硬件正确交互，您必须遵守每个特定于硬件的HAL接口中定义的合同。

为了确保HAL具有可预测的结构，每个特定于硬件的HAL接口都具有在中定义的属性 `hardware/libhardware/include/hardware/hardware.h`。此界面允许Android系统以一致的方式加载正确的HAL模块版本。HAL接口包含两个组件：模块和设备。


# 二、HAL 模块介绍
模块代表打包的HAL实现，存储为共享库（.so file）。该 `hardware/libhardware/include/hardware/hardware.h`头文件定义一个结构（hw_module_t），其表示一个模块，并包含元数据，如版本，名称，以及该模块的作者。Android使用此元数据来正确查找和加载HAL模块。

另外，该hw_module_t结构包含指向另一个结构的指针，该结构包含指向hw_module_methods_t模块的打开函数的指针。此开放功能用于启动与HAL用作其抽象的硬件的通信。每个特定于硬件的HAL通常hw_module_t 使用该特定硬件的附加信息来扩展通用结构。例如，在摄像机HAL中，该camera_module_t结构包含一个 hw_module_t结构以及其他摄像机特定的函数指针：

```
typedef struct camera_module {
    hw_module_t common;
    int（* get_number_of_cameras）（void）;
    int（* get_camera_info）（int camera_id，struct camera_info * info）;
} camera_module_t;
```
实现HAL并创建模块struct时，必须将其命名为 `HAL_MODULE_INFO_SYM`。Nexus 9音频HAL中的示例：

```
struct audio_module HAL_MODULE_INFO_SYM = {
    .common = {
        .tag = HARDWARE_MODULE_TAG,
        .module_api_version = AUDIO_MODULE_API_VERSION_0_1,
        .hal_api_version = HARDWARE_HAL_API_VERSION,
        .id = AUDIO_HARDWARE_MODULE_ID,
        .name = "NVIDIA Tegra Audio HAL",
        .author = "The Android Open Source Project",
        .methods = &hal_module_methods,
    },
};
```

# 三、HAL 设备介绍

设备会抽象您产品的硬件。例如，音频模块可以包含主要音频设备，USB音频设备或Bluetooth A2DP音频设备。

设备由该hw_device_t结构表示。类似于模块，每种类型的设备都定义了泛型的详细版本， hw_device_t其中包含针对硬件特定功能的功能指针。例如，audio_hw_device_t结构类型包含指向音频设备操作的函数指针：
```
struct audio_hw_device {
    struct hw_device_t common;

    / **
     *由音频flinger用来枚举受支持的设备
     *每个audio_hw_device实现。
     *
     *返回值是audio_devices_t的1个或多个值的位掩码
     * /
    uint32_t（* get_supported_devices）（const struct audio_hw_device * dev）;
  ...
};
typedef struct audio_hw_device audio_hw_device_t;
```
除了这些标准属性，每个特定于硬件的HAL接口都可以定义更多自己的功能和要求。有关详细信息，请参见[HAL参考文档](https://source.android.google.cn/reference/hal/)以及每个HAL的单独说明。

# 四、构建HAL模块

HAL实现内置于模块（.so）文件中，并在适当时由Android动态链接。您可以通过Android.mk为每个HAL实现创建文件并指向源文件来构建模块。通常，必须以特定格式命名共享库，以便可以找到并正确加载它们。各个模块的命名方案略有不同，但遵循以下一般模式：`<module_type>.<device_name>`。


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

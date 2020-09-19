
####和你一起终身学习，这里是程序员Android




本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
> 一、设备兼容性分类
>二、硬件设备兼容
> 三、软件 APP 兼容
> 四、兼容不同语言
> 五、兼容不同分辨率
> 六、兼容不同屏幕方向布局
>七、兼容不同硬件 Feature
> 八、兼容不同SDK平台


# 一、设备兼容性分类

`Android`设计用于运行在许多不同类型的设备上，从手机到平板电脑和电视机。 作为开发人员，各种设备为您的应用程序提供了巨大的潜在受众。 为了使您的应用程序在所有这些设备上取得成功，`APP`应该容忍一些功能变化，并提供适应不同屏幕配置的灵活的用户界面。

兼容性分类主要分： 硬件兼容性，软件兼容性两大类。

# 二、硬件设备兼容

不同厂商（比如：手机厂商）生产不同尺寸的设备，此时，设备要兼容不同类型的`APP`，`Google`也对此有强烈的要求，国外手机，必须通过`CTS` （兼容性测试）才可以上市售卖。国内手机由于没有预制`GMS`包，不用测试兼容性，故，有时候小厂商生产的手机在兼容性上可能不太完美。

 #三、软件 APP 兼容

作为 `APP`开发者，`APP`兼容性是必须的。兼容不同`Feature`，兼容不同语言、兼容不同屏幕尺寸、兼容不同分辨率，兼容不同`SDK`版本等

# 四、兼容不同语言

为了更加国际化，`APP`通常会兼容不同国家语言，最基本的是兼容英文，简体中文，繁体中文等


## 1. 文件名称命名规则如下：

values-ISO语言代码

## 2 .使用语法：

- java ： 
R.string.<string_name> 引用字符串资源

- XML ：
@string/<string_name> 



- 常用语言如下：
 简体中文 values-zh-rCN
 繁体中文 values-zh-rTW 、values-zh-rHK
 美式英文 values
 英文    values-en-rGB
 
##  3. 兼容不同语言举例

`Android`手机兼容不同国家的语言，进而更方便用户使用。

![Android兼容不同国家语言](http://upload-images.jianshu.io/upload_images/5851256-0429747f566ae70c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 五、兼容不同分辨率
Android 运行在不同的设备上，比如手机、`TV、Car`等设备载体。为了分类这些载体，`Android `设备分两大类：

##  1. 屏幕大小

物理尺寸上的大小 区分如下：
`small, normal, large, and xlarge`

## 2. 屏幕密度（DPI）
 
屏幕像素的物理密度，区分如下：
`mdpi (medium), hdpi (hdpi), xhdpi (extra high), xxhdpi (extra-extra high), and others`


##  3. UI 标准化，常用图片兼容性总结

开发过程中适应不同图片时候的参考总结

 密度 | 建议尺寸|手机屏幕密度DPI|图片分辨率 |基准图缩放倍数
----- |----- | ----| ----|----
| drawable-mdpi  | 48 * 48 | 120dpi ~ 160dpi|320x480|1.0
| drawable-hdpi | 72 * 72 |160dpi ~ 240dpi|480x800、480x854|1.5
| drawable-xhdpi  | 96 * 96 |240dpi ~ 320dpi|960*720|2.0
| drawable-xxhdpi  | 144 * 144 |320dpi ~ 480dpi|1280×720|3.0
| drawable-xxxhdpi  | 192 * 192 |480dpi ~ 640dpi|1920*1080|4.0



## 4.手机屏幕密度DPI获取方法

```
float xdpi = getResources().getDisplayMetrics().xdpi;
float ydpi = getResources().getDisplayMetrics().ydpi;
```
##  5. 兼容屏幕分辨率举例

![兼容不同屏幕分辨率](http://upload-images.jianshu.io/upload_images/5851256-3d8ee8db86a37204.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Android Studio推荐方法](http://upload-images.jianshu.io/upload_images/5851256-beae45f589b5962a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#六、 兼容不同屏幕方向布局（横向 landscape  、纵向 portrait）

虽然`Android` 在横竖屏切换的时候可以自适应，但是，效果经常不是太好，为了更好适应手机屏幕的旋转，横屏、竖屏需要不同的布局，进一步提升UI交互体验。

##1. 兼容不同屏幕方向布局举例：

| 布局|适应屏幕
----|----
layout |   默认纵向
layout-land|  横向布局
layout-large | 大屏纵向 
layout-large-land | 大屏横向
layout-sw600dp| 双窗口布局，常用平板
layout-sw600dp-land| 双窗口布局，常用横向 平板
layout-sw720dp |双窗口布局，常用平板

![兼容不同屏幕大小](http://upload-images.jianshu.io/upload_images/5851256-70b248e1fb1b5fe3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 七、兼容不同硬件 Feature

为了兼顾不同的手机版本，在应用使用不同的`Feature`时候进行判断是否支持，这样会更好的提升用户体验。
比如有些低配手机会没有陀螺仪等`Feature`，此时`APK`要兼容不容的硬件`Feature`。
## 兼容 Feature 的使用方法
例如：在AndroidManifest文件中声明使用Feature
```
<manifest ... >
    <uses-feature android:name="android.hardware.sensor.compass"
                  android:required="true" />
    ...
</manifest>
```
然后在使用该Feature 功能时候进行判断取舍

```
PackageManager pm = getPackageManager();
if (!pm.hasSystemFeature(PackageManager.FEATURE_SENSOR_COMPASS)) {
    // This device does not have a compass, turn off the compass feature
    disableCompassFeature();
}
```
# 八、 兼容不同SDK平台

不同的设备会运行在不同的Android版本上，比如`Android 2.*、Android 4.* 、Android 5.* 、Android6.* 、Android 7.* 、Android 8.*` 。

创建工程时候,在`AndroidManifest.xml`文件中可以选择`APP` 要兼容的`Android`版本

```
<manifest ... >
    <uses-sdk android:minSdkVersion="14" android:targetSdkVersion="19" />
    ...
</manifest>
```
当然也可以在`Java`代码中动态判断当前设备版本，进而执行不同的代码。
```
if (Build.VERSION.SDK_INT < Build.VERSION_CODES.HONEYCOMB) {
    // Running on something older than API level 11, so disable
    // the drag/drop features that use ClipboardManager APIs
    disableDragAndDrop();
}
```

 
 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

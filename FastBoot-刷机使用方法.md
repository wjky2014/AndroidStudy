 

#####  和您一起终身学习，这里是程序员Android 


本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、Fastboot 简介
>二、Fastboot 刷机准备
>三、Fastboot 刷机命令
>四、其他刷机工具


# 一、Fastboot 简介

在安卓手机中`Fastboot`是一种比`recovery`更底层的刷机模式（俗称引导模式）。就是使用`USB`数据线连接手机的一种刷机模式。相对于`Recovery`、Fota等卡刷来说，线刷更可靠，安全。

# 二、Fastboot 刷机准备

 **解锁 BootLoader**

使用 `Fastboot` 刷机必须先解锁`BootLoader `，否则无法刷机。解锁`BootLoader `的方法是在开发者模式中开起`OEM unlocking `开关。如开发者模式隐藏，请进入`Settings`--`System`--`About Phone`--多次点击`build number`  即可打开隐藏的开发者模式。

![打开OEM unlock 允许对BootLoader 进行修改](https://upload-images.jianshu.io/upload_images/5851256-c6b8b102ef8ca531.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![选择允许](https://upload-images.jianshu.io/upload_images/5851256-25efb1a766f88eb5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![不开会导致解锁失败，无法刷机](https://upload-images.jianshu.io/upload_images/5851256-3bf7df5f3449f828.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![解锁成](https://upload-images.jianshu.io/upload_images/5851256-8199cc3be247cfc3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 三、Fastboot 刷机命令

1.进入Fastboot 模式

一般手机常用 `Power `跟 `音量+ ` 进入`fastboot mode`

![fastboot 模式](https://upload-images.jianshu.io/upload_images/5851256-c14232991eb617ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 首先解锁设备
开发者模式打开`oem` 开关后，连接 `USB` ，对设备进行解锁
解锁命令如下：
`fastboot flashing unlock`

![解锁fastboot](https://upload-images.jianshu.io/upload_images/5851256-6dc154aa374c17a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.选择所要刷的镜像
  - 刷 boot分区
 如果修改`kernel `底层代码，需要刷`boot`。 
命令如下 ：
`fastboot flash boot boot.img`

![刷boot 命令](https://upload-images.jianshu.io/upload_images/5851256-62df993e18b9a446.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 刷 system分区
如果修改上层代码，比如增删`apk` 等，需要刷`system`
命令如下：
`fastboot flash system system.img`

![刷 system 命令](https://upload-images.jianshu.io/upload_images/5851256-388e559175d0a740.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 刷 recovery 分区

如果修改到` recovery `模式下的代码，需要刷 `recovery.img`
命令如下
`fastboot flash recovery recovery.img`

![刷recovery 命令](https://upload-images.jianshu.io/upload_images/5851256-39dab2689b91650d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 擦除Frp 分区

`frp` 即 **Factory Reset Protection**，用于防止用户信息在手机丢失后外泄
命令如下：
` fastboot  erase  frp`

![擦除Frp 分区](https://upload-images.jianshu.io/upload_images/5851256-be77afc0a812a2b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 设备上锁

刷完之后，给设备上锁
命令如下：
`fastboot  flashing lock`

![设备上锁](https://upload-images.jianshu.io/upload_images/5851256-058f729c147df8a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 退出Fastboot ，重启手机

退出 `Fastboot `重启手机命令如下：
`  fastboot  continue`

![退出fastboot ，重启手机](https://upload-images.jianshu.io/upload_images/5851256-139966cfc5ec8957.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



- Fastboot 常用命令

![fastboot 常用命令 如下](https://upload-images.jianshu.io/upload_images/5851256-a5c630acbf52a866.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 四、其他刷机工具

其实平台厂商也会有一些线刷工具，这些工具也可以单刷一些模块。
MTK平台：Flashtools
[点击下载](https://pan.baidu.com/s/1X5tSsn7Lzii82BNaG2dv4Q)
![MTK 平台刷机工具](https://upload-images.jianshu.io/upload_images/5851256-98a6de6f162ec347.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

展讯平台 ：ResearchDownload
[点击下载](https://pan.baidu.com/s/1ztoIY6wylxxDDzuxscQ91g)

![展讯平台刷机工具](https://upload-images.jianshu.io/upload_images/5851256-2ae7c4d9ee57f4d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


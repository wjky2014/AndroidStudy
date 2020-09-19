#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
 >一、BugReport 总体概览
>二、电池电量变化
>三、Doze 模式分析
>四、通过索尼的CkBugreport分析Log
>五、手机端抓取bugreport方法

# 一、BugReport 总体概览

Bugreport 总体概览信息如下：

![Bugreport 总体概览信息](https://upload-images.jianshu.io/upload_images/5851256-87b98d1228eca142.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#二、电池电量变化

## 1.电池电量总体变化图

![电池电量总体变化图](https://upload-images.jianshu.io/upload_images/5851256-bb651524d5eac9f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.图标1 明显下降点（100%-93%）分析

图标 1. Screen on 状态，CPU 运行频繁，此时电池温度，耗电量都会增加，掉电曲线比较明显 。
Screen on 状态 耗电分析如下：

![Screen on 状态 耗电分析](https://upload-images.jianshu.io/upload_images/5851256-bef6d8ec274b31c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此时主要运行的Top APP如下：

![主要 Top app如下](https://upload-images.jianshu.io/upload_images/5851256-443002f5c2a97e32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##3.图标2 明显下降点（87%-86%）分析


  图标 2. 系统已经进入Doze 睡眠模式， 此时按Power 键唤醒屏幕，此时Screen on状态，根据后续的wifi on状态断开，推测此时的操作为点亮屏幕，关闭 wifi。

![按Power键亮屏](https://upload-images.jianshu.io/upload_images/5851256-0e6c6a0e48b0c6cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 4. 图标3 明显下降点（81%-72%）分析


 图标3 . 后台 app 执行一些 Alarm 操作，进而导致耗电曲线明显下降，此时查看Alarm 有以下两个进程（Fota Service，Google Alarm）在唤醒CPU。

![Fota Service，Google Alarm](https://upload-images.jianshu.io/upload_images/5851256-d05be07787fdb2c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 - Fota Service

![fota Service](https://upload-images.jianshu.io/upload_images/5851256-a1634dc2378a7b91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- Google Alarm

![Google alarm](https://upload-images.jianshu.io/upload_images/5851256-0cc18f32e9cbc936.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 三、Doze 模式 分析

## 1.Doze 过程概览 

Doze 过程概览 如下：

 ![Doze 过程概览](https://upload-images.jianshu.io/upload_images/5851256-d89296a35c943032.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.Doze 过程分析

图标1 明显下降点（100%-93%）分析如下：

![图标1 明显下降点（100%-93%）分析](https://upload-images.jianshu.io/upload_images/5851256-38ab26f27bf1420e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图标2 明显下降点（87%-86%）分析如下：

![图标2 明显下降点（87%-86%）分析](https://upload-images.jianshu.io/upload_images/5851256-c2db8e369108b58d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 四、通过索尼的CkBugreport分析Log

通过索尼的CkBugreport 分析所提提供的log发现，正在睡眠过程中，google Message 有在频繁报错,由于没有Google Message 源码，建议更新GMS 后再次复测一下，看看 Google Message 是否修复此问题。

报错log如下：

![ ](https://upload-images.jianshu.io/upload_images/5851256-fa8e85ed57677d86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 五、手机端抓取bugreport方法

手机端需要在 打开开发者模式---> Bugreport 

![手机端抓取bugreport的地方](https://upload-images.jianshu.io/upload_images/5851256-8b10d41879e7fb1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
生成的bugreport 文件保存路径如下：
![生成的文件路径](https://upload-images.jianshu.io/upload_images/5851256-df1f954a25dbb6d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以使用如下adb 命令 pull 出来
```
adb pull data/user_de/0/com.android.shell/files/bugreports/ .
```
![adb pull 出bugreport](https://upload-images.jianshu.io/upload_images/5851256-9dc2b97d968f481c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

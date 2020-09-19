 

##### 您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的**Google play Store** 应用 部分知识点，通过阅读本篇文章，您将收获以下内容:


## 一、Google Play Store 应用无法安装解决方案

### Google PlayStore应用无法安装的原因：

**1. 国家或地区限制导致无法安装**

某些应用只在某些国家和地区才能使用，所以`Google PlayStore`会根据用户当前网络情况屏蔽这些应用；

此情况属于正常情况，可以使用对比机在同样的网络环境下验证。如果您确实需要下载，则可通过`vpn`翻墙搜索下载。

**2. 手机`feature`不支持导致无法安装**

关于`feature` 有以下两种行为

- a.手机确实没有相应`feature`，如`GPS`

属于正常情况，如果您一定需要下载，则可通过强制声明此`feature`方式下载，但是不能保证下载后可以正常安装以及使用。

- b.手机有但是未声明相应`feature`，导致系统显示为缺少对此`feature`的支持

此类问题解决方案如下：

1. 确保对比机在同样网络条件下可以搜索到，并将此应用下载下来(某某.apk)

2. 获取 apk Feature 要求

**aapt**（`AndroidAssetPackagingTool`）在SDK的`\sdk\build-tools\27.0.3 `目录下
使用`aapt`命令可以解析`apk `信息
解析命令如下：       
```
aapt dump badging file.apk > 某某.xml
```
![使用adb 命令获取 应用，手机Feature ](https://upload-images.jianshu.io/upload_images/5851256-debc2a46ccd7e47b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 此命令用于查看APK包的`packageName、versionCode、applicationLabel、launcherActivity、permission`等各种详细信息，请记录应用`uses-feature`和`uses-library`项
![举例 app 申请权限等](https://upload-images.jianshu.io/upload_images/5851256-a8941851029885c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 查看手机Feature 支持情况

可以使用以下命令`dump `手机`library`和`feature`信息
```
adb shell dumpsys package > 某某.xml
```
![举例获取手机 Feature ](https://upload-images.jianshu.io/upload_images/5851256-6ab559b4b1ecdc7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 对比应用需要的与手机声明的`feature`和`library`，补上手机缺少的相应`feature`，声明各个`feature`的位置可能根据`feature`不同而在不同的文件里


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

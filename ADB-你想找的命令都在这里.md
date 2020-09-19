

##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、ADB 简介
>二、ADB的工作方式
>三、ADB常用命令




# 一、ADB 简介

ADB（Android Debug Bridge） 是一个通用命令行工具，其允许您与模拟器实例或连接的 Android 设备进行通信。它可为各种设备操作提供便利，如安装和调试应用，并提供对 Unix shell（可用来在模拟器或连接的设备上运行各种命令）的访问。该工具作为一个客户端-服务器程序。

- **客户端**，该组件发送命令。客户端在开发计算机上运行。您可以通过发出 adb 命令从命令行终端调用客户端。

- **后台程序**，该组件在设备上运行命令。后台程序在每个模拟器或设备实例上作为后台进程运行。

- **服务器**，该组件管理客户端和后台程序之间的通信。服务器在开发计算机上作为后台进程运行。

**adb 工具路径**
`android_sdk/platform-tools/`

# 二、ADB的工作方式

### 1.  连接 Android 模拟器

ADB与本地 TCP 端口 5037 绑定，并侦听从 adb 客户端发送的命令—所有 adb 客户端均使用端口 5037 与 adb 服务器通信。然后，服务器设置与所有运行的Android模拟器/Android 设备连接。

### 2.USB 连接 Android 机器

- a. 打开开发者选项

**Settings**  >**About phone**>连续点击 **Build number**七次

- b.  开启 **USB debugging**

- c .**Dos** 下输入**adb devices**验证手机是否连上 **adb**

![验证adb 是否连接成功](https://upload-images.jianshu.io/upload_images/5851256-5ce62fb340c87eb4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3.WLAN  连接 Android 机器
 此方法不常用，暂时忽略

# 三、ADB常用命令

### 1. 安装卸载apk

- a. 安装apk 

`adb install  apk路径`

![安装apk](https://upload-images.jianshu.io/upload_images/5851256-8eb3e7d108b5798b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 - b. 卸载apk

`adb uninstall apk包名`

![查询包名，并根据包名卸载apk](https://upload-images.jianshu.io/upload_images/5851256-b21f34a0ecd379d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2. 导入/导出 文件到手机中

- a. 导出手机文件

`adb pull remote local`

![将手机Setting.apk 导出到电脑D盘](https://upload-images.jianshu.io/upload_images/5851256-0047cb00ec137c47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- b. 导入文件到手机

`adb push local remote`

![将电脑D 盘的文件 导入到手机/system/priv-app/Settings目录下](https://upload-images.jianshu.io/upload_images/5851256-36272188ceb0eea2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3.开启、停止ADB 服务

- a. 开启ADB 服务

`adb start-server`

- b. 停止ADB服务 

`adb kill-server`

![ADB 服务的开启与停止](https://upload-images.jianshu.io/upload_images/5851256-933edcdaf3c98e58.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 4. 使用ADB 命令截屏、录像

- a. 截屏 **screencap** 

`adb shell screencap 文件保存路径`

![使用adb 命令截图](https://upload-images.jianshu.io/upload_images/5851256-e33b0479aef42428.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- b. 录像 **screenrecord**

`adb shell screenrecord 文件保存路径`

![使用adb 命令录屏录像](https://upload-images.jianshu.io/upload_images/5851256-619af7fafc17f3dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

按 **Control + C** 停止屏幕录制，否则，到三分钟或 `--time-limit` 设置的时间限制时，录制将自动停止。

**screenrecord 部分参数** 

选项|说明
----|----
--size width x height | 设置分辨率 eg：1280x720
--bit-rate rate |视频比特率,默认值为 4Mbps,可以设6Mbps，这样质量更好 eg:`adb shell screenrecord --bit-rate 6000000 /sdcard/demo.mp4`
--time-limit time |设置最大录制时长（以秒为单位）。默认值和最大值均为 180（3 分钟）。

### 5. 调用ActivityManager（am 命令） 

- a. 发送 intent

`adb shell am start -a android.intent.action.VIEW`

- b.启动Activity

`adb shell am start -n 包名/类名 `

![启动QQ ](https://upload-images.jianshu.io/upload_images/5851256-366bb68594ac2eed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- c. 启动service

`adb shell am startservice 包名/类名`

![启动指定的Service](https://upload-images.jianshu.io/upload_images/5851256-04a7c6533636e6ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- d. 发送广播

`adb shell am boradcast -a  广播Action`

![adb 命令发送开机广播](https://upload-images.jianshu.io/upload_images/5851256-1c29e5830372b644.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- e. 强行停止应用

`adb shell force-stop 包名`

![强行停止QQ进程，正在使用的QQ就会闪退被杀掉](https://upload-images.jianshu.io/upload_images/5851256-0a73e35e59ca0a7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 6. 调用 PackageManager（pm 命令）

- a. 卸载apk

`adb shell pm uninstall 包名`

![卸载QQ](https://upload-images.jianshu.io/upload_images/5851256-05582cb5ce2c6048.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- b. 查看手机中所有apk 包名

`adb shell pm list packages`

![部分apk包名查看](https://upload-images.jianshu.io/upload_images/5851256-b7c880b18426a04b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- c. 查看已知权限组

`adb shell pm list permission-groups`

![所有手机权限组查看](https://upload-images.jianshu.io/upload_images/5851256-c04854e02283e877.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- d. 查看手机Feature 支持

`adb shell pm list features`

![查看手机Feature ](https://upload-images.jianshu.io/upload_images/5851256-c9a7ba7601743155.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- e. 根据包名，查看apk 安装路径

**adb shell pm path 包名**

![查看SystemUI apk 路径](https://upload-images.jianshu.io/upload_images/5851256-df01f4587d0ccd09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- f. 清除app 数据

`adb shell pm clear 包名`

![清除QQ apk 数据](https://upload-images.jianshu.io/upload_images/5851256-d0d007868572e0a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- g. 多用户相关

查看支持最多用户数
`adb shell pm get-max-users`

查询系统所有用户
`adb shell pm list users`

创建新用户
`adb shell pm create-user user_name`

移除指定id用户
`adb shell pm remove-user user_id`

![测试发现只有 使用adb 命令创建的多用户才可用命令移除](https://upload-images.jianshu.io/upload_images/5851256-4c75fa85c5612e0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 7. dumpsys将系统数据转储到屏幕

- a. 获取当前运行的Activity

`adb shell dumpsys activity | findstr Run`

![获取最近运行的Activity ，已经Top Activity](https://upload-images.jianshu.io/upload_images/5851256-9f6782dc3b580d4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- b. 获取apk 版本号，权限等信息的方法
`adb shell dumpsys package com.xxx.xxx(包名) `

![adb 获取apk 版本号，权限等](https://upload-images.jianshu.io/upload_images/5851256-e3627efea7e2b7fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 8. 查看手机系统进程

- a.使用Top命令查看系统进程

`adb shell top`

![使用Top命令查看系统进程](https://upload-images.jianshu.io/upload_images/5851256-887fdcd1d82bccf9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- b. 使用 **ps** 命令查看系统进程

`adb shell ps`

![使用 ps 命令查看系统进程](https://upload-images.jianshu.io/upload_images/5851256-bcae72c3b3ce03e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

结合findstr 命令 过滤多余的信息 `adb shell ps | findstr qq`

![结合findstr 命令 过滤多余的信息](https://upload-images.jianshu.io/upload_images/5851256-299cfdd395bb5ad1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 9. 使用logcat抓 log信息

- a.使用 logcat 抓取log信息

`adb logcat > 1.txt`

![使用logcat 抓取的信息](https://upload-images.jianshu.io/upload_images/5851256-328d5af5f5be380e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- b.使用 **-s** 过滤log标签

`adb logcat -s 关注log标签`

![使用-s 过滤关注log标签](https://upload-images.jianshu.io/upload_images/5851256-d44b167f28ac017a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- c. 使用 **-c** 清除缓存log

`adb logcat -c`

### 10. 电量管理相关命令

- a.模拟拔下设备电源

`adb shell dumpsys battery unplug`

- b. 低电量条件下的行为

`adb shell settings put global low_power 1`

- c .恢复电源修改

` adb shell dumpsys battery reset`

![电源管理相关命令](https://upload-images.jianshu.io/upload_images/5851256-09017e27e916cc8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 11. 使用adb 命令进入recovery 模式
进入Recovery 模式可以使用组合键，也可以使用adb 命令
adb 命令进入recovery 模式如下`adb reboot recovery`
![adb 命令进入recovery模式 ](https://upload-images.jianshu.io/upload_images/5851256-44f63c2d0f511b42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 12. 跳过Google 开机向导的命令

```
adb shell pm disable com.google.android.setupwizard
adb shell settings put global device_provisioned 1
adb shell settings put secure user_setup_complete 1
```
启动开机向导命令
```

C:\Users\Administrator>adb shell am start  com.google.android.setupwizard/.user.WelcomeActivity
Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=com.google.android.setupwizard/.user.WelcomeActivity }

C:\Users\Administrator>
```
## 13. logcat 抓取kernel log方法
```
adb shell logcat -b kernel > kernel.txt
```

## 14. 查看手机屏幕亮度的方法
```
C:\Users\Administrator>adb shell cat /sys/class/leds/lcd-backlight/brightness
255

C:\Users\Administrator>
```
## 15. 查看手机Permission 授权方法

```
C:\Users\Administrator>adb shell dumpsys  package permissions
  ... ... 
  AppOp Permission android.permission.ACCESS_NOTIFICATIONS:
    com.android.settings
  AppOp Permission android.permission.REQUEST_INSTALL_PACKAGES:
    com.gameloft.android.GloftANPH
    com.gameloft.android.GloftDBMF
    com.google.android.gm
    com.google.android.apps.nbu.files
    com.google.android.apps.docs
    com.opera.browser
    com.android.chrome
    com.android.bluetooth
```



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


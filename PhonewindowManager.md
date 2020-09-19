 
#####  和您一起终身学习，这里是程序员Android 



本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、Android 按键修改
>二、PhoneWindowManager 简介
>三、如何打开 或者 关闭 Navigation Bar
>四、如何长按Home 键启动Google Now
>五、如何长按实体Menu键进入多窗口模式
>六、如何点击 Menu键进入调出最近任务列表
>七、如何让 App 拿到Power key 值
>八、如何修Activity启动背景窗口
>九、WindowManagerPolicy 简介


#  一、Android 按键修改 

在`Android`  中会有以下`5`个按键`（Back`、`Home`、`Menu`、`Power`、`Volume`）与用户进行交互，`Framework `层中实现按键功能，因此，从手机系统定制的角度，可以满足客户的客制化要求。本文主要从`Framework`层浅析这些客制化需求的实现。

![Back、Home、Menu、Power、Volume 按键图 ](http://upload-images.jianshu.io/upload_images/5851256-de46438a76bac08a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
以`MTK` 平台为例，按键客制化的代码主要存放在以下类中

- 1. PhoneWindowManager

`PhoneWindowManager `代码路径如下：
```
alps\frameworks\base\services\core\java\com\android\server\policy\PhoneWindowManager.java
alps\frameworks\base\core\java\android\view\WindowManagerPolicy.java
```


#  二、 PhoneWindowManager 简介

`PhoneWindowManager`  类实现接口如下：

```
java.lang.Object
    ↳  android.view.WindowManagerPolicy.java
         ↳ com.android.server.policy.PhoneWindowManager.java
```
![ PhoneWindowManager 类实现关系](http://upload-images.jianshu.io/upload_images/5851256-4093c19fa6f756ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`PhoneWindowManager`主要用于实现各种实体或虚拟按键处理，如需特殊处理按键，请修改源码。

# 三、  如何打开 或者 关闭 Navigation Bar

![虚拟导航栏](http://upload-images.jianshu.io/upload_images/5851256-341a1d031a6cad51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如何打开 或者 关闭 Navigation Bar 的解决方法如下：

## 1. 修改config.xml 文件中 

搜索关键字` config_showNavigationBar`， 查看 `config_showNavigationBar` 值 
`true` 表示显示,`false` 表示不显示


```
  <!-- Whether a software navigation bar should be shown. NOTE: in the future this may be
         autodetected from the Configuration. -->
    <bool name="config_showNavigationBar">true</bool>
```

参考路径如下：
`alps\frameworks\base\core\res\res\values\config.xml`


##  2. 修改 system.prop 文件

查询关键字 `qemu.hw.mainkeys `，并查看值，`1 `表示关闭`  0`.表示开启 。

```
# temporary enables NAV bar (soft keys)
qemu.hw.mainkeys=1
```
不同项目文件存放地址不一样，可以使用以下命令查找
终端下查找文件方法
```
find 路径 -name "文件名.java"
````
或者直接查找文件中的字符串
```
 find 路径 -type f -name "文件名" | xargs grep "文件中的字符串"

```
##  3. 修改PhoneWindowManager代码

如果上面两个修改都不生效（搜索关键字`config_showNavigationBar`、`qemu.hw.mainkeys`），请在`PhoneWindowManager` 查看`setInitialDisplaySize`方法中`mHasNavigationBar ` 的值是否被写死，`true`表示会显示、`false `表示不显示导航栏。

![底部导航卡显示代码控制](http://upload-images.jianshu.io/upload_images/5851256-b31c7cd8f85b37ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#四、 如何长按Home 键启动Google Now 

##  1. 预制 `Google Now APK `

请自行安装`APK`

##  2. 修改 PhoneWindowManager 代码

长按`Home`键启动`Google Now `,实现方法参考`launchAssistLongPressAction` 功能实现。
![PhoneWindowManager 长按Home 建启动Google Now](http://upload-images.jianshu.io/upload_images/5851256-9dcd9ebdc3384a11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



自己实现常按`Home` 键吊起`Google Now `方法，供在按键分发处理事件时候调用。

![自己实现常按Home 键吊起Google Now 方法](http://upload-images.jianshu.io/upload_images/5851256-2395cc23a5eded62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##  3. 在按键事件分发之前处理

在按键分发处理之前调用自定义长按`Home `键的方法


![自定义长按Home 键的方法
](http://upload-images.jianshu.io/upload_images/5851256-75f2cea8c99648e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##   4.双击Home 键调出最近任务列表请用以下方法



在` phoneWindowManager.java` 的` interceptKeyBeforeQueueing` 方法中修改
修改方法如下：

![双击Home 键调出最近任务列表](http://upload-images.jianshu.io/upload_images/5851256-beef518b2a6d64f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#五、 如何长按实体Menu键进入多窗口模式

`Android  N`上支持`Multi-Window`，通过`recent key `进入多窗口，对于没有打开虚拟导航栏，只有实体`menu`按键的手机，可以考虑向`SystemUI`发送广播的形式，进入`Android` 分屏多任务模式。
解决方案如下：

## 1.  PhoneStatusBar 里注册广播

`PhoneStatusBar` 是`SystemUI`模块的代码，参考路径如下：

`frameworks/base/packages/SystemUI/src/com/android/systemui/statusbar/phone/PhoneStatusBar.java
`

自定义广播实现可以参考系统`mDemoReceiver` 的实现方法
动态注册广播方法如下：
![自定义广播注册](http://upload-images.jianshu.io/upload_images/5851256-2d408a6d768fbae0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

自定义接收广播后，`onReceive`处理事件实现分屏方法如下：

![自定义接收广播处理](http://upload-images.jianshu.io/upload_images/5851256-9c6698cb5fc95cbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 2.  PhoneWindowManager 中发送广播

在 `PhoneWindowManager  `的`interceptKeyBeforeDispatching`方法中发送广播
 
![interceptKeyBeforeDispatching 发送广播](http://upload-images.jianshu.io/upload_images/5851256-2fbf28f4fb8d392c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##  3. Destory 方法注销广播

再`Destory `方法中记得一定要注销广播

```
 mContext.unregisterReceiver(mDemoReceiver);
 mContext.unregisterReceiver(mAppLongSwitchReceiver);
```
# 六、 如何点击 Menu键进入调出最近任务列表

如果想调出最近任务列表，需要拦截`menu`的事件，在`PhoneWindowManager`的`interceptKeyBeforeDispatching 中`处理即可
![menu 键调出最近任务列表](http://upload-images.jianshu.io/upload_images/5851256-aba1510cb5bbe2b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果想`长按Menu `调出可以使用以下方法

![长按menu 键调出任务列表](http://upload-images.jianshu.io/upload_images/5851256-81974eb2fa11ffdf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 七、 如何让 App 拿到Power key 值

一般情况下`App `是拿不到`Power`的`Key`值，但通过以下方法可以实现。

##  1. 修改PhoneWindowManager 文件实现

在`PhoneWindowManager` 中修改`interceptKeyBeforeQueueing `方法实现让特定的`APP`拿到`Power key` 值

![power key 启动App ](http://upload-images.jianshu.io/upload_images/5851256-edb1c538d071c990.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##  2. 如果只想让某个app的某个Activity 处理

![Power 键启动Activity 的方法](http://upload-images.jianshu.io/upload_images/5851256-bfb91ccf73572230.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 八、 如何修Activity启动是的窗口（app启动白屏，黑屏问题）

当用户从主菜单进入其他应用程序例如时钟、联系人、文件管理等时，可能会出现屏幕闪一下黑屏、白屏等问题，这种现象在当前手机主题`(Theme)`是浅色（例如白色）的情况下比较明显。

此所谓的闪"黑屏",其实是应用程序的启动窗口。
启动窗口出现的条件如下：

1. 仅在要启动的`Activity`在新的`Task`或者新的`Process`时，才可能显示启动窗口

2. 启动窗口先于`Activity`窗口显示，当`Activity`窗口的内容准备好之后，启动窗口就会被移除掉，`show`出真正的`activity` 窗口

3. 启动窗口和普通的`Activity window`类似，只是没有画任何内容，默认是一个黑色背景的窗口

正是由于启动窗口默认是黑色背景的，所以在当前的手机主题为浅色调的时候，就比较容易因为颜色的深浅对比而产生一种视觉上的闪动感。

解决方法如下：

##  1.去掉启动窗口

在 `ActivityStack.java`中将`SHOW_APP_STARTING_PREVIEW  `设置为`false `既可

## 2. 修改启动窗口样式

在 `PhoneWindowManager`中的`addStartingWindow `方法中添加自定义样式或者背景等

![修改启动窗口样式](http://upload-images.jianshu.io/upload_images/5851256-7597b112784f32c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 # 九、 WindowManagerPolicy 简介

PhoneWindowManager 实现 的接口类如下：

`alps\frameworks\base\core\java\android\view\WindowManagerPolicy.java`

![WindowManagerPolicy 接口实现](http://upload-images.jianshu.io/upload_images/5851256-96434162e6426cf8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

WindowManagerPolicy 是一个接口类，主要对外提供一些接口。
常用接口如下：
![WindowState 接口](http://upload-images.jianshu.io/upload_images/5851256-2144202d5232cfe5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![WindowMangerFuncs接口](http://upload-images.jianshu.io/upload_images/5851256-b7544ad19858ed56.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Screen On 接口](http://upload-images.jianshu.io/upload_images/5851256-7d8eb27567fe5055.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Keyguard 接口](http://upload-images.jianshu.io/upload_images/5851256-e7185309a61d7a51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

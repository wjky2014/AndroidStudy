 #####  和您一起终身学习，这里是程序员Android
本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、priv-app 白名单简介
>二、添加priv-app 白名单
>三、生成priv-app白名单
>四、自定义priv-app白名单
>五、查找缺少的priv-app权限
>六、执行priv-app白名单
>七、priv-app 中申请权限未添加特许白名单遇到的坑

# 一、priv-app 白名单简介

特权应用程序是位于`/system/priv-app`系统映像目录中的系统应用程序 。从历史上看，设备实施者几乎无法控制哪些特权权限可以授予特权应用程序。从`Android 8.0`开始，实现者可以在`/etc/permissions`目录中的系统配置`XML`文件中明确地将特权应用程序列入白名单。未在这些XML文件中明确列出的应用程序未被授予特权权限。

**特别注意事项**
仅对 具有`package =“android”`的应用程序声明的 权限才需要白名单 。`Google Play`进展使用`com.example`和`com.android`命名空间。

#二、添加priv-app 白名单
应用程序的权限白名单可以列在`frameworks/base/etc/permissions `目录中的单个或多个`XML`文件中，如下所示：
```
frameworks/base/etc/permissions/privapp-permissions-OEM_NAME.xml
frameworks/base/etc/permissions/privapp-permissions-DEVICE_NAME.xml
```
组织内容没有严格的规则。只要所有应用程序/system/priv-app都列入白名单，设备实施者就可以确定内容结构 。例如，`Google`为`Google`开发的所有特权应用程序都有一个白名单。我们建议使用以下组织：

- 已包含在Android开源项目（AOSP）中的应用的权限列于
` /etc/permissions/privapp-permissions-platform.xml`。
- `Google`应用程序的权限列于
` /etc/permissions/privapp-permissions-google.xml`。
- 对于其他应用程序，请使用以下形式的文件：
` /etc/permissions/privapp-permissions-DEVICE_NAME.xml`

#三、生成priv-app白名单

要为系统映像上的所有可用应用程序自动生成白名单，请使用`AOSP`命令行工具 `development/tools/privapp_permissions/privapp_permissions.py`。要生成特定于设备的初始版本 `privapp-permissions.xml`

1.构建系统映像：
```
  . build/envsetup.sh
    lunch PRODUCT_NAME
    make -j
```
2.运行该`privapp_permissions.py`脚本以生成一个`privapp-permissions.xml`文件，该 文件列出了要列入白名单的所有签名特权权限：
```
development/tools/privapp_permissions/privapp_permissions.py
```
此工具生成  `XML`内容，可以将其用作单个文件或拆分为多个文件`/etc/permissions`。如果设备已在`/etc/permissions`目录中包含白名单，则 该工具仅打印差异（即缺少要添加到白名单的特权权限）。这对于审计目的也很有用：添加新版本的应用程序时，该工具会检测所需的其他权限。
3.将生成的文件复制到`/etc/permissions`目录，系统将在引导期间读取这些文件。

#四、自定义priv-app白名单

AOSP包括白名单实现，可根据需要进行自定义。AOSP中包含的应用的权限已列入白名单
`/etc/permissions/privapp-permissions-platform.xml。`

默认情况下，`privapp_permissions.py`脚本会生成输出，该输出会自动授予特权应用程序请求的任何权限。如果存在应拒绝的权限，请编辑XML以使用“拒绝权限”标记而不是“权限”标记。例如`（frameworks/base/data/etc/privapp-permissions-platform.xml）`：

```
<!--
    This XML file declares which signature|privileged permissions should be
    granted to privileged applications that come with the platform
    -->
    <permissions>
<privapp-permissions package="com.android.backupconfirm">
    <permission name="android.permission.BACKUP"/>
    <permission name="android.permission.CRYPT_KEEPER"/>
</privapp-permissions>
<privapp-permissions package="com.android.cellbroadcastreceiver">
    <!-- don't allow application to interact across users -->
    <deny-permission name="android.permission.INTERACT_ACROSS_USERS"/>
    <permission name="android.permission.MANAGE_USERS"/>
    <permission name="android.permission.MODIFY_PHONE_STATE"/>
    <permission name="android.permission.READ_PRIVILEGED_PHONE_STATE"/>
    <permission name="android.permission.RECEIVE_EMERGENCY_BROADCAST"/>
</privapp-permissions>
    ...
```

#五、查找缺少的priv-app权限

启动新设备时，通过启用过渡日志模式来查找缺少的权限：
```
ro.control_privapp_permissions=log
```
可以使用`adb 命令`获取该属性的值，命令如下：
```
C:\Users\Administrator>adb root
C:\Users\Administrator>adb remount
Not running as root. Try "adb root" first.
C:\Users\Administrator>adb shell getprop ro.control_privapp_permissions
log
C:\Users\Administrator>
```
![adb 获取特权白名单属性 ](https://upload-images.jianshu.io/upload_images/5851256-836f1204151825e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

日志文件中会报告违规，但仍会授予权限。这样可以在提供违规列表的同时使设备保持工作状态。错误消息格式如下：
```
PackageManager: Privileged permission {PERMISSION_NAME} for package {PACKAGE_NAME} - 
 not in privapp-permissions whitelist
```
必须通过将应用添加到白名单来解决所有违规问题。如果未添加，即使应用程序位于priv-app路径中，也不会向应用程序授予缺少的权限。

#六、执行priv-app白名单
在使用白名单后，通过设置构建属性来启用运行时强制实施`ro.control_privapp_permissions=enforce`

# 七、priv-app 中申请权限未添加特许白名单遇到的坑

如果`priv-app`中申请了一些权限后没有在此文件添加，在强制执行`ro.control_privapp_permissions=enforce`时候，可能会导致手机无法开机情况。

小编遇到无法开机的异常`Log`信息如下：
```
06-04 10:48:38.750  1756  1756 E AndroidRuntime: *** FATAL EXCEPTION IN SYSTEM PROCESS: main
06-04 10:48:38.750  1756  1756 E AndroidRuntime: java.lang.IllegalStateException: Signature|privileged permissions not in privapp-permissions whitelist: {com.android.cellbroadcastreceiver: android.permission.MANAGE_ACTIVITY_STACKS, com.android.cellbroadcastreceiver: android.permission.STATUS_BAR}
06-04 10:48:38.750  1756  1756 E AndroidRuntime: 	at com.android.server.pm.permission.PermissionManagerService.systemReady(PermissionManagerService.java:2010)
06-04 10:48:38.750  1756  1756 E AndroidRuntime: 	at com.android.server.pm.permission.PermissionManagerService.access$100(PermissionManagerService.java:89)
06-04 10:48:38.750  1756  1756 E AndroidRuntime: 	at com.android.server.pm.permission.PermissionManagerService$PermissionManagerInternalImpl.systemReady(PermissionManagerService.java:2057)
06-04 10:48:38.750  1756  1756 E AndroidRuntime: 	at com.android.server.pm.PackageManagerService.systemReady(PackageManagerService.java:21446)
06-04 10:48:38.750  1756  1756 E AndroidRuntime: 	at com.android.server.SystemServer.startOtherServices(SystemServer.java:1803)
06-04 10:48:38.750  1756  1756 E AndroidRuntime: 	at com.android.server.SystemServer.run(SystemServer.java:453)
06-04 10:48:38.750  1756  1756 E AndroidRuntime: 	at com.android.server.SystemServer.main(SystemServer.java:316)
06-04 10:48:38.750  1756  1756 E AndroidRuntime: 	at java.lang.reflect.Method.invoke(Native Method)
06-04 10:48:38.750  1756  1756 E AndroidRuntime: 	at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:504)
06-04 10:48:38.750  1756  1756 E AndroidRuntime: 	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:838)
```
**解决方案**
根据无法开机的报错信息，需要将小区广播中申请的权限添加到`privapp-permissions-platform`白名单中。

1.小区广播申请权限`AndroidManifest.xml`如下：

```
 1 <?xml version="1.0" encoding="utf-8"?>
   2 <!--
   3 /*
   4  * Copyright (C) 2011 The Android Open Source Project
   5  *
   6  * Licensed under the Apache License, Version 2.0 (the "License");
   7  * you may not use this file except in compliance with the License.
   8  * You may obtain a copy of the License at
   9  *
  10  *      http://www.apache.org/licenses/LICENSE-2.0
  11  *
  12  * Unless required by applicable law or agreed to in writing, software
  13  * distributed under the License is distributed on an "AS IS" BASIS,
  14  * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  15  * See the License for the specific language governing permissions and
  16  * limitations under the License.
  17  */
  18 -->
  19 <manifest xmlns:android="http://schemas.android.com/apk/res/android"
  20         package="com.android.cellbroadcastreceiver"
  21         android:sharedUserId="com.android.cellbroadcastreceiver.plugins">
  22 
  23     <original-package android:name="com.android.cellbroadcastreceiver" />
  24     <uses-sdk android:minSdkVersion="20" android:targetSdkVersion="28" />
  25     <uses-permission android:name="android.permission.RECEIVE_SMS" />
  26     <uses-permission android:name="android.permission.RECEIVE_EMERGENCY_BROADCAST" />
  27     <uses-permission android:name="android.permission.READ_PRIVILEGED_PHONE_STATE" />
  28     <uses-permission android:name="android.permission.MODIFY_PHONE_STATE" />
  29     <uses-permission android:name="android.permission.WAKE_LOCK" />
  30     <uses-permission android:name="android.permission.DISABLE_KEYGUARD" />
  31     <uses-permission android:name="android.permission.VIBRATE" />
  32     <uses-permission android:name="android.permission.INTERACT_ACROSS_USERS" />
  33     <uses-permission android:name="android.permission.MANAGE_USERS" />
  34     <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
  35     <uses-permission android:name="android.permission.WRITE_SETTINGS" />
  36     <uses-permission android:name="android.permission.STATUS_BAR_SERVICE" />
  37     <uses-permission android:name="android.permission.STATUS_BAR" />
  38     <uses-permission android:name="android.permission.EXPAND_STATUS_BAR" />
  39     <uses-permission android:name="android.permission.MANAGE_ACTIVITY_STACKS" />
  40 
  41     <application android:name="CellBroadcastReceiverApp"
  42             android:label="@string/app_label"
  43             android:supportsRtl="true"
  44             android:icon="@mipmap/ic_launcher"
  45             android:backupAgent="CellBroadcastBackupAgent"
  46             android:defaultToDeviceProtectedStorage="true"
  47             android:directBootAware="true">
            ... ...
 205 
 206     </application>
 207 </manifest>
```

2.需要将 priv-app 在AndroidMainfest.xml 中申请的权限 添加到特许白名单中`（frameworks/base/data/etc/privapp-permissions-platform.xml）`
```
   1 <?xml version="1.0" encoding="utf-8"?>
   2 <!--
   3   ~ Copyright (C) 2016 The Android Open Source Project
   4   ~
   5   ~ Licensed under the Apache License, Version 2.0 (the "License");
   6   ~ you may not use this file except in compliance with the License.
   7   ~ You may obtain a copy of the License at
   8   ~
   9   ~      http://www.apache.org/licenses/LICENSE-2.0
  10   ~
  11   ~ Unless required by applicable law or agreed to in writing, software
  12   ~ distributed under the License is distributed on an "AS IS" BASIS,
  13   ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  14   ~ See the License for the specific language governing permissions and
  15   ~ limitations under the License
  16   -->
  17 
  18 <!--
  19 This XML file declares which signature|privileged permissions should be granted to privileged
  20 applications that come with the platform
  21 -->
  22 <permissions>
          ... ... 
  36     <privapp-permissions package="com.android.cellbroadcastreceiver">
  37         <permission name="android.permission.INTERACT_ACROSS_USERS"/>
  38         <permission name="android.permission.MANAGE_USERS"/>
  39         <permission name="android.permission.MODIFY_PHONE_STATE"/>
  40         <permission name="android.permission.READ_PRIVILEGED_PHONE_STATE"/>
  41         <permission name="android.permission.RECEIVE_EMERGENCY_BROADCAST"/>
  42         <permission name="android.permission.MANAGE_ACTIVITY_STACKS"/>
  43         <permission name="android.permission.STATUS_BAR"/>
  44     </privapp-permissions>

         ... ...
452 </permissions>
```


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


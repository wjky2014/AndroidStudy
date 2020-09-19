本文主要包含以下内容
>一、查看项目所在分支
>二、切换到目标分支
>三、查看当前所在分支
>四、编译Android源码
>五、source Android 编译环境
>六、lunch 所需的编译项目
>七、单编 模块
>八、push 模块 验证修改是否生效

#一、查看项目所在分支

`git branch -a`
表示：查看并列出当前项目所有分支

高通项目举例如下：  
```
wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android$ git branch -a
* linux_android_development
  master
  remotes/origin/A/B_update_linux_android_development
  remotes/origin/HEAD -> origin/master
  ... ...
  remotes/origin/secure_linux_android_development
  remotes/origin/streamlined_code_engineering
wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android$ 

```
#二、切换到目标分支

`git checkout 分支名` 
表示： 切换到某个分支。

高通项目举例如下：  
`git checkout linux_android_development `

```
wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android$ git branch -a
* linux_android_development
  master
  remotes/origin/A/B_update_linux_android_development
  remotes/origin/HEAD -> origin/master
  remotes/origin/cts_development_branch
  ... ...
wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android$ git checkout linux_android_development 

```

#三、查看当前所在分支

` git branch `
表示：查看当前所在分支

高通项目举例如下：  

```
wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android$ git branch 
* linux_android_development
  master
wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android$ 

```
#四、编译Android源码
`Android` 源码编译，每个项目由于脚本各不相同，编译的命令 有时候也会有说差异。
#####google 官方编译命令如下
1.source ./build/envsetup.sh 
2.lunch 项目
3.make -j8

#####高通项目编译命令 如下:
`./buildall_userdebug.sh  E5527M all`
#五、source Android 编译环境

首先进入`Android` 源码根目录，执行`source ./build/envsetup.sh `,
如不`source`,后续则无法单编模块。

高通项目举例如下：
```
wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android$ source ./build/envsetup.sh 
including device/generic/car/vendorsetup.sh
including device/generic/mini-emulator-arm64/vendorsetup.sh
including device/generic/mini-emulator-armv7-a-neon/vendorsetup.sh
including device/generic/mini-emulator-x86_64/vendorsetup.sh
including device/generic/mini-emulator-x86/vendorsetup.sh
including device/generic/uml/vendorsetup.sh
including device/google/muskie/vendorsetup.sh
including device/google/taimen/vendorsetup.sh
including device/qcom/common/vendorsetup.sh
including device/qcom/qssi/vendorsetup.sh
including vendor/partner_gms/products/vendorsetup.sh
including vendor/qcom/opensource/core-utils/vendorsetup.sh
including vendor/qcom/proprietary/common/vendorsetup.sh
including vendor/qcom/proprietary/prebuilt_HY11/vendorsetup.sh
Created 9 symlinks out of 9 mapped links..
including sdk/bash_completion/adb.bash
wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android$ 

```
#六、lunch  所需的编译项目

执行 `lunch` 命令，查看所有 编译项目列表，然后选择 编译项目。
高通项目 举例如下：

#####1. lunch ,然后选择所需编译分支

```
wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android$ lunch

You're building on Linux

Lunch menu... pick a combo:
     1. aosp_arm-eng
     2. aosp_arm64-eng
     3. aosp_mips-eng
     ... ...
     40. msm8937_64-userdebug
     41. msm8937_64-user
     ... ...
     65. taimenb2-userdebug
// 选择要编译的分支名
Which would you like? [aosp_arm-eng] msm8937_64-userdebug

device/qcom/msm8937_64/msm8937_64.mk:42: warning: "Build with 4.9 kernel"

 ... ...
OUT_DIR=out
============================================
wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android$ 

```

##### 2. 直接lunch 分支

比如我们需要编译`msm8937_64-userdebug`项目，其实我们可以执行最简单的方法,直接`lunch`这个项目所排列的位置，比如`lunch 40`

```
wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android$ lunch 40
device/qcom/msm8937_64/msm8937_64.mk:42: warning: "Build with 4.9 kernel"

============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=9
TARGET_PRODUCT=msm8937_64
... ...

HOST_CROSS_2ND_ARCH=x86_64
HOST_BUILD_TYPE=release
BUILD_ID=PKQ1.190601.001
OUT_DIR=out
============================================
wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android$ 
```
#七、单编 模块

我们常用 `mmm` 以及`mm` 来及对单模块进行编译。
`mmm` 与`mm` 主要区别在于你当前所在的目录位置。
如果当前正在所要编译模块的地方`（需要有android.mk文件，才可以进行）`，请使用 `mm`，否则使用` mmm`。

高通项目举例如下：

#####1. mmm使用举例（此时不在FM 目录）

比如单编`FM`，不在`FM `目录下，需要执行` mmm vendor/qcom/opensource/commonsys/fm/fmapp2/`
```
wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android$ mmm vendor/qcom/opensource/commonsys/fm/fmapp2/

/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android/vendor/qcom/opensource/commonsys/fm/fmapp2/
Restriction Checker not present, skipping..
device/qcom/msm8937_64/msm8937_64.mk:42: warning: "Build with 4.9 kernel"
============================================
... ...
OUT_DIR=out
============================================
QSSI: not enabled for msm8937_64 target as vendor/qcom/proprietary/release/QSSI/QSSI_enforced_targets_list.txt was not found.

... ...
#### build completed successfully (7 seconds) ####

wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android$ 

```

#####2. mm使用举例(在FM 目录下)


比如单编`FM`，在`FM `目录下，需要执行` mm`

```
wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android$ cd  vendor/qcom/opensource/commonsys/fm/fmapp2/
wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android/vendor/qcom/opensource/commonsys/fm/fmapp2$ mm
Restriction Checker not present, skipping..
device/qcom/msm8937_64/msm8937_64.mk:42: warning: "Build with 4.9 kernel"
============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=9
... ...
OUT_DIR=out
============================================
QSSI: not enabled for msm8937_64 target as vendor/qcom/proprietary/release/QSSI/QSSI_enforced_targets_list.txt was not found.
ninja: no work to do.
... ...
build/make/core/base_rules.mk:412: warning: ignoring old commands for target `out/target/product/msm8937_64/vendor/lib64/hw/fingerprint.default.so'
ninja: no work to do.

#### build completed successfully (41 seconds) ####

wangjie@wangjie:/wangjie/Qualcomm_p/E5527M_MSM8917_QM215_r26/LA.UM.7.6.2/LINUX/android/vendor/qcom/opensource/commonsys/fm/fmapp2$ 

```
#八、 push 模块 验证修改是否生效


##### 1. 使用debug版本，挂载手机

`adb shell getprop ro.build.type` 主要用来查看当前使用的版本，调试只能使用`debug` 版本，`user`版本无法调试。

高通项目举例如下：
```
C:\Users\Administrator>adb shell getprop ro.build.type
userdebug

C:\Users\Administrator>adb root

C:\Users\Administrator>adb remount
remount succeeded

C:\Users\Administrator>
```

##### 2.将生成的单模块编译的apk 拷贝到桌面并push到手机中

`adb push 本地文件  手机目录`
`adb push`主要用来 替换手机中的`apk `，调试验证单编是否生效。

高通 `FM` 举例如下：
```
C:\Users\Administrator>adb push C:\Users\Administrator\Desktop\FM2.apk /system/app/FM2
C:\Users\Administrator\Desktop\FM2.apk: 1 file pushed. 9.7 MB/s (861880 bytes in 0.085s)

C:\Users\Administrator>
```

##### 3. 清除 push apk 的存储数据

`adb shell pm clear 包名` 
用来清除当前包名的数据。
比如：`adb shell pm clear com.caf.fmradio`

高通 `FM apk `举例如下：
```

C:\Users\Administrator>adb shell dumpsys activity | findstr Run
    Running activities (most recent first):
        Run #0: ActivityRecord{3bc8d7d u0 com.caf.fmradio/.FMRadio t52}
    Running activities (most recent first):
        Run #0: ActivityRecord{89df584 u0 com.android.launcher3/com.android.searchlauncher.SearchLauncher t51}
C:\Users\Administrator>adb shell pm clear com.caf.fmradio
Success

C:\Users\Administrator>
```
 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

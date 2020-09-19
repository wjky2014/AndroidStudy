#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、CTS 测试培训内容
>二、GTS 测试培训内容
>三、STS 测试培训内容
>四、GSI 测试培训内容
>五、VTS 测试培训内容
>六、CTS-Verifier 测试培训内容
>七、CDD 检查测试培训内容

# 第一部分：CTS 测试培训内容

##一、CTS 是什么？

Compatibility Test Suite意为兼容性测试，是Google推出的Android平台兼容性测试机制。CTS的目的是让所有的Android设备开发商能够开发出兼容性更好的Android设备（比如：手机）。

##二、CTS 测试的意义

Android的CTS测试，是Google推出的Android平台兼容性测试。这是一套包含了上万个自动运行 测试用例的测试框架 。

CTS测试主要用来测试OEM厂商设计的Android平台是不是符合Android的API接口定义。通过 CTS 测试不仅可以保证Android设备具有良好的用户体验 ，而且可保证大量优质的应用在Android设备上正常运行，同时，也能够让应用开发者能够放心地制作高质量的应用程序，因此只有通过 CTS 认证的设备才能合法的安装使用 Google market 等 Google 的应用。

##三、CTS 测试需要的准备工作

- 1. 测试工具

目前最新`cts(9.0_r9)`/`(8.1_r16)`/`（7.0_r32）`。

![CTS 测试工具](https://upload-images.jianshu.io/upload_images/5851256-fe78653c51314444.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![CTS 测试脚本](https://upload-images.jianshu.io/upload_images/5851256-748f8a2e3a37030a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

也可以通过下面网址去下载最新的测试工具`（需要翻墙）`
`https://source.android.com/compatibility/cts/downloads.html`
- 2. 测试版本 

测试需要使用`user版本`的软件(ROM)
- 3. 准备刷好版本的测试机

32位的一般是2-4台，大概需要三十个小时左右跑完一轮；
64位的项目需要4-6台，一轮测试需要时间是60个小时左右。

- 4. 手机设置

①. 测试需要把`锁屏的开关设置为none`(Settings > Security > Screen Lock should be 'None').

②. 测试需要设置`灭屏时间sleep设置为最长时间或者never`测试需要打开`"USB Debugging"`开关(Settings > Developer options > USB debugging).

![开发者选择Debug 开关](https://upload-images.jianshu.io/upload_images/5851256-9df68aa7effc832b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

③. 打开Settings > Developer options > `Stay Awake`
④. 有GPS的手机需要打开GPS

- 5. 连接一个可以翻墙的WIFI网络
- 6. 预备 Media 媒体包
- 7. 测试环境搭建（PC端）

需要Linux系统电脑、ADB 的环境搭建、修改 adb 的权限等。

![需要配置adb 环境](https://upload-images.jianshu.io/upload_images/5851256-10a0cc07822861fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![修改adb 权限](https://upload-images.jianshu.io/upload_images/5851256-88de0d891ec686e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##  四、CTS 测试命令

- 1.CTS 测试常用测试命令

测试项|测试命令
----|----
1.全测命令|run cts 
2.3台一起运行命令|run cts --shard-count 3
3.单测一项| run cts –m <模块名> -t <单项名称>
4.继续测试|run  retry --retry  <session  id> （session     id是l r查看结果，最前面的数字序号）；
5.单测一个模块| run cts –m <测试模块名>

- 2.CTS 3台手机一起测试命令

![CTS 3台手机一起测试命令举例](https://upload-images.jianshu.io/upload_images/5851256-3d00047eabd645c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##  五、CTS 测试报告解读


CTS  测试结果会自动生成到`android-cts/results`中，会存储到日期加时间的文件夹中，其中的`test_result_failures.xml`可以用浏览器打开，查看详细的测试结果，如图所示
![CTS 报告存储路径](https://upload-images.jianshu.io/upload_images/5851256-74988e8e8927b2b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![CTS 报告头](https://upload-images.jianshu.io/upload_images/5851256-b9cd8a99f9fd2510.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##  六 、CTS测试套件文档目录结构

- 1. CTS测试套件文档目录结构如下：

![CTS测试套件文档目录结构](https://upload-images.jianshu.io/upload_images/5851256-fafc908088e12baf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5851256-6b72d3a361f4810b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![device-info-files 测试举例](https://upload-images.jianshu.io/upload_images/5851256-bc1a6291b13a2e1b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 2. 部分目录解释如下：

名称|作用
----|----
1.logs|测试过程中抓取得log
2.results |测试结果
3.device-info-files|测试设备的信息
4.tools |测试工具

##七、CTS 测试过程中需要注意的地方

1. 测试过程中需要SIM（联通）卡，白卡
2. 测试GPS需要把手机放置到外边露天的地方测试
3. 测试链接IPV6网络
4. 测试国外的数据流量（香港卡）------针对9.0的项目
5. 需插入T卡，设置成内部存储

# 第二部分：GTS 测试培训内容

## 一、GTS 测试是什么

Google Mobile Services Test Suite，所谓的Google Mobile Services即谷歌移动服务
是谷歌开发并推动Android的动力，也是Android系统的灵魂所在

## 二、GTS 测试的意义

谷歌移动服务测试套件（Google Mobile Services Test Suite），谷歌移动服务提供了Search、 Search by Voice、Gmail、Contact Sync、 Calendar Sync、Talk、 Maps、 Steet View、 YouTube、 Android Market (Play store)等服务，当用户使用谷歌时，谷歌可以把各种广告嵌入到谷歌的服务中。

## 三、GTS 测试需要的准备工作

1. 最新工具：7.0-r2
2. GTS的环境安装、执行、报告分析这一系列操作和CTS类似，只是GTS必须连接VPN。
3. user版本
4. 设置手机、连接外网
5. 打开蓝牙
6. 显示中睡眠时间显示最大
7. 语言选择英语
8. 键盘选择谷歌键盘
9. 打开定位、高精度
10. 安全中锁屏选择None
11. 在开发者模式中打开stay awake和USB debugging

## 四、GTS 测试命令

- 1. GTS 测试命令 如下：

测试项| 测试命令
---- |----
1.全测命令|run gts 
2.3台一起运行命令|run gts --shard-count 3
3.单测一项| run gts –m <模块名> -t <单项名称>
4.继续测试|run  retry --retry  <session  id> （session   id是l r查看结果，最前面的数字序号） 
5.特殊命令|run gts -m GtsMediaTestCases --test com.google.android.media.gts.MediaDrmTest#testUsageTableCapacity --module-arg "GtsMediaTestCases:test-timeout:3600000"

- 2. 单独测试某项举例：
`run gts -m GtsSettingsTestCases  -t  com.google.android.settings.gts.MADAComplianceTest#testMADACompliance`

![对应的fail 项](https://upload-images.jianshu.io/upload_images/5851256-31d95981bad38d20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 五、GTS 测试报告解读

测试结果存放地址：
![测试结果存放地址](https://upload-images.jianshu.io/upload_images/5851256-b596b3b33ee82a15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5851256-1356111a30996749.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 六、GTS测试过程中需要注意的地方

1. GTS测试9.0的项目需要插白卡
2. 测试GPS
3. 截屏信息

![截屏信息举例](https://upload-images.jianshu.io/upload_images/5851256-61b11f8b76b8b4fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 第三部分：STS测试培训内容

## 一、STS 是什么？

主要是测试 Android Security Patch 是否生效，Goolge最新通告，2019年03月开始STS要全部pass。

## 二、STS 测试意义

Android Security Test Suite (STS). 是谷歌关于android安全补丁安装情况的一个测试套件STS和security patch相关的，是CTS测试新增加一项安全测试套件。STS是201808才开始测试的。Security patch日期在3个月内是GTS的一个case，如果不通过无法获得google认证。在18年5月之前谷歌对于security patch这个属性都是在build库下面跟着aosp更新的。现在需要通过STS之后由vendor来更新。STS需要通过userdebug版本的targetfile来生成一个user版本的结果

## 三、STS 测试准备工作

- 1. 测试工具要和系统的安全patch相一致

![Google STS 释放版本列表](https://upload-images.jianshu.io/upload_images/5851256-6d50251a27ae5b4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 2. 测试版本 
测试需要使用userdebug版本。

![不同平台需要不同的测试工具](https://upload-images.jianshu.io/upload_images/5851256-2a0a9376e02e8b21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Android 8.1 举例](https://upload-images.jianshu.io/upload_images/5851256-37913d10beb2a89d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 3. 准备刷好版本的测试机一台
32位机子测试时间6个小时左右，64位测试时长8个小时左右

- 4. 手机设置
测试需要把`锁屏的开关设置为none`(Settings > Security > Screen Lock should be 'None').
测试需要设置`灭屏时间sleep设置为最长时间或者never`
- 5.链接翻墙 WIFI 

##四、STS 测试命令

- 1. STS 测试命令如下：

测试项|测试命令
----|----
1.全测命令|run sts-engbuild
2.单测一项| run sts-engbuild –m <模块名> -t <单项名称>
3.继续测试|run  sts-engbuild --retry  <session  id> （session   id是l r查看结果，最前面的数字序号） 

- 2. STS 全测举例如下：
![STS 全测举例](https://upload-images.jianshu.io/upload_images/5851256-df13da5c86cfad1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 五、STS 测试报告解读

- 1. STS 测试报告目录

STS 测试报告目录与用途如下：

目录|用途
----|----
logs| 测试过程中抓取得log
results |测试结果
tools |测试工具

![STS报告目录](https://upload-images.jianshu.io/upload_images/5851256-1c573f61ef3754d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![STS 报告头](https://upload-images.jianshu.io/upload_images/5851256-012899757b4935a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 六、STS 测试过程中需要注意的地方

  - 1. Security patch日期在3个月内

这是GTS的一个case，如果不通过无法获得google认证。
例子 ：
```
GtsOsTestCases–com.google.android.os.gts.SecurityPatchTest#testSecurityPatchDate
```
否则会有明确报错，需要更新安全patch日期：
```
junit.framework.AssertionFailedError: ro.build.version.security_patch should be “2018-08” or later. Found “xxxx-xx-xx”
```

- 2. fingerprint和user版本要求一致


# 第四部分：GSI 测试培训内容

## 一、GSI 是什么 
（Generic system image）— Reference AOSP system image通用系统映像上的兼容性测试套件
这个文件包也是签约获取授权后才能获取，Google也会定期更新GSI包Android O要求测试VTS和CTS on GSI，此时对应版本必须是GSI版本，测试包都为VTS。

## 二、GSI 测试的准备工作

- 1. vts测试工具 

9.0-r11   8.1_r17

- 2. Media 媒体包

要求 CTS Media 1.4 及以上版本，在Android8.1的测试中，media文件要放在电脑中的/tmp/android-cts-media路径下，测试前不需要将Media文件拷贝到手机中，测试时会自动拷贝。如果/tmp/android-cts-media路径下没有media文件，将会从网上下载，由于文件比较大，比较耗时

- 3. 替换谷歌system.img

VTS测试要求刷入谷歌提供AOSP的system.img (GSI)。在user版本中，如果直接使用flash tool单独烧录GSI时，会导致无法开机。之所以会出现这种问题，是由于在user/userdebug版本中，dm-verity是使能的，替换GSI后导致dm-verity不能通过。 如果要解决这个问题，就`需要进行unlock操作`，并且要用fastboot来刷入刷入谷歌提供的system.img .

- 4. MTK 项目 替换system.img 操作方法

①在设置中打开 OEM unlocking 选项
②在设置中打开 USB debugging 选项
③长按音量 + 和电源键进入fastboot模式
或者执行
```
adb reboot bootloader
```
④连接到电脑上，分别执行`fastboot flashing unlock`和`fastboot oem unlock`。
⑤执行命令后需要选择音量 + 来确认unlock。
⑥执行 fastboot 命令刷入google提供的 system.img (GSI)
`fastboot flash system system.img`（需要根据软件版本的信息来选择GSI版本）
(Android P版本需要执行) `fastboot flash vbmeta vbmeta.img`
⑦重启：` fastboot reboot`

- 5. 展讯项目替换system.img 操作方法

①展讯项目项目具体替换system.img操作如下：

刷版本过程中将system文件替换为system.img文件，正常刷机
(Android P版本需要执行) VB文件替换为最新的 vbmeta.img

②展讯平台Research DownLoad 单独替换方法：
![展讯平台Research DownLoad 单独替换方法](https://upload-images.jianshu.io/upload_images/5851256-b9ef7cb281ca45b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 6. vts测试环境配置

安装 Python 开发工具包：
```
sudo apt-get install python-dev
```
安装协议缓冲区工具（适用于 Python）：
```
sudo apt-get install python-protobuf protobuf-compiler
```
安装 Python 虚拟环境相关工具：
```
sudo apt-get install python-virtualenv python-pip
```

- 7. Media 媒体包

要求 CTS Media 1.4 及以上版本，在Android8.1的测试中，media文件要放在电脑中的`/tmp/android-cts-media`路径下，测试前不需要将Media文件拷贝到手机中，测试时会自动拷贝。如果`/tmp/android-cts-media`路径下没有media文件，将会从网上下载，由于文件比较大，比较耗时

- 8. 替换谷歌system.img

VTS测试要求刷入谷歌提供`AOSP的system.img (GSI)`。在user版本中，如果直接使用`flash tool`单独烧录GSI时，会导致无法开机。之所以会出现这种问题，是由于在`user/userdebug`版本中，`dm-verity`是使能的，替换GSI后导致dm-verity不能通过。 如果要解决这个问题，就需要进行unlock操作，并且要用fastboot来刷入刷入谷歌提供的system.img .

## 三、GSI 测试命令

- 1. GSI 测试命令如下表格：

测试项|测试命令
----|----
1.全测命令|run cts-on-gsi -s+SN
2.3台一起运行命令|run  cts-on-gsi  --shard-count 3
3.单测一项| run  cts-on-gsi  –m <模块名> -t <单项名称>
4.继续测试|run   cts-on-gsi-retry  --retry  <session  id> 
5.单测一个模块| run cts-on-gsi  –m <测试模块名>

- 2. 2台手机一起测试方法举例

![2台手机一起测试方法举例](https://upload-images.jianshu.io/upload_images/5851256-d8c3744ed3976644.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 四、GSI 测试过程中需要注意的事项

- 1.cts-on-gsi默认只测64位，但是如果后面加上-a armeabi-v7a的话，也能进行32位的测试；
- 2. 时区设置问题，请把手机时区设置为上海，或者北京时区



# 第四部分：VTS测试培训内容

## 一、VTS 是什么

VTS的全称是 Vendor Test Suite（供应商测试套件）

## 二、VTS 的测试意义

- 1. 以前Android的系统升级是很麻烦的，为了能更快的将设备升级到新的Android版本，Android O 开始新引入了 Project Treble，Project Treble 适用于搭载 Android O 及后续版本的所有新设备。

 - 2. Android 7.x 及更早版本中没有正式的Vendor层接口，因此每次更新系统都相对耗时和困难：

- 3. Android O 之后，Treble 提供了稳定的Vendor层接口，供设备制造商访问 Android 代码中特定于硬件的部分，这样就可以只更新框架层，减少升级系统带来的成本和困难：

- 4. 为了确保Vendor层实现的前向兼容性，新的Vendor层接口会由供应商测试套件 (VTS) 进行验证，该套件类似于兼容性测试套件 (CTS)。

##三、VTS 测试前的准备工作


替换谷歌system.img和gsi准备工作一样，替换img文件之前需要确保手机解锁，展讯的项目解锁如下：
使用unbuntu系统解锁
解锁前请先进入fastbootbootloader状态,指令如下:

```
1. adb reboot bootloader
2. cd 到当前文件
3. ./fastboot oem get_identifier_token（获取设备号）
```
![](https://upload-images.jianshu.io/upload_images/5851256-21fdbee75718e03d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5851256-1e12d04a16fe7f26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. `sudo ./signidentifier_unlockbootloader.sh 
53433938363331413130313833323330313437 rsa4096_vbmeta.pem signature.bin 
rsa4096_vbmeta.pem `密匙文件请自行到对应软件中获取

5. `sudo ./fastbootflashing unlock_bootloader signature.bin `（用户根据相
应提示使用“volume down”键进行解锁确认
 
![](https://upload-images.jianshu.io/upload_images/5851256-762b7b8f17dd2b39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 四、VTS 测试命令

- 1. VTS 测试命令如下：

测试项|测试命令
----|----
1.全测命令|run vts 
2.单测一项| run  vts –m <模块名> -t <单项名称>
3.继续测试|run   vts --retry  <session  id> 
4.单测一个模块| run  vts –m <测试模块名>

## 五、VTS 测试过程中需要注意的地方

- 针对9.0项目，vts测试过程中需要插SIM卡和白卡
- 测试过程中会测试fingerprint，要选择指纹锁正常的手机来测试
- 会测试到GPS


![](https://upload-images.jianshu.io/upload_images/5851256-effbd320e8df4723.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 第六部分：CTS-Verifier 测试培训内容

## 一、CTS-Verifier 是什么

CTS Verifier算是CTS的一部分，需要手动进行，主要用于测试那些自动测试系统无法测试的功能，比如相机、传感器等。由于硬件配置或其他原因，不同手机上部分测试项目被隐藏，也就是说CTS Verifier中case的总数，取决于测试机支持哪些功能

##  二、CTS-Verifier 测试前的准备工作

1. 把测试机刷成需要测试的版本。
2. 安装*/android-cts-verifier/CtsVerifier.apk。
3. 设置手机语言为English。
4. 打开蓝牙，无需配对。
5. 打开并连接可用wifi。
6. 请再另外准备一台手机，以便测server和client相关的case。

![CTS-Verifier apk](https://upload-images.jianshu.io/upload_images/5851256-9527471709b1e5b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 三、CTS-Verifier 测试的主内容

- 1.CTS-Verifier 测试的主内容 如下：

· Camera
· Clock
· Device administration
· Features
· Hardware
· Job scheduler
· Location
· Managed Provisioning
· Networking
· Notifications
· Others
· Projection tests
· Security
· Sensors
· Streaming

## 四、CTS-Verifier 需要注意的地方

- 1. 每项测试均包含如下三个 Button（有些测试项 Pass/Fail 是自动进行判断的）

手动测试选项|意义
----|----
Pass | 如果待测设备满足 Info 中说明的测试要求，单击 Pass 按钮
Info | 对该项测试方法的说明，它将出现在第一次进入该测试项或者每次单击 Info 按钮的时候
Fail |如果待测设备不满足 Info 中说明的测试要求，单击 Fail 按钮

- 2. 手动测试选项 举例

![手动测试选项 举例](https://upload-images.jianshu.io/upload_images/5851256-b5c59287d505565d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 第七部分：CDD检查测试培训内容

## 一、CDD检查是什么？

CDD checklist全称Compatibility Definition Document 安卓兼容性定义文档。

## 二、CDD 检查的意义

CDD列举兼容的Android设备的软件和硬件要求。
因为Android源代码的开放性，众多手机厂商都在Android源代码的基础上添加了自己的定制，可能包括从Linux kernel层到上面的framework层都有修改，为了打造一个共同的生态系统，让不同的APP开发者开发出来的APP在所有的Android设备上都能正常运行，保证用户的使用体验，Google就提出了一些限制，这些限制就是CDD文档。

##三、CDD 检查的内容

- 1.首先确认自己做的项目是哪个系统
9.0 ,8.1 还是 7.0 ；然后确认是不是Go的项目，对Go的区分是以1g的RAM来要求的，大于1G的 都属于非Go；最后一定要知道这个项目是做哪种版本：init(初次送测) ，MR版本 ， Smr版本 ，每个版本都有自己不一样的检查cdd的方式,输入`adb shell pm list features `查看什么类型的项目。 

- 2. 参与Google Funding项目应该为：`com.google.android.feature.GMSEXPRESS_PLUS_BUILD`
   具体测试标准参考mtk文档，必须按照文档检查下去。

- 3. 不参与Funding普通项目之前设置的为：`com.google.android.feature.GMSEXPRESS_BUILD`,以后建议取消

- 4. 俄罗斯项目应为：`com.google.android.feature.RU `测试标准也有文档，主要集中于测试Seacher  

- 5. 出欧洲项目应为：  

```
    <feature name="com.google.android.feature.EEA_DEVICE" />
      <feature name="com.google.android.paid.chrome" />
      <feature name="com.google.android.paid.search" /> 
```
- 6.查看 API Level

使用以下命令`adb shell "getprop | grep first_api_level"`可以查看API level。  

Google 对不同版本的 API Level 要求如下：

Android 版本| API level 要求
----|----
对于9.0项目 |不管Init还是MR版本 都需要设置为28 
对于8.1项目| Init项目需要设置为空；MR版本需要设置为27
对于7.0的项目 | Init项目需要设置为空；MR版本需要设置为24
对于6.0项目 |  Init项目需要设置为空；MR版本需要设置为23

- 7.Check Default Home Screen

Duo必须放在default home screen（适应于非Go项目）
Android Messages和Chrome必须放在Hotseat。

![适应于非Go项目](https://upload-images.jianshu.io/upload_images/5851256-f750ab305cb8f699.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 8.Check Google Folder 非Go项目


打开google文件夹按照“ 从左到右，从上到下” 的顺序，摆放core app (如右图）
分别有`Google Search、Gmail、Maps、YouTube、Drive、Play Music（11月以后的新项目替换YouTu Music  老项目延续之前所用）、Play Movies、Photos`。

![Google Folder 举例](https://upload-images.jianshu.io/upload_images/5851256-24c3186e4c2ac477.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 9. Check “ro.base_build”  for Android P Devices

对于Android P GMS Express+设备 (不管是新版本还是维护版本), 请确保有设置该属性 `“ro.base_build = noah”` .

可用如下命令检查:
```
adb shell “getprop | grep ro.base_build”
```

 MR版本 需要去掉base_os值 每个项目都需要查一下。

![](https://upload-images.jianshu.io/upload_images/5851256-75be860e884ba2e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5851256-9abeb2295fa02259.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 10.检查Android P上的DNS 隐私模式设置

搭载 Android 9 或更高版本的 GMS 设备必须支持DNS over TLS ，并且必须提供UI界面，以便用户从 AOSP 中定义的关闭、自动或私人 DNS 提供商主机名之一中选择 DNS 隐私模式设置。
当设备在开箱后进行设置时，其默认的 DNS 隐私模式设置应为自动。

![](https://upload-images.jianshu.io/upload_images/5851256-77a96c1cd6a68f8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

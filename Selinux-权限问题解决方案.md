##1. 概述

SELinux是Google从android 5.0开始，强制引入的一套非常严格的权限管理机制，主要用于增强系统的安全性。

然而，在开发中，我们经常会遇到由于SELinux造成的各种权限不足，即使拥有“万能的root权限”，也不能获取全部的权限。本文旨在结合具体案例，讲解如何根据log来快速解决90%的SELinux权限问题。

## 2. 调试确认SELinux问题
为了澄清是否因为SELinux导致的问题，临时禁用selinux ，重启失效，可先执行：
```

C:\Users\Administrator>adb shell getenforce
Enforcing
//临时禁用selinux ，重启失效
C:\Users\Administrator>adb shell setenforce 0
C:\Users\Administrator>adb shell getenforce
Permissive
C:\Users\Administrator>
```
如果问题消失了，基本可以确认是SELinux造成的权限问题，需要通过正规的方式来解决权限问题。

遇到权限问题，在logcat或者kernel的log中一定会打印avc denied提示缺少什么权限，可以通过命令过滤出所有的avc denied，再根据这些log各个击破：
```
cat /proc/kmsg | grep avc 
```
或
```
dmesg | grep avc
```
例如：
```
audit(0.0:67): avc: denied { write } for path="/dev/block/vold/93:96" dev="tmpfs" ino=1263 scontext=u:r:kernel:s0 tcontext=u:object_r:block_device:s0 tclass=blk_file permissive=0
```
可以看到有avc denied，且最后有permissive=0，表示不允许。

##3. 具体案例分析

解决原则是：缺什么权限补什么，一步一步补到没有avc denied为止。

解决权限问题需要修改的权限文件如下位置，以.te结尾
```
A：Android/devicesoftwinner/astar-common/sepolicy/*.te

B：Android/external/sepolicy/*.te
```
其中，A是对B的overlay（覆盖），能在A修改的尽量在A修改，尽量避免修改B，修改B可能会导致CTS fail问题，修改A不会影响CTS测试。

（如果不需要深入了解，请直接跳到万能公式这一章阅读更简洁）

下面给出四个案例：

### 案例1
```
audit(0.0:67): avc: denied { write } for path="/dev/block/vold/93:96" dev="tmpfs" ino=/1263 scontext=u:r:kernel:s0 tcontext=u:object_r:block_device:s0 tclass=blk_file permissive=0
```
 

分析过程：

缺少什么权限：      { write }权限，

谁缺少权限：        scontext=u:r:kernel:s0

对哪个文件缺少权限：tcontext=u:object_r:block_device

什么类型的文件：    tclass=blk_file

完整的意思： kernel进程对block_device类型的blk_file缺少write权限。

 

解决方法：在上文A位置，找到kernel.te这个文件，加入以下内容：
```
allow  kernel  block_device:blk_file  write;
```
`make installclean && make bootimage `
后重新编译 ，刷boot.img才会生效。

 

### 案例2
```
audit(0.0:53): avc: denied { execute } for  path="/data/data/com.mofing/qt-reserved-files/plugins/platforms/libgnustl_shared.so" dev="nandl" ino=115502 scontext=u:r:platform_app:s0 tcontext=u:object_r:app_data_file:s0 tclass=file permissive=0
```
 

分析过程：

缺少什么权限：      { execute}权限，

谁缺少权限：        scontext = u:r:platform_app:s0

对哪个文件缺少权限：tcontext = u:object_r:app_data_file

什么类型的文件：    tclass= file

完整的意思： platform_app进程对app_data_file类型的file缺少execute权限。

 

解决方法：在上文A位置，找到platform_app.te这个文件，加入以下内容：
```
allow  platform_app  app_data_file:file  execute;
```
make installclean后重新编译，刷boot.img才会生效。

 

### 案例3
```
audit(1444651438.800:8): avc: denied { search } for pid=158 comm="setmacaddr" name="/" dev="nandi" ino=1 scontext=u:r:engsetmacaddr:s0 tcontext=u:object_r:vfat:s0 tclass=dir permissive=0
```
解决方法 ：engsetmacaddr.te
```
allow  engsetmacaddr  vfat:dir  { search write add_name create }; 或者

allow  engsetmacaddr   vfat:dir  create_dir_perms;

(create_dir_perms包含search write add_name create可参考external/sepolicy/global_macros的定义声明)

 ```

### 案例4
```
audit(1441759284.810:5): avc: denied { read } for pid=1494 comm="sdcard" name="0" dev="nandk" ino=245281 scontext=u:r:sdcardd:s0 tcontext=u:object_r:system_data_file:s0 tclass=dir permissive=0
```
解决方法 ：sdcardd.te 

```
allow  sdcardd  system_data_file:dir  read;  或者
allow  sdcardd  system_data_file:dir  rw_dir_perms;

 (rw_dir_perms包含read write，可参考external/sepolicy/global_macros的定义声明)

 ```

 

## 4. 万能公式
通过这四个案例，我们可以总结出一般规律,

以第案例4为例：
```
audit(1441759284.810:5): avc: denied { read } for pid=1494 comm="sdcard" name="0" dev="nandk" ino=245281 scontext=u:r:sdcardd:s0 tcontext=u:object_r:system_data_file:s0 tclass=dir permissive=0
```
 

某个scontext对某个tclass类型的tcontext缺乏某个权限，我们需要允许这个权限：

我们的log重新排列一下，
```
scontext = u:r:sdcardd

tcontex t= u:object_r:system_data_file:s0

tclass = dir

avc: denied { read }

 ```

得到万能套用公式如下：

在scontext所指的.te文件（例如sdcardd.te）中加入类似如下allowe内容：

 

## 5. TIPS
1. 以上以.te为后缀的文件都在以下位置：
```
A：Android/devicesoftwinner/astar-common/sepolicy/*.te

B：Android/external/sepolicy/*.te
```
其中，A是对B的overlay（覆盖），能在A修改的尽量在A修改，修改B可能会导致CTS fail问题，修改A不会影响CTS测试。修改之后，为了节约验证时间，只重刷boot.img即可看效果；

 

2. 有时候avc denied的log不是一次性暴露所有权限问题，要等解决一个权限问题之后，才会暴露另外一个权限问题。比如提示缺少某个目录的read权限，加入read之后，才显示缺少write权限，要一次次一次试，一次一次加，时间成本极大。
针对dir缺少的任何权限，建议赋予create_dir_perms，基本涵盖对dir的所有权限，比如：
{ open search write read rename create rmdir getattr }等等。
针对file缺少的任何权限，建议赋予rwx_file_perms，基本涵盖对file的所有权限，比如：
包含{ open read write open execute getattr create ioctl }等等。

更多内容请参考external/sepolicy/global_macros来了解更多权限声明。

3. 要加入的权限很多时，可以用中括号，比如：
```
allow engsetmacaddr  vfat:dir { search write add_name create};
```
4. 修改A位置的.te文件遇到编译错误怎么办？
（首先请排除拼写错误）说明此项权限是SELinux明确禁止的，也是Google CTS禁止的，如果产品不需要过CTS，可以修改。一般来说，编译出错的log会提示相关哪个文件哪一行出错，文件位置一定会在B里的.te文件。比如B规定了以下neverallow,
neverallow system_server sdcard_type:dir { open read write };
那么system_server是不能拥有这些权限的，如果赋予这些权限就编译报错，解决方法是根据编译错误提示的行号，把这一句注释掉即可。

 

 

## 6. 高级进阶
### 6.1. 新建.te安全策略文件方法
以上基本是对已经存在的进程增加权限，但对第三方进程改如何新增一个全新的te文件并赋予权限呢？

以写mac地址的setmacaddr执行文件为例（这个执行档android原生不存在，自行添加的）：

在init.xxx.rc中如下服务：
```
service engsetmacaddr  /system/bin/setmacaddr  /data/misc/wifi/wifimac.txt

    class main

    disabled

oneshot
```
 

 

1. 在device/softwinner/astar-common/sepolicy/file_contexts中，参考其他进程声明一个scontext：

```

/system/bin/install-recovery.sh u:object_r:install_recovery_exec:s0

/system/bin/dex2oat     u:object_r:dex2oat_exec:s0

/system/bin/patchoat    u:object_r:dex2oat_exec:s0

/system/bin/setmacaddr u:object_r:engsetmacaddr_exec:s0
```
指定setmacaddr的路径，并指定一个名字，一定要以service名+_exec结尾

 

2.参考其.te文件在device/softwinner/astar-common/sepolicy/file_contexts 创建engsetmacaddr.te文件，内容如下：
```
type engsetmacaddr, domain;

type engsetmacaddr_exec, exec_type, file_type;

init_daemon_domain(engsetmacaddr)


allow engsetmacaddr  vfat:dir { search write add_name create};
allow engsetmacaddr  vfat:file { create read write open };
allow engsetmacaddr  engsetmacaddr:capability dac_override;
allow engsetmacaddr  shell_exec:file { execute read open execute_no_trans};
allow engsetmacaddr  system_data_file:dir { write add_name remove_name };
allow engsetmacaddr  system_data_file:file { create execute_no_trans write open setattr};

allow engsetmacaddr  system_file:file { execute_no_trans};
```
以上赋予的权限全部是根据avc denied的log缺什么一步一步补什么来的。

 

###6.2. 新设备节点增加访问权限
驱动创建了一个新的设备节点，即使权限是777，android层也是没有访问权限的。

下面以一个/dev/wifi_bt节点为示范，让此节点被用户空间的system_server进程访问。

1. 编辑devicesoftwinner/astar-common/sepolicy/device.te，仿照这个文件里的写法，定义一个dev_type类型的wifi_bt_device设备：
```
type misc_block_device, dev_type;

type private_block_device, dev_type;

……

type wf_bt_device, dev_type;  

 ```

2. 编辑file_contexts.te，将/dev/wf_bt节点声明为第1步定义的wf_bt_device:
```
/dev/block/by-name/misc         u:object_r:misc_block_device:s0

/dev/block/by-name/alog         u:object_r:log_block_device:s0

/dev/block/by-name/private      u:object_r:private_block_device:s0

# We add here  

/dev/wf_bt              u:object_r:wf_bt_device:s0  

 ```

3. 在system_server.te，根据dmesg | grep avc允许system_server对wf_bt_device这个节点可读可写：
```
# Read/Write to /proc/net/xt_qtaguid/ctrl and and /dev/xt_qtaguid.  

allow system_server qtaguid_proc:file rw_file_perms;  

allow system_server qtaguid_device:chr_file rw_file_perms;  

 ……

allow system_server wf_bt_device:chr_file rw_file_perms;  
```
其他进程如需访问/dev/wf_bt节点，依样画葫芦，增加对wf_bt_device的权限即可。

作者：缥缈孤鸿影_love
原文链接：https://blog.csdn.net/tung214/java/article/details/72734086


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

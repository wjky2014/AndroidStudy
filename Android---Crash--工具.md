
 
##### 和你一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、Crash 简介
>二、搭建Crash 分析kernel ramdump平台
>三、Crash 命令简介
>四、使用 Crash 分析 sysdump log
>五、Crash 常规调试




# 一、Crash  简介

   当`Linux`系统内核发生崩溃的时候，可以通过 **KEXEC+KDUMP** 等方式收集内核崩溃之前的内存，生成一个转储文件`vmcore`。内核开发者通过分析该`vmcore`文件就可以诊断出内核崩溃的原因，从而进行操作系统的代码改进。那么`Crash`就是一个被广泛使用的内核崩溃转储文件分析工具.

对调试来讲，`gdb`是非常适合的，但`gdb`始终是调试`native`的工具，不支持`kernel`信息显示，比如`task`信息之类的。`crash`补足了这个短板，由`Dave Anderson`开发和维护的一个内存转储分析工具，是基于`GDB`开发的 (`GDB`适用于用户进程的`coredump`，而`Crash`扩展了 `GDB`，使其适用于 `linux kernel coredump`)，目前它的最新版本是7.2.3。

在没有统一标准的内存转储文件的格式的情况下，Crash工具支持众多的内存转储文件格式，包括：

- Live linux系统
- kdump产生的正常的和压缩的内存转储文件
- 由makedumpfile命令生成的压缩的内存转储文件
- 由Netdump生成的内存转储文件
- 由Diskdump生成的内存转储文件
- 由Kdump生成的Xen的内存转储文件
- IBM的390/390x的内存转储文件
- LKCD生成的内存转储文件
- Mcore生成的内存转储文件

而MTK在KE时会抓取full dump文件：SYS_COREDUMP，则可以用crash来调试。

#二、 搭建Crash 分析kernel ramdump平台

- 1. Crash 官方下载源码

[Crash 官网 源码下载地址 ](http://people.redhat.com/anderson/)

- 2.  编译前确保必要的组件（ncurese和zlib），如果没有需要：
```
sudo apt-get install libncurses5-dev
sudo apt-get install zlib1g-dev
```
- 3. 编译 ARM32 / ARM64 位的Crash

1.ARM32
```
    cd crash-7.1.0
    make target=ARM
```
2.ARM64 
```
    cd crash-7.1.0
    make target=ARM64
```
3.去除编译生成的Crash 中的多余符号
```
strip -s crash
```
4. 使用对应的vmLinux解析sysdump文件
当发生kernel crash时，会有db生成，用GAT的logviewer解开db，里面有SYS_COREDUMP，结合对应的vmlinux（必须是烧录前备份的vmlinux！）
```
./crash vmlinux syscoredump(log)
```
# 三、Crash 命令简介

[Crash 命令官方简介](http://people.redhat.com/anderson/crash_whitepaper/)
至此，本篇已结束，如有不对的地方，欢迎您的建议与指正。期待您的关注，
感谢您的阅读，谢谢！

# 四、使用 Crash 分析 sysdump log

- 1. 将 **vmlinux**  、**crash_arm**、  **sysdump log** 放置同一目录

- ` cp out/target/product/sp9832e_1h10_go/obj/KERNEL/vmlinux  reboot/`
- ` cp vendor/sprd/tools/crash/crash_arm reboot/`

![将 vmlinux  crash_arm  sysdump log 放置同一目录](https://upload-images.jianshu.io/upload_images/5851256-765865d1ddb38a92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 2. 将 **sysdump** 所有文件 追加到一个文件中

![将sysdump 所有文件 追加到一个文件中](https://upload-images.jianshu.io/upload_images/5851256-a836f8a3349dcf2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 3. 使用 **crash_arm** 脚本 联合 **vmlinux** 解析 **sysdump log**

如果解析失败，可以参数带一下参数的命令
32位系统使用如下：
`./crash_arm vmlinux all  -m phys_base=0x80000000`
64位系统使用如下命令：
`./crash_arm64  vmlinux  all  -m phys_offset=0x80000000`


![使用 crash_arm 脚本 联合 vmlinux 解析sysdump log](https://upload-images.jianshu.io/upload_images/5851256-f1556887925467c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 4. 使用 **Log** 命令 将 `Crash log`追加到指定文件中

![使用 log 命令  读取log 到指定文件 ](https://upload-images.jianshu.io/upload_images/5851256-fca2b8a010cc36c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 5. 查看`log`，分析重启的具体原因

![重启log举例 ](https://upload-images.jianshu.io/upload_images/5851256-3d5ad8270e397faf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 6. 调高Kernel log Buffer

请将如下两处修改为=21，增大kernel log buffer后，先抓一份 “进开机向导并能正常启动至idle” 的ylog。

```
/kernel/sprd-diffconfig/pike2/user_diff_config
VAL:CONFIG_LOG_BUF_SHIFT=16
/kernel/arch/arm/configs/sprd_pike2_defconfig
CONFIG_LOG_BUF_SHIFT=17
```
# 五、Crash 常规调试

crash使用gdb作为它的内部引擎，crash中的很多命令和语法都与gdb相同。如果曾经使用过gdb，就会发现crash并不是很陌生。如果想获得crash更多的命令和相关命令的详细说明，可以使用crash的内部命令help来获取：

命令	|说明|	例子
----|----|----
*	|指针的快捷方式，用于代替struct/union	|*page 0xc02943c0：显示0xc02943c0地址的page结构体
files	| 显示已打开的所有文件的信息	| files 462：显示进程462的已打开文件信息
mach|	 显示与机器相关的参数信息	| mach：显示CPU型号，核数，内存大小等
sys	| 显示特殊系统的数据	 s|ys config：显示CONFIG_xxx配置宏状态
timer	 |无参数。按时间的先后顺序显示定时器队列的数据	| timer：显示详细信息
mod|	 显示已加载module的详细信息|	 mod：列出所有已加载module信息
runq|	 显示runqueue信息	| runq：显示所有runqueue里的task
tree	 |显示基数树/红黑树结构	| tree -t rbtree -o vmap_area.rb_node vmap_area_root：显示所有红黑树vmap_area.rb_node节点地址
fuser|	 显示哪些task使用了指定的文件/socket	 |fuser /usr/lib/libkfm.so.2.0.0：显示使用了该文件的所有进程
mount|	显示已挂载的文件系统信息	|mount：当前已挂载的文件系统信息
ipcs	|显示System V IPC信息|	ipcs：显示系统中System V IPC信息
ps|	显示进程状态	|ps：类似ps命令
struct|	显示结构体的具体内容	|struct vm_area_struct c1e44f10：显示c1e44f10结构
union|	显示联合体的具体内容，用法与struct一致	|union bdflush_param：显示bdflush_param结构
waitq|	列出在等待队列中的所有task。参数可以指定队列的名称、内存地址等|	waitq buffer_wait：显示buffer_wait等待队列信息
irq|	显示中断编号的所有信息|	irq 18：显示中断18的信息
list|	显示链表的内容	|list task_struct.p_pptr c169a000：显示c169a000地址所指task里p_pptr链表
log|	显示内核的日志，以时间的先后顺序排列	|log -m：显示kernel log
dev	|显示数据关联着的块设备分配，包括端口使用、内存使用及PCI设备数据	|dev：显示字符/块设备相关信息
sig|	显示一个或者多个task的signal-handling数据	|sig 8970：显示进程8970的信号处理相关信息
task|	显示指定内容或者进程的task_struct的内容|	task -x：显示当前进程task_struct等内容
swap|	无参数。显示已配置好的交换设备信息	|swap：交换设备信息
search|	在给定范围的用户、内核虚拟内存或者物理内存搜索值	|search -u deadbeef：在用户内存搜索0xdeadbeef
bt|	显示调用栈信息|	bt：显示当前调用栈
net|	显示各种网络相关的数据|	net：显示网络设备列表
vm	|显示task的基本虚拟内存信息	|vm：类似于/proc/self/maps
btop|	把一个16进制地址转换成它的分页号	|N/A
ptob|	该命令与btop相反，是把一个分页号转换成地址	|N/A
vtop	|显示用户或内核虚拟内存所对应的物理内存|	N/A
ptov|	该命令与vtop相反。把物理内存转换成虚拟内存	|N/A
pte	|16进制页表项转换为物理页地址和页的位设置|	N/A
alias	|显示或建立一个命令的别名	|alias kp kmem -p：以后用kp命令相当于kmem -p
foreach|	用指定的命令枚举	|foreach bt：显示所有进程的调用栈
repeat|	循环执行指定命令	|repeat -1 p jiffies：每个1s执行p jiffies
ascii	|把16进制表示的字符串转化成ascii表示的字符串	|ascii 62696c2f7273752f：结果为/usr/lib
set	|设置要显示的内容，内容一般以进程为单位，也可以设置当前crash的内部变量	|set -p：切换到崩溃进程的上下文环境
p|	print的缩写，打印表达式的值。表达式可以为变量，也可以为结构体	|N/A
dis	|disassemble的缩写。把一个命令或者函数分解成汇编代码	|dis sys_signal：反汇编sys_signal函数
whatis|	搜索数据或者类型的信息	|whatis linux_binfmt：显示linux_binfmt结构体
eval	|计算表达式的值，及把计算结果或者值显示为16、10、8和2进制|	N/A
kmem|	显示当前kernel使用内存状况	|kmem -i：显示kernel使用内存状况
sym|	显示符号所在的虚拟地址，或虚拟地址对应的符号|	sym jiffies：显示jiffies地址
rd	|显示指定内存的内容。缺少的输出格式是十六进制输出|	rd -a linux_banner：显示linux_banner内容
wr|	根据参数指定的写内存。在定位系统出错的地方时，一般不使用该命令	wr |my_debug_flag 1：修改my_debug_flag值为1
gdb|	执行GDB原生命令	|gdb help：执行gdb的help命令
extend|	动态装载或卸载crash额外的动态链接库	|N/A
q	|退出|N/A
exit	|同q，退出|	N/A
help	|帮助命令	|N/A



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

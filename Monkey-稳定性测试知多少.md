 

##### 和您一起终身学习，这里是程序员Android


**Monkey** 在开发中非常常见，本篇主要梳理` monkey` 测试相关知识点。主要包括以下内容
> 一、Android 整机 monkey 测试方法
>二、App monkey 测试方法
>三、判断Monkey 测试方法
> 四、停止monkey测试的方法
> 五、Monkey 命令使用手册
> 六、Monkey Crash Log 分析
>七、Monkey ANR Log 分析
>八、Monkey 测试中关机log 分析
> 九、Monkey 运行机制


# 一、Android 整机 monkey 测试方法

Android 整机测试需要忽略 `crash timeout  security-exceptions`等导致的monkey测试中断，并将` Log` 保存到指定文件中。
比如我要模拟99999 次点击事件，并将测试log保存到monkey_log.txt中，可以使用以下方法：

```
adb shell monkey --ignore-crashes --ignore-timeouts --ignore-security-exceptions --throttle 100 -v 99999 > monkey_log.txt

```

# 二、App monkey 测试方法

执行`app`测试，如遇到`crash `会打印出`crash `信息，方便我们解决`crash`。
`adb shell monkey  -p com.qiyi.video（要测试app的包名）  999999`

忽略`Crash ANR  、安全异常`等测试方法。
比如我要模拟99999 次爱奇艺app的点击事件，并将测试log保存到aiqiyi_log.txt中，可以使用以下方法：
```
adb shell monkey -p  com.qiyi.video（要测试app的包名） --ignore-crashes --ignore-timeouts --ignore-security-exceptions --throttle 100 -v 99999 > aiqiyi_log.txt
```
# 三、判断Monkey 测试方法

判断 Monkey 测试是否正在运行，我们可以使用以下方法：

 ```  
    /**
     * Returns true if Monkey is running.
     */
    public static boolean isMonkeyRunning() {
        return ActivityManager.isUserAMonkey();
    }
```

# 四、停止monkey测试的方法

## 1.查看monkey进程，然后kill掉

查看手机`monkey `进程的命令`adb shell ps |findstr monkey`, 通过稍等进程 **id**（ `adb shell kill -9 18333（monkey进程ID）`），实现停止`monkey `测试，适用于debug版本。
```
C:\Users\Administrator>adb shell ps |findstr monkey
shell     18333 273   1627720 34672 binder_thr 759b3b8884 S com.android.commands.monkey

C:\Users\Administrator>adb shell kill -9 18333
```

![停止monkey 测试的方法 ](https://upload-images.jianshu.io/upload_images/5851256-f3e2da42ece187c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.重启手机

此种方案简单暴力,原理也是杀掉monkey 进程，适用于user版本。

# 五、Monkey 命令使用手册

`monkey` 使用参数命令帮助手册命令如下：`adb shell monkey -help`
![Monkey 参数使用手册](https://upload-images.jianshu.io/upload_images/5851256-c3282d3705d28fd2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- ` -v ` 表示`Log`信息登记
-  `--throttle` 表示毫秒数
- `-s ` 表示发送随机数种子 
- `-p`  表示测试`Monkey app` 包名


![monkey 部分参数](https://upload-images.jianshu.io/upload_images/5851256-c7c5a43aa1faeeeb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 六、Monkey  Crash Log 分析

在抓取的`adb log`中，使用文本编辑器(`建议使用Notepad++,匹配大小写 `)打开，
搜索一下关键字 **CRASH:**

![Monkey log 分析举例 ](https://upload-images.jianshu.io/upload_images/5851256-6f9b66e9c39815e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 七、Monkey  ANR Log 分析

在抓取的`adb log`中，使用文本编辑器(`建议使用Notepad++,匹配大小写 `)打开，
搜索一下关键字  **ANR in**  或 者  **NOT RESPONDING**。

![ANR Log 分析](https://upload-images.jianshu.io/upload_images/5851256-67b9dc95b0a0928b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如有`ANR `还需要将 `data/anr` 下的`trace` 文件`pull` 出来辅助分析` ANR`原因。

##1.导出`ANR` 文件的命令如下：
`adb pull data/anr .`
![导出 ANR 文件](https://upload-images.jianshu.io/upload_images/5851256-d1ec9435e696169d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 八、Monkey 测试中关机Log 分析

`Monkey` 测试过程中关机可以先从以下方法入手。
1.搜索关键字**battery_level** 查看电池电量。
通过次关键字可以在`events_log` 中查看关机时候的电池电量信息、电池电压信息、电池温度信息。

![电池相关信息](https://upload-images.jianshu.io/upload_images/5851256-33a412f7bc4ffef5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 九、Monkey 运行机制

因为Android 系统中已经将`monkey.jar`打包到 `system/framework/`中 ,故`monkey `命令可以在手机上直接运行。
## 1.monkey.jar 后台支持
![monkey jar 包文件](https://upload-images.jianshu.io/upload_images/5851256-71c4c84dae969dd5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![手机 monkey  jar包存放路径](https://upload-images.jianshu.io/upload_images/5851256-c77c753c9f9705b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.手机中monkey bin 文件的支持

执行`monkey `命令的脚本存放地址在`system/bin`目录下，通过此脚本，既可以开始执行`monkey` 相关的命令测试。

![monkey 脚本](https://upload-images.jianshu.io/upload_images/5851256-9b572b2f64a5035c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![手机monkey命令脚本存放地址](https://upload-images.jianshu.io/upload_images/5851256-0b25582821bdce58.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 

##### 和您一起终身学习，这里是程序员Android 

# DuraSpeed
## 1.概念
DuraSpeed  是MTK 在 Android M/N 上开发的进程管理软件，目的是“缓解手机长时间使用后的性能下降问题”。
DuraSpeed 在APP 启动时开始执行，在后台限制“被保护之外”的进程，从而为前台进程提供更多的系统资源。
在Setting--DuraSpeed 中可以查看
 ## 2. 宏控开关
3个Feature开关在ProjectConfig.mk 下

```
//1. DuraSpeed是否默认开启
MTK_RUNNING_BOOSTER_DEFAULT_ON = yes
//2. 是否支持DuraSpeed
MTK_RUNNING_BOOSTER_SUPPORT = yes
//3.APK是否可以升级
MTK_RUNNING_BOOSTER_UPGRADE = yes
```
## 3.DuraSpeed 白名单
```
//1.在platform_list.txt 添加apk白名单包名，这样APK就不会出现在DuraSpeed  ListView 中
alps\frameworks\base\core\java\com\mediatek\runningbooster\platform_list.txt
//2.对应编译生成 的文件
system\etc\runningbooster\platform_list.txt
```
ps:
所有的聊天应用默认允许后台运行，也就是说默认在白名单里，但可以提供用户手动关闭。
## 4.触发脚本开启
开启DuraSpeed ，将会限制一些后台Activity 的运行，进而使前台Activity 运行更加流畅
## 5.MTK SDK API 供后续开发
 MTK 技术保护，需要先申请。
```
//1.无源码APK存放路径
\vendor\mediatek\proprietary\packages\apps\RunningBooster 
//2.生成APK 目录如下
system\app\DuraSpeed
```


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

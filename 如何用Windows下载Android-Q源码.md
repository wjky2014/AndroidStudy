#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
 >一、下载AOSP前的准备
>  二、国内网络下 clone 清华大学开源软件镜像 
>三、编写Python脚本，开始下载android-10.0.0_r40 源码
>四、源码下载工具包

#一、下载AOSP前的准备

想在国内网络下载AOSP源码，需要电脑配置如下环境
- 1.安装Git  
- 2.安装 Python
- 3.配置python脚本，硬盘大于100G

## 1. 安装 Git Bash

[Git官网下载地址](https://git-scm.com/download/win)

## 2.安装Python

[Python 官网下载地址]([https://www.python.org/downloads/](https://www.python.org/downloads/)
)
# 二、国内网络下 clone [清华大学开源软件镜像 ](https://mirrors.tuna.tsinghua.edu.cn/)
## 1. clone 命令
```
// 没有翻墙网络 只能clone 清华镜像
git clone https://aosp.tuna.tsinghua.edu.cn/platform/manifest.git
```
## 2.操作截图

![使用命令如下 clone 清华镜像操作步骤如上](https://upload-images.jianshu.io/upload_images/5851256-20a3c858fe7b23da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 3.操作结果
![Clone 结束如上](https://upload-images.jianshu.io/upload_images/5851256-ca14c8bbaad37e3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 4. 切换要下载的Android源码分支

比如我想下载到`android-10.0.0_r40`的源码，可以使用如下命令：

```
git switch -c android-10.0.0_r40
```
**操作结果如下**

![切换要选择下载的Android版本分支](https://upload-images.jianshu.io/upload_images/5851256-e49b687152f6997d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 三、编写Python脚本，开始下载android-10.0.0_r40 源码

## 1. 自动化下载Android 10 脚本参考如下
```
import xml.dom.minidom
import os
from subprocess import call

## 注意地址中使用的是 "/" 而不是"\", unbantu 跟Windows 是有区别的
 
#代码保存位置，硬盘建议大于100G
rootdir = "E:/AOSP/android_10_0_0_r40"
 
#git 安装路径，可以使用 where git 命令查看 
git = "E:/software/git/path/mingw64/bin/git.exe"

# 刚刚切换 android-10.0.0_r40 目录下的defaul.xml 文件
dom = xml.dom.minidom.parse("E:/AOSP/clone_tsinghua/manifest/default.xml")
root = dom.documentElement

# clone 清华大学镜像库地址 
prefix = git + " clone https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/"
suffix = ".git"
 
if not os.path.exists(rootdir):
    os.mkdir(rootdir)
 
for node in root.getElementsByTagName("project"):
    os.chdir(rootdir)
    d = node.getAttribute("path")
    last = d.rfind("/")
    if last != -1:
        d = rootdir + "/" + d[:last]
        if not os.path.exists(d):
            os.makedirs(d)
        os.chdir(d)
    cmd = prefix + node.getAttribute("name") + suffix
    call(cmd)
```
## 2. 执行下载Android 10 的脚本

双击`downloadAOSP.py `或者执行 `python downloadAOSP.py` 既可以开始下载Android 10 源码，经过一段时间漫长等待，就可以查看Android Q的源码了。

## 3. 开始成功下载源码截图

![双击downloadAOSP.py 即可开始Android 10的源码下载](https://upload-images.jianshu.io/upload_images/5851256-745507e62b973aee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#  四、源码下载工具包

## 1.源码下载工具包地址

[百度网盘下载地址](https://pan.baidu.com/s/1GkGiKROmcfPD7oSL8EYhOw): 提取码: uiv4
```
链接: https://pan.baidu.com/s/1GkGiKROmcfPD7oSL8EYhOw 
提取码: uiv4
```
## 2.源码下载工具包内容

![源码下载工具包](https://upload-images.jianshu.io/upload_images/5851256-82fa5b7dd7c1834d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)






##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、使用ls -l 显示文件的详细信息
>二、Linux下的文件权限分组
>三、drwx 代表的意思
>四、修改文件，用户权限方法



#一、使用ls -l 显示文件的详细信息

`ls -l`是用来显示文件的详细信息，举例如下：
```
wangjie@ubuntu:/opt/qcom$ ls -l 
total 12
drwxr-xr-x 7 root root 4096 Jun  3 15:29 ARM_Compiler_5
drwxr-xr-x 4 root root 4096 Jun  3 15:43 HEXAGON_Tools
drwxr-xr-x 9 root root 4096 Jun  3 15:48 LLVM
```

# 二、Linux下的文件权限分组

`Linux` 下权限共分三组，分别如下：
- 1.文件所有者
- 2.文件所有者所在的组
- 3.其他用户组
 
举例解释如下：
```
drwxr-xr-x 4 root root 4096 Jun  3 15:43 HEXAGON_Tools
```
![](https://upload-images.jianshu.io/upload_images/5851256-403ed5a242f27e85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 三、drwx 代表的意思

名称|简称|代表意义|数值
----|----|----|----
`d`|dir|目录|-
`r`|read|读文件权限|4
`w`|write|写文件权限|2
`x`|-|可执行权限`（通常是以下sh脚本，库文件等）`|1
# 四、修改文件，用户权限方法
使用`chmod` 可以更改文件权限
比如需要修改所有的读`（4）`、写`(2)`、可执行权限`(1)`。就可以使用`Chmod`命令 将所有的权限值相加改变 用户、用户组、其他用户的权限。
其中`7`代表`（4+2+1）`
```
chmod 777 -R . 
```




**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)






##### ，和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、cannot create temp file for here-document: No space left on device
>二、df 查看 Linux 空间使用情况
>三、使用 du 命令查看目录文件占用空间大小
>四、rf 删除没用文件


# 一、cannot create temp file for here-document: No space left on device

编译Android 源码时候莫名其妙的报错，各种查找验证发现代码没问题，使用`tab`自动补全功能 报`-bash: cannot create temp file for here-document: No space left on device`，于是使用`df` 看一下根目录爆满。猜想肯定跟空间爆满有关。

# 二、df 查看 Linux 空间使用情况

使用 `df` 命令 查看磁盘空间分布情况。
发现`/dev/sda4        30G   30G     0 100% /` 已经被占满。
```
wangjie@ubuntu:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            7.8G  4.0K  7.8G   1% /dev
tmpfs           1.6G  4.7M  1.6G   1% /run
/dev/sda4        30G   30G     0 100% /
none            4.0K     0  4.0K   0% /sys/fs/cgroup
none            5.0M     0  5.0M   0% /run/lock
none            7.8G     0  7.8G   0% /run/shm
none            100M     0  100M   0% /run/user
/dev/sda1        60M  3.4M   56M   6% /boot/efi
/dev/sda3       1.8T  1.1T  582G  66% /home
wangjie@ubuntu:~$ 
```
![](https://upload-images.jianshu.io/upload_images/5851256-612f6c948b828563.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 三、使用 du 命令查看目录文件占用空间大小

`Filesystem`下的挂载点 `/dev/sda4  `爆满，使用`du`命令定位根目录下的大文件。
```
root@ubuntu:/home/wangjie# cd /
root@ubuntu:/# du -h --max-depth=1 
3.8M	./lib32
4.0K	./dev
611M	./tmp
4.0K	./srv
1.6G	./usr
du: cannot access ‘./proc/26242/task/26242/fd/4’: No such file or directory
du: cannot access ‘./proc/26242/task/26242/fdinfo/4’: No such file or directory
du: cannot access ‘./proc/26242/fd/3’: No such file or directory
du: cannot access ‘./proc/26242/fdinfo/3’: No such file or directory
0	./proc
55M	./boot
16K	./lost+found
6.7M	./etc
355M	./lib
12M	./sbin
4.2M	./libx32
4.0K	./lib64
0	./sys
4.4G	./opt
22G	./root
4.0K	./mnt
```
使用`du`命令发现 `root`目录下有个`22G`的大文件。
![root 目录下有个22G的文件](https://upload-images.jianshu.io/upload_images/5851256-e1f1d7af4afe1fda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后继续使用`du`命令，进入`root` 继续查看`22G`大文件是什么？
经再次查看发现是之前安装的`ccache`软件生成的垃圾导致的。
```
root@ubuntu:~# du -h --max-depth=1 
4.0K	./.aptitude
4.0K	./.InstallAnywhere
22G	./.ccache_sprd9
8.0K	./.ssh
24K	./.oracle_jre_usage
16K	./.git_template
48K	./.java
40M	./.jack-server
1.2M	./.cache
55M	./.ccache
22G	.
root@ubuntu:~#
```
![22G大文件](https://upload-images.jianshu.io/upload_images/5851256-cc71055d9a7ce1f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 四、rf 删除没用文件

发现大文件后，发现其缓存内容非必须的，可以使用`rm`命令删除掉。

![](https://upload-images.jianshu.io/upload_images/5851256-a4d95a11b56ed759.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后再查看磁盘大小,测试`Tab`自动补全功能`ok`，编译代码`ok`，发现果然是它导致的。


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 和你一起终身学习，这里是程序员 Android
当我电脑硬件内存不够的时候，系统又很卡，怎么办？
硬件不够，虚拟内存来凑。

例如增加4G虚拟内存，操作如下:

```
$ free -m
              total        used        free      shared  buff/cache   available
Mem:          15827         225         188          14       15413       15262
Swap:         24575          30       24545
$ sudo dd if=/dev/zero of=/tmp/big_swap bs=1024 count=2000000
2000000+0 records in
2000000+0 records out
2048000000 bytes (2.0 GB, 1.9 GiB) copied, 4.21505 s, 486 MB/s
$ sudo dd if=/dev/zero of=/tmp/big_swap bs=1024 count=4000000
4000000+0 records in
4000000+0 records out
4096000000 bytes (4.1 GB, 3.8 GiB) copied, 22.6776 s, 181 MB/s
$ sudo du -sh /tmp/big_swap
3.9G	/tmp/big_swap
$ sudo mkswap /tmp/big_swap
mkswap: /tmp/big_swap: insecure permissions 0644, 0600 suggested.
Setting up swapspace version 1, size = 3.8 GiB (4095995904 bytes)
no label, UUID=447b1652-31b5-414c-9b20-73f7a6f8cb25
$ sudo swapon /tmp/big_swap
swapon: /tmp/big_swap: insecure permissions 0644, 0600 suggested.
$ free -m
              total        used        free      shared  buff/cache   available
Mem:          15827         220         166          14       15439       15265
Swap:         28482          30       28451
$ 

```



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

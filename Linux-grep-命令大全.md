


##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、搜索并显示字符串 前后 N 行代码
>二、搜索并显示字符串后 N 行代码
>三、搜索并显示字符串前 N 行的代码
>四、递归搜索字符串，区分大小写
>五、递归搜索字符串，不区分大小写
>六、显示查找字符串所在的行数
>七、grep 更多命令


#一、搜索并显示字符串 `前后 N 行`代码

如果想要搜索并显示结果`前后 N 行`内容，请使用`-C`参数。
`-C `参数代表搜索字符串所在的行。
`-C, --context=NUM         print NUM lines of output context`

#####命令格式如下：
`grep -C N `

备注：
其中`N`代表行数。

#####举例： 
在当前目录下搜索并显示 `low_power_set_value_entries_values` 字符串 `前后 5行`代码内容使用命令如下：
```
grep  "low_power_set_value_entries_values" -r -C 5 .
```
![请使用grep -C N](https://upload-images.jianshu.io/upload_images/5851256-dea2d50c0fffbad0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#二、 搜索并显示字符串`后 N 行`代码

如果想搜索并显示结果字符串的`后 N 行` 代码，请使用 `-A`参数。
`-A `参数代表 `After`意思。
`-A, --after-context=NUM   print NUM lines of trailing context`

##### 命令格式如下：
`grep -A N `

 备注：
其中`N`代表行数。

#####举例： 
在当前目录搜索并显示 `low_power_set_value_entries_values`  字符串 `后 5行`代码 ，可以使用以下命令：
```
grep  "low_power_set_value_entries_values" -r -A 5 .
```
![请使用grep -A N](https://upload-images.jianshu.io/upload_images/5851256-af95c892b49a8f44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#三、 搜索并显示字符串`前 N 行`的代码

请使用 `grep -B N 行数` 来显示要搜索到的字符串的前 N 行 代码。
`-B `参数代表 `Before`意思。
`-B, --before-context=NUM  print NUM lines of leading context`
#####举例： 
在当前目录搜索并显示 `low_power_set_value_entries_values`  字符串` 前 5行`代码 可以使用以下命令：
```
grep  "low_power_set_value_entries_values" -r -B 5 .
```
![请使用grep -B N ](https://upload-images.jianshu.io/upload_images/5851256-232d1bb85471a6e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 四、递归搜索字符串，区分大小写

当我们想要在某些文本中`递归搜索字符串`时候，可以使用`-r`参数。
`-r` 代表递归的意思。
`-r, --recursive           like --directories=recurse`

#####命令格式如下：
`grep "字符串" -r  文件目录`

##### 举例
在当前目录下搜索`aa`字符串方法如下：
```
grep "aa" -r .
```

![使用grep 递归查找文本中的字符串](https://upload-images.jianshu.io/upload_images/5851256-8e945c80bbc11f9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 五、递归搜索字符串，不区分大小写

当我们递归搜索字符串，同时又不想区别大小写字母，可以使用`-i`参数。
`-i ` 表示忽略区分大小写 
` -i, --ignore-case         ignore case distinctions`

#####命令格式如下：
`grep "字符串" -ir  文件目录`

#####举例
在当前目录下搜索`aa`字符串方法如下：
```
grep "aa" -ir .
```
![使用递归方法搜索字符串，并忽略大小写](https://upload-images.jianshu.io/upload_images/5851256-1181e6ab80125ed3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 六、显示查找字符串所在的行数

当我们递归查找字符串，同时想知道在代码多少行时候，可以使用`-n`参数。
`-n`代表行数
`-n, --line-number         print line number with output lines`
#####命令格式如下：
`grep "字符串" -nr  文件目录`
#####举例
在当前目录下搜索`aa`字符串,并显示在文本多少行的方法如下：
```
grep "aa" -nir .
```
![-n 显示字符串所在的行数](https://upload-images.jianshu.io/upload_images/5851256-86cceae53f212b91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#七、grep 更多命令

如需查看`grep` 更多命令，请使用`grep --help`
![grep --help](https://upload-images.jianshu.io/upload_images/5851256-d8dbc127af851327.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

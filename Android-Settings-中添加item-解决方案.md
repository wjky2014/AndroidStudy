
 

#### 和你一起终身学习，这里是程序员Android 
在`Settings`中添加`item`,为自己的`APK`留个接口，在`Android `系统开发中经常会用到，本解决方案适用于`Android N`版本，由于`Android O，Android Go `版本`Settings `存在差异，后续会更新`Android 8.0 `之后的解决方案。 

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:

>1. 在Settings 的AndroidManfest.xml 中注册Activity
>2. 添加自定义实现SettingsPreferenceFragment
>3. 在 Settings添加实现的类
>4. 在 SettingsActivity添加需要实现的类
>5. 添加图片资源，字符串资源





##在Settings 中添加Item ，实现效果如下：

![在Settings 中添加Item](http://upload-images.jianshu.io/upload_images/5851256-bafb7a83238dda34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如何实现，请按以下操作步骤。

#1.在Settings 的AndroidManfest.xml 中注册Activity

- 1.代码路径

`packages\apps\Settings\AndroidManfest.xml `

- 2.修改方案

![Activity 注册方案](http://upload-images.jianshu.io/upload_images/5851256-bffa0a0766cc59db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于没有找到合适图片，本图片引用系统原始图片。
> 注意:
 `<intent-filter android:priority="9"> ` 
值越大，item 在分组内排的会更靠上  `android:value="com.android.settings.category.device" `
此值表示item 放在哪个分组内

#2.添加自定义实现SettingsPreferenceFragment

- 1.代码路径

`packages\apps\Settings\src\com\android\settings\ProgramAndroid.java `

- 2.修改方案

![自定义实现ProgramAndroid类 ](http://upload-images.jianshu.io/upload_images/5851256-e4b76e2c6f3ce5ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#3.在 Settings添加实现的类

- 1.代码路径
`packages\apps\Settings\src\com\android\settings\Settings.java `

- 2.修改方案

![Settings 修改方案](http://upload-images.jianshu.io/upload_images/5851256-4bf817f307616a06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#4.在 SettingsActivity添加需要实现的类

- 1.代码路径
`packages\apps\Settings\src\com\android\settings\SettingsActivity.java `

- 2.修改方案

![修改点1](http://upload-images.jianshu.io/upload_images/5851256-8d23cfe018e19a22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![修改点 2](http://upload-images.jianshu.io/upload_images/5851256-eaf71901dc8878b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![修改点 3](http://upload-images.jianshu.io/upload_images/5851256-a6c60d709cb90ca1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#5. 添加图片资源，字符串资源

- 1.代码路径

字符串路径如下：
`\packages\apps\Settings\res\values\strings.xml `

图片资源路径如下：
`\packages\apps\Settings\res\drawable-xhdpi`

- 2.修改方案

![添加字符串资源](http://upload-images.jianshu.io/upload_images/5851256-62220ed3f7b9bbd1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


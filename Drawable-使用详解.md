

 
##### 和您一起终身学习，这里是程序员Android




本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:

> 一、Drawable 简介
>二、Bitmap 位图 BitmapDrawable
>三、可拉伸图(*.9.png) NinePatchDrawable。
> 四、图层 LayerDrawable
>五、 不同状态图（选择器） StateListDrawable
> 六、级别列表 LevelListDrawable
> 七、转换图像 TransitionDrawable
>  八、插入可绘制对象
>九、 剪裁可绘制对象 ClipDrawable
>十、 缩放可绘制对象 ScaleDrawable
> 十一、形状可绘制对象 ShapeDrawable


# 一、Drawable 简介

`Drawable`  是`Android` 中图像显示的常用方法。
概念：`Drawable `是指可在屏幕上绘制的图形，已经通过`getDrawable(int)`等API检索或者应用到具有 `android:drawable` 和 `android:icon` 等属性的其他` XML` 资源的图形。

## 1.继承关系如下：
```
[java.lang.Object]
   ↳
     android.graphics.drawable.Drawable
```
## 2.Drawable 分类如下：
1.Bitmap 位图 `BitmapDrawable`
2.可拉伸图`(*.9.png)` `NinePatchDrawable。`
3.图层` LayerDrawable`
4.不同状态图（选择器） `StateListDrawable`
5.级别列表` LevelListDrawable`
6.转换图像 `TransitionDrawable`
7.插入可绘制对象
8.剪裁可绘制对象 `ClipDrawable`
9.缩放可绘制对象 `ScaleDrawable`
10.形状可绘制对象 `ShapeDrawable`

## 3. 资源引用：
在 `Java `中：
`R.drawable.filename`

在` XML` 中：
`@[package:]drawable/filename`

# 二、Bitmap 位图 BitmapDrawable

位图图像。`Android `支持以下三种格式的位图文件：`.png`（首选）、`.jpg`（可接受）、`.gif`（不建议）。这些文件保存到 `res/drawable/ `目录中


在构建过程中，可通过` aapt `工具自动优化位图文件，对图像进行无损压缩。例如，不需要超过 `256 `色的真彩色` PNG` 可通过调色板转换为 `8 `位 `PNG`。这样产生的图像质量相同，但所需内存更少。因此请注意，此目录中的图像二进制文件在构建时可能会发生变化。如果您计划将图像解读为比特流以将其转换为位图，请改为将图像放在` res/raw/ `文件夹中，在那里它们不会进行优化

使用方法如下：
##  1.常规位图

- XML 布局中使用方法

```
        <ImageView
            android:id="@+id/img_round"
            android:layout_width="80dp"
            android:layout_height="80dp"
            android:src="@drawable/gril" />
```

- Java 代码中使用方法
```
getResources().getDrawable(R.drawable.xml_bitmap)
```

##  2.XML 位图

- 在XML中创建位图资源文件

注意一下属性使用方法：

1. `antialias`   
           启用、停用抗锯齿  

  2. `dither `
           当位图的像素配置与屏幕不同时（例如：`RGB 8888 `位图和 `RGB 565 `屏幕），启用或停用位图抖动。

    3. `filter `
           启用或停用位图过滤。当位图收缩或拉伸以使其外观平滑时使用过滤。

   4. `mipmap `
          启用或停用` mipmap `提示

   5.`tileMode` 
        定义平铺模式。当平铺模式启用时，位图会重复。重力在平铺模式启用时将被忽略
`xml_bitmap` 位图实现
```
<?xml version="1.0" encoding="utf-8"?>
<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
    android:antialias="true"
    android:dither="true"
    android:filter="false"
    android:gravity="center_vertical|clip_vertical"
    android:mipMap="true"
    android:src="@drawable/gril"
    android:tileMode="repeat" >

    <!--
    antialias   
           启用、停用抗锯齿  
    dither 
           当位图的像素配置与屏幕不同时（例如：ARGB 8888 位图和 RGB 565 屏幕），启用或停用位图抖动。
    filter 
           启用或停用位图过滤。当位图收缩或拉伸以使其外观平滑时使用过滤。
    mipmap 
          启用或停用 mipmap 提示
    tileMode 
        定义平铺模式。当平铺模式启用时，位图会重复。重力在平铺模式启用时将被忽略

    -->

</bitmap>
```

- 引用XML位图资源方法

```
        <ImageView
            android:id="@+id/img_drawable_bitmap"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:src="@drawable/xml_bitmap" />

```


- java 代码实现方法
![java 代码使用位图的方法 ](http://upload-images.jianshu.io/upload_images/5851256-26d5d2c9f3d73d89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#三、可拉伸图(*.9.png) NinePatchDrawable。

`NinePatch `是一种 `PNG` 图像，在其中可定义当视图中的内容超出正常图像边界时 `Android` 缩放的可拉伸区域。此类图像通常指定为至少有一个尺寸设置为 `"wrap_content"` 的视图的背景，而且当视图扩展以适应内容时，九宫格图像也会扩展以匹配视图的大小。`Android` 的标准 `Button `小部件使用的背景就是典型的九宫格图像，其必须拉伸以适应按钮内的文本（或图像）。

## 1. 常规使用方法同其他图片引用方式


```<?xml version="1.0" encoding="utf-8"?>
<nine-patch xmlns:android="http://schemas.android.com/apk/res/android" 
    android:src="@drawable/main_bg_green"
    android:dither="false">
</nine-patch>

```


#四、 图层 LayerDrawable

`LayerDrawable` 是管理其他可绘制对象阵列的可绘制对象。列表中的每个可绘制对象按照列表的顺序绘制，列表中的最后一个可绘制对象绘于顶部。每个可绘制对象由单一 `<layer-list> `元素内的 `<item> `元素表示。
```
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android" >

    <item
        android:id="@+id/bird001"
        android:drawable="@drawable/bird0001_risk">
    </item>
    <item
        android:id="@+id/bird002"
        android:drawable="@drawable/bird0002_risk"
        android:left="50dp"
        android:top="50dp">
    </item>
    <item
        android:id="@+id/bird003"
        android:drawable="@drawable/bird0003_risk"
        android:left="100dp"
        android:top="100dp">
    </item>

</layer-list>
```

#五、 不同状态图（选择器） StateListDrawable

`StateListDrawable `是在 `XML `中定义的可绘制对象，它根据对象的状态，使用多个不同的图像来表示同一个图形。例如，`Button` 小部件可以是多种不同状态（按下、聚焦或这两种状态都不是）中的其中一种，而且可以利用状态列表可绘制对象为每种状态提供不同的背景图片。

您可以在 `XML` 文件中描述状态列表。每个图形由单一 `<selector> `元素内的 `<item> `元素表示。每个` <item> `均使用各种属性来描述应用作可绘制对象的图形的状态。

在每个状态变更期间，将从上到下遍历状态列表，并使用第一个与当前状态匹配的项目 —此选择并非基于“最佳匹配”，而是选择符合状态最低条件的第一个项目。
此方法非常常用，比如状态选择器

- 1. 常规使用方法同其他图片引用方式

- 2. 选择器 XML的使用方式

![状态选择器](http://upload-images.jianshu.io/upload_images/5851256-7f4ea06bd1824161.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#六、 级别列表 LevelListDrawable

管理大量备选可绘制对象的可绘制对象，每个可绘制对象都分配有最大的备选数量。使用` setLevel() `设置可绘制对象的级别值会加载级别列表中` android:maxLevel` 值大于或等于传递到方法的值的可绘制对象资源。
资源引用：
在 Java 中：
`R.drawable.filename`
在 XML 中：
`@[package:]drawable/filename`

```
<?xml version="1.0" encoding="utf-8"?>
<level-list xmlns:android="http://schemas.android.com/apk/res/android" >

    <item
        android:drawable="@drawable/gril"
        android:maxLevel="100"
        android:minLevel="10"/>

</level-list>
```
可通过 `setLevel() `或 `setImageLevel()` 更改级别。

#七、 转换图像 TransitionDrawable

`TransitionDrawable` 是可在两种可绘制对象资源之间交错淡出的可绘制对象。

每个可绘制对象由单一` <transition> `元素内的 `<item>` 元素表示。不支持超过两个项目。要向前转换，请调用 `startTransition()`。要向后转换，则调用 `reverseTransition()`。

## 1. xml 布局声明

```
<?xml version="1.0" encoding="utf-8"?>
<transition xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/bird0001_risk" />
    <item android:drawable="@drawable/bird0002_risk" />
</transition>
```

##2.java 代码中使用

![java代码中TransitionDrawable使用方法 ](http://upload-images.jianshu.io/upload_images/5851256-915167c8a4ef7ccb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#八、插入可绘制对象

在 `XML `文件中定义的以指定距离插入其他可绘制对象的可绘制对象。当视图需要小于视图实际边界的背景时，此类可绘制对象很有用。
```
<?xml version="1.0" encoding="utf-8"?>
<inset xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/gril"
    android:insetBottom="10dp"
    android:insetLeft="10dp"
    android:insetRight="10dp"
    android:insetTop="10dp"
    android:visible="true" >

</inset>

```


#九、 剪裁可绘制对象 ClipDrawable

在 `XML `文件中定义的对其他可绘制对象进行裁剪（根据其当前级别）的可绘制对象。您可以根据级别以及用于控制其在整个容器中位置的重力，来控制子可绘制对象的裁剪宽度和高度。通常用于实现进度栏之类的项目。
##1. xml 初始化剪裁样式
```
<?xml version="1.0" encoding="utf-8"?>
<clip xmlns:android="http://schemas.android.com/apk/res/android"
    android:clipOrientation="vertical"
    android:drawable="@drawable/gril"
    android:gravity="center" >

</clip>
```

##2. java 代码中使用

![ClipDrawable java 代码中使用方法 ](http://upload-images.jianshu.io/upload_images/5851256-0aff2ba5d31cf91b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#十、 缩放可绘制对象 ScaleDrawable

在 `XML `文件中定义的更改其他可绘制对象大小
```
<?xml version="1.0" encoding="utf-8"?>
<scale xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/gril"
    android:scaleGravity="center"
    android:scaleHeight="10%"
    android:scaleWidth="10%" >

</scale>
```



#十一、形状可绘制对象 ShapeDrawable

在 `XML` 中定义的一般形状。


##1. 绘制直线
```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="line" >
    <gradient
        android:angle="45"
        android:centerX="0.5"
        android:centerY="0.5"
        android:centerColor="@android:color/holo_green_dark"
        android:endColor="@android:color/holo_orange_light"
        android:gradientRadius="5dp"
        android:startColor="@android:color/holo_purple"
        android:type="linear"
        android:useLevel="true"/>"
    <padding
        android:left="5dp"
        android:top="5dp"
        android:right="5dp"
        android:bottom="5dp" />
    <size
        android:width="1dp"
        android:height="1dp" />
    <solid
        android:color="@android:color/holo_orange_light" />
    <stroke
        android:width="5dp"
        android:color="@android:color/darker_gray"
        android:dashWidth="5dp"
        android:dashGap="5dp" />
</shape>
```
##2.绘制圆角矩形
```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">

    <!-- 圆角-->
    <corners android:radius="5dp" />
    <!--描边-->
    <stroke
        android:width="1dp"
        android:color="@android:color/holo_blue_light" />

</shape>
```



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
！

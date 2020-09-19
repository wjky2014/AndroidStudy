 
##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
> 一、帧动画 使用详解
> 二、补间动画 使用详解 
>三、属性动画 使用详解

动画在Android 开发中经常会被用到，使用好的动画效果有时候可以达到事半功倍的效果。
Android动画常见的有 帧动画 、补间动画、属性动画 三种。

#一、帧动画 使用详解

## 1. 在xml 声明帧动画  
我们可以在 xml中设置要播放帧动画的图片资源，持续时间，播放属性等。

在xml 声明帧动画举例如下：
```
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android" >

    <item
        android:drawable="@drawable/bird0001_risk"
        android:duration="80"/>
    <item
        android:drawable="@drawable/bird0002_risk"
        android:duration="80"/>
  ... ...
        android:duration="80"/>
    <item
        android:drawable="@drawable/bird0016_risk"
        android:duration="80"/>
    <item
        android:drawable="@drawable/bird0017_risk"
        android:duration="80"/>
    <item
        android:drawable="@drawable/bird0018_risk"
        android:duration="80"/>
    <item
        android:drawable="@drawable/bird0019_risk"
        android:duration="80"/>
    <item
        android:drawable="@drawable/bird0020_risk"
        android:duration="80"/>

</animation-list>
```
## 2.控件中帧动画 xml文件

我们使用帧动画的时候可以通过`@anim/自定义文件名`引用我们定义的xml动画资源。
ImageView 中将帧动画设置为背景的方法举例如下：
```
    <ImageView
        android:id="@+id/img"
        android:layout_width="80dp"
        android:layout_height="80dp"
        android:layout_gravity="center_horizontal"
        android:background="@anim/frame_animation" />
```

# 二、补间动画 使用详解 

补间动画也是Android中常用的动画之一，相对属性动画来说，补间动画的点击事件不会跟着动画的位置变化而变化。因为属性动画的点击事件会随着动画的位置变化而变化，后续将逐渐被属性动画替代。

##1. 补间动画分类

补间动画 常用的分类如下：
  1. 透明动画 AlphaAnimation
  2. 旋转动画 ScaleAnimation
  3. 缩放动画 RotateAnimation
  4. 平移动画 TranslateAnimation
  5. 动画集合 AnimationSet
  6. XML 实现动画效果

## 2. 透明动画

 AlphaAnimation 透明动画 可以设置动画的透明效果，执行时间，重复方式、重复次数等等。
AlphaAnimation  使用举例如下：
```
 /**
         * 透明度动画AlphaAnimation 从不透明到完全透明
         * **/
        // 起始透明度 到结束透明度 不透明到透明（1f-0f）
        AlphaAnimation alphaAnimation = new AlphaAnimation(0.0f, 1.0f);
        // 动画执行时间
        alphaAnimation.setDuration(4000);
        // 设置重复次数
        alphaAnimation.setRepeatCount(2);
        // 重复模式
        alphaAnimation.setRepeatMode(Animation.RESTART);
        // alphaAnimation.setRepeatMode(Animation.REVERSE);
        // 保持结束时候的状态
        alphaAnimation.setFillAfter(true);
        mImageView.startAnimation(alphaAnimation);
```
##   3. 旋转动画

 RotateAnimation 可以设置旋转动画的旋转角度、持续时间、重复次数等等。

RotateAnimation  使用举例如下：

```
   /**
         * 旋转动画RotateAnimation,旋转360度
         **/

        RotateAnimation rotateAnimation = new RotateAnimation(0, 360);
        rotateAnimation.setDuration(2000);
        rotateAnimation.setRepeatCount(2);
        mImageView.startAnimation(rotateAnimation);
```
##   4. 缩放动画

ScaleAnimation可以设置缩放动画的倍数，持续时间，重复模式等等。

ScaleAnimation使用举例如下：

```
        /**
         * 缩放动画 ScaleAnimation使用方法 缩放2倍
         * */
        ScaleAnimation scaleAnimation = new ScaleAnimation(1, 2, 1, 2);
        scaleAnimation.setDuration(2000);
        scaleAnimation.setRepeatCount(2);
        scaleAnimation.setRepeatMode(Animation.RESTART);
        mImageView.startAnimation(scaleAnimation);
```
##   5. 平移动画

TranslateAnimation可以设置平移动画的举例，持续时间，重复模式等等。

TranslateAnimation 使用举例如下：

```
    /***
         * 平移动画TranslateAnimation 从x,y 轴 从（0,0）平移到（300,200） *
         **/
        TranslateAnimation translateAnimation = new TranslateAnimation(0,
                300.f, 0, 200.f);
        translateAnimation.setDuration(2000);
        translateAnimation.setRepeatCount(2);
        translateAnimation.setRepeatMode(Animation.RESTART);
        mImageView.startAnimation(translateAnimation);
````
## 6. 动画集合 

 AnimationSet，可以实现设置AlphaAnimation、TranslateAnimation 、RotateAnimation、RotateAnimation 4种动画累加在一起，进而实现非一般的动画效果。

动画集合效果使用举例如下：
```
   /***
         * 动画集合使用效果如下：
         * ***/
        AnimationSet sets = new AnimationSet(true);

        AlphaAnimation alphaAnimation1 = new AlphaAnimation(0.0f, 1.0f);
        TranslateAnimation translateAnimation1 = new TranslateAnimation(0,
                100.f, 0, 100.f);
        RotateAnimation rotateAnimation1 = new RotateAnimation(0, 360);
        ScaleAnimation scaleAnimation1 = new ScaleAnimation(1, 2, 1, 2);

        // 将动画添加到set集合中
        sets.addAnimation(alphaAnimation1);
        sets.addAnimation(translateAnimation1);
        sets.addAnimation(rotateAnimation1);
        sets.addAnimation(scaleAnimation1);

        sets.setDuration(4000);
        sets.setRepeatCount(2);
        sets.setRepeatMode(Animation.RESTART);
        mImageView.startAnimation(sets);
```

##  7. XML 实现动画效果

实现动画有两种方式 ，一种是通过xml 设置，另一种是通过Java代码动态设置，上面讲的是Java代码中的动态设置，下面我们来看xml代码中的动画设置实现方法。

- 1.XML文件声明动画内容

比如我们在`res/anim/hyperspace_jump.xml`中通过set 集合，声明旋转以及缩放动画方法举例如下：

```
<set android:shareInterpolator="false">
    <scale
        android:interpolator="@android:anim/accelerate_decelerate_interpolator"
        android:fromXScale="1.0"
        android:toXScale="1.4"
        android:fromYScale="1.0"
        android:toYScale="0.6"
        android:pivotX="50%"
        android:pivotY="50%"
        android:fillAfter="false"
        android:duration="700" />
    <set android:interpolator="@android:anim/decelerate_interpolator">
        <scale
           android:fromXScale="1.4"
           android:toXScale="0.0"
           android:fromYScale="0.6"
           android:toYScale="0.0"
           android:pivotX="50%"
           android:pivotY="50%"
           android:startOffset="700"
           android:duration="400"
           android:fillBefore="false" />
        <rotate
           android:fromDegrees="0"
           android:toDegrees="-45"
           android:toYScale="0.0"
           android:pivotX="50%"
           android:pivotY="50%"
           android:startOffset="700"
           android:duration="400" />
    </set>
</set>
```

- 2.Java 代码中加载xml动画文件

完成在xml定义完动画后，我们可以在Java代码中动态引用xml文件，使用方法举例如下：

 ```
    ImageView spaceshipImage = (ImageView) findViewById(R.id.spaceshipImage);
    Animation hyperspaceJumpAnimation = AnimationUtils.loadAnimation(this, R.anim.hyperspace_jump);
    spaceshipImage.startAnimation(hyperspaceJumpAnimation);
```

#三、 属性动画 使用详解

属性动画是补间动画的延伸，主要解决动画点击事件可以随位置到改变而改变，将来会替换补间动画。

##1. 属性动画分类
 1. ValueAnimator
 2. ObjectAnimator
 3. TypeEvaluator

## 2. ValueAnimator 动画

ValueAnimator  使用方法如下：
```
    ValueAnimator animation = ValueAnimator.ofFloat(0f, 100f);
    animation.setDuration(1000);
    animation.start();
```
## 3. ObjectAnimator动画


###1.  属性动画分类 

属性动画跟补间动画类似主要分为以下几类，
  1. 透明动画 alpha
  2. 旋转动画 rotation
  3. 缩放动画 scaleX
  4. 平移动画 translationX
  5. 动画集合 AnimatorSet
 

###   2. 旋转动画
rotation 可以设置旋转动画的旋转角度、持续时间、重复次数等等。

rotation 使用举例如下：
```
     /**
         * 旋转动画 rotation
         * */
        ObjectAnimator rotationanimator = ObjectAnimator.ofFloat(mImageView,
                "rotation", 0, 360);
        rotationanimator.setDuration(2000);
        rotationanimator.setRepeatCount(2);
        rotationanimator.setRepeatMode(Animation.RESTART);
        rotationanimator.start();
```

###   3. 缩放动画

 scale可以设置缩放动画的倍数，持续时间，重复模式等等。

scaleX 使用举例如下：

```
       /**
         * scaleX 缩放动画
         *
         * */
        ObjectAnimator scaleXanimator = ObjectAnimator.ofFloat(mImageView,
                "scaleX", 0, 2);
        scaleXanimator.setDuration(2000);
        scaleXanimator.setRepeatCount(2);
        scaleXanimator.setRepeatMode(Animation.RESTART);
        scaleXanimator.start();
```

###  4. 平移动画
 translation可以设置平移动画的举例，持续时间，重复模式等等。

translationX 使用举例如下：

```
      /**
         * translationX平移动画
         *
         * */
        ObjectAnimator translationanimator = ObjectAnimator.ofFloat(mImageView,
                "translationX", 0, 200f);
        translationanimator.setDuration(2000);
        translationanimator.setRepeatCount(2);
        translationanimator.setRepeatMode(Animation.RESTART);
        translationanimator.start();

```
### 5. 透明动画 
  alpha透明动画 可以设置动画的透明效果，执行时间，重复方式、重复次数等等。

alpha 使用举例如下：

```
   /**
         * alpha 透明动画 1.属性动画作用在谁身上 2.属性名称 3.属性的变化范围值 透明值
         * **/
        ObjectAnimator alphaanimator = ObjectAnimator.ofFloat(mImageView,
                "alpha", 0, 1.0f);
        alphaanimator.setDuration(4000);
        alphaanimator.setRepeatCount(2);
        alphaanimator.start();
```
###   6. 动画集合
AnimationSet，可以实现设置alpha、scale、translation、rotation 4种动画累加在一起，进而实现非一般的动画效果。

动画集合效果使用举例如下：
```
 /**
         * 动画集合效果 rotation
         * */

        AnimatorSet animatorSet = new AnimatorSet();
        ObjectAnimator animator1 = ObjectAnimator.ofFloat(mImageView, "alpha",
                0, 1.0f);
        ObjectAnimator animator2 = ObjectAnimator.ofFloat(mImageView,
                "translationX", 0, 100f);
        ObjectAnimator animator3 = ObjectAnimator.ofFloat(mImageView, "scaleX",
                0, 3);
        ObjectAnimator animator4 = ObjectAnimator.ofFloat(mImageView,
                "rotation", 0, 90);
        List<Animator> list = new ArrayList<Animator>();

        // 将动画集合添加到list集合中
        list.add(animator1);
        list.add(animator2);
        list.add(animator3);
        list.add(animator4);

        // 播放集合中的动画
        animatorSet.playSequentially(list);
        animatorSet.setDuration(2000);
        animatorSet.start();
```

###  7. 动画监听事件

动画监听事件主要用来监听动画播放时的Listener时间，比如监听动画播放取消onAnimationCancel，动画播放完成onAnimationEnd的事件。
动画监听事件举例如下：

```
  scaleXanimator.addListener(new AnimatorListenerAdapter() {
            @Override
            public void onAnimationCancel(Animator animation) {
                super.onAnimationCancel(animation);
            }

            @Override
            public void onAnimationEnd(Animator animation) {
                super.onAnimationEnd(animation);
            }

            @Override
            public void onAnimationRepeat(Animator animation) {
                super.onAnimationRepeat(animation);
            }

            @Override
            public void onAnimationStart(Animator animation) {
                super.onAnimationStart(animation);
            }

            @Override
            public void onAnimationPause(Animator animation) {
                super.onAnimationPause(animation);
            }

            @Override
            public void onAnimationResume(Animator animation) {
                super.onAnimationResume(animation);
            }
        });
```



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

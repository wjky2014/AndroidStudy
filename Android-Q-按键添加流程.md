#### 和你一起终身学习，这里是程序员 Android



本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
 >一、驱动通过GPIO连接的按键
>二、Framework 层添加按键响应方法


# 一、驱动通过GPIO连接的按键

此类按键采用GPIO来连接，通过监测其中断来判断按键的动作，需要根据具体硬件设计在项目对应的dts文件配置gpio_keys节点。

底层驱动主要修改以下两个文件上报键值。

## 1.修改 sp9863a-3c10.dts

修改方法如下：
```
+++ b/idh.code/bsp/kernel/kernel4.14/arch/arm64/boot/dts/sprd/sp9863a-3c10.dts
	gpio-keys {
		compatible = "gpio-keys";
		key-power {
			label = "Power Key";
			linux,code = <KEY_POWER>;
			gpios = <&pmic_eic 1 GPIO_ACTIVE_LOW>;
			debounce-interval = <2>;
			wakeup-source;
		};
+
+               key-smart {
+                       label = "Smart Key";
+                       linux,code = <KEY_OK>;/* linux下的key code，linux下input.h中有定义键值 */
+                       gpios = <&ap_gpio 53 GPIO_ACTIVE_LOW>;/* 按键连接的GPIO */
+                       debounce-interval = <2>;/* 按键去抖时间，单位ms，如果出现按键不稳定，请适当加大 */
+                       wakeup-source;
+               };
	};
```

## 2. 修改pinmap-sp9863a.c文件
修改方法如下：
```
+++ b/idh.code/bsp/bootloader/u-boot15/board/spreadtrum/sp9863a_3c10/pinmap-sp9863a.c
@@ -348,8 +348,8 @@ static pinmap_t pinmap[]={
 {REG_MISC_PIN_LVDSRF0_DACON,            BITS_PIN_DS(1)|BIT_PIN_NULL|BIT_PIN_NUL|BIT_PIN_SLP_AP|BIT_PIN_SLP_NUL|BIT_PIN_SLP_OE},//LCM_SOURCE_AVEEEN
 {REG_PIN_SPI2_CSN,                      BITS_PIN_AF(2)},
 {REG_MISC_PIN_SPI2_CSN,                 BITS_PIN_DS(1)|BIT_PIN_NULL|BIT_PIN_WPU|BIT_PIN_SLP_CM4|BIT_PIN_SLP_WPU|BIT_PIN_SLP_IE},//PROX_INT
-{REG_PIN_SPI2_DO,                       BITS_PIN_AF(2)},
-{REG_MISC_PIN_SPI2_DO,                  BITS_PIN_DS(1)|BIT_PIN_NULL|BIT_PIN_NUL|BIT_PIN_SLP_CM4|BIT_PIN_SLP_NUL|BIT_PIN_SLP_OE},//M_RSTN
+{REG_PIN_SPI2_DO,                       BITS_PIN_AF(3)},
+{REG_MISC_PIN_SPI2_DO,                  BITS_PIN_DS(1)|BIT_PIN_NULL|BIT_PIN_WPU|BIT_PIN_SLP_AP|BIT_PIN_SLP_WPU|BIT_PIN_SLP_IE},//SMART_KEY
 {REG_PIN_SPI2_DI,                       BITS_PIN_AF(3)},
 {REG_MISC_PIN_SPI2_DI,                  BITS_PIN_DS(1)|BIT_PIN_NULL|BIT_PIN_WPD|BIT_PIN_SLP_AP|BIT_PIN_SLP_WPD|BIT_PIN_SLP_Z},//NC
 {REG_PIN_SPI2_CLK,                      BITS_PIN_AF(2)},
```
 

# 二、Framework 层添加按键响应方法

通过`getevent `查看驱动调节的按键值是否上传ok。

命令查看方法如下：
```
C:\Users\Administrator>adb shell getevent
 
/dev/input/event2: 0001 0160 00000001
/dev/input/event2: 0000 0000 00000000
/dev/input/event2: 0001 0160 00000000
/dev/input/event2: 0000 0000 00000000
                                                                                                 
```
其中：
`0160`: 是十六进制数，对应十进制数为352.
`1`，`0`: 是指按下和弹起的动作。

## 1.在 gpio-keys.kl 文件中添加自定义key值

在kl文件中我们可以仿照power键添加key 值。
```
+++ b/idh.code/device/sprd/sharkl3/common/rootdir/system/usr/keylayout/gpio-keys.kl
@@ -3,3 +3,4 @@ key 115     VOLUME_UP       WAKE
 key 116     POWER           WAKE
 key 212     CAMERA          WAKE
 key 0x210   FOCUS           WAKE
+key 352     KEY_OK          WAKE

```
这样就可以完成物理按键 kl 文件到 KEY_OK的映射。

这个kl 文件是通过`DeviceCommon.mk `中编译到系统中，在手机/system/usr/keylayout目录下可以找到。
不同平台代码可能存在差异。

`device\sprd\sharkl3\common\DeviceCommon.mk`的部分代码如下：

```
PRODUCT_COPY_FILES += \
    $(LOCAL_PATH)/rootdir/root/init.common.rc:$(TARGET_COPY_OUT_VENDOR)/etc/init/hw/init.common.rc \
    $(LOCAL_PATH)/recovery/init.recovery.common.rc:root/init.recovery.common.rc \
    $(BOARDDIR)/recovery/init.recovery.$(TARGET_BOARD).rc:root/init.recovery.$(TARGET_BOARD).rc \
    $(LOCAL_PATH)/rootdir/system/usr/keylayout/gpio-keys.kl:$(TARGET_COPY_OUT_VENDOR)/usr/keylayout/gpio-keys.kl \
     
```
## 2. 在Generic.kl 文件中添加key 值

修改方法如下：

```
+++ b/idh.code/frameworks/base/data/keyboards/Generic.kl
@@ -299,7 +299,7 @@ key 317   BUTTON_THUMBL
 key 318   BUTTON_THUMBR

-# key 352 "KEY_OK"
+key 352   KEY_OK
 key 353   DPAD_CENTER
 # key 354 "KEY_GOTO"
 # key 355 "KEY_CLEAR"
```
## 3.在 qwerty.kl文件中添加key值
 
修改方法如下：

```
+++ b/idh.code/frameworks/base/data/keyboards/qwerty.kl
@@ -129,3 +129,4 @@ key 581   STEM_PRIMARY
 key 582   STEM_1
 key 583   STEM_2
 key 584   STEM_3
+key 352   KEY_OK

```

## 4.在Native 层添加keycode 值与标签

注意下面的289 keycode 值，是延续上面288 keycode 的值，跟驱动上报的352不一样，那是底层的数值，上层最好跟底层差分。

修改方法如下：

```
+++ b/idh.code/frameworks/native/include/android/keycodes.h
@@ -776,8 +776,11 @@ enum {
     AKEYCODE_THUMBS_DOWN = 287,
     /** Used to switch current account that is consuming content.
      * May be consumed by system to switch current viewer profile. */
-    AKEYCODE_PROFILE_SWITCH = 288
-
+    AKEYCODE_PROFILE_SWITCH = 288,
+    /**
+       *Nokia custom key 
+       **/
+    AKEYCODE_KEY_OK = 289,
```

同样我们仿照288的定义，在InputEventLabels.h 添加标签定义

```
+++ b/idh.code/frameworks/native/include/input/InputEventLabels.h
@@ -328,7 +328,7 @@ static const InputEventLabel KEYCODES[] = {
     DEFINE_KEYCODE(THUMBS_UP),
     DEFINE_KEYCODE(THUMBS_DOWN),
     DEFINE_KEYCODE(PROFILE_SWITCH),
+    DEFINE_KEYCODE(KEY_OK),
```

## 5. 在attrs.xml 中添加属性值

修改方法如下：
```
+++ b/idh.code/frameworks/base/core/res/res/values/attrs.xml
@@ -1924,6 +1924,7 @@
         <enum name="KEYCODE_THUMBS_UP" value="286" />
         <enum name="KEYCODE_THUMBS_DOWN" value="287" />
         <enum name="KEYCODE_PROFILE_SWITCH" value="288" />
+               <enum name="KEYCODE_KEY_OK" value="289" />
     </attr>
```
## 6.在KeyEvent 中添加key 值方便PhoneWindowMangager中调用

修改方法如下：
```
+++ b/idh.code/frameworks/base/core/java/android/view/KeyEvent.java
@@ -823,6 +823,11 @@ public class KeyEvent extends InputEvent implements Parcelable {
      * consuming content. May be consumed by system to set account globally.
      */
     public static final int KEYCODE_PROFILE_SWITCH = 288;
+        /**
+     * Integer value of the last KEYCODE. Nokia custom  ok key.
+     * @hide
+     */
+       public static final int KEYCODE_KEY_OK = 289;
 
```

## 7. 最后我们在PhoneWindowManager 中处理按键行为

调通之后，我们就可以在PWM 的` interceptKeyBeforeDispatchingInner `方法处理我们想做的事情，比如 吊起Google Assist等。
```
+++ b/idh.code/frameworks/base/services/core/java/com/android/server/policy/PhoneWindowManager.java

@@ -2870,7 +2870,15 @@ public class PhoneWindowManager extends AbsPhoneWindowManager implements WindowM
         if (mPendingCapsLockToggle && !KeyEvent.isMetaKey(keyCode) && !KeyEvent.isAltKey(keyCode)) {
             mPendingCapsLockToggle = false;
         }
-
+               // add for key ok
+               if(keyCode == KeyEvent.KEYCODE_KEY_OK){
+                       if("Nokia_India".equals(android.os.Build.CUSTOMER_SKU)){
+                         // launch Google Assist
+                         launchAssistAction(null, event.getDeviceId());
+                       }
+                       return -1;
+               }
+               // add for key ok
```
##### 关注程序员Android，回复`按键添加`,既可获取完整代码。 



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

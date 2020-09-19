
##### 和您一起终身学习，这里是程序员Android 


本篇文章主要介绍 `Android` 开发中 `Android.mk `部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、Android.mk 简介
>二、Android.mk 的基本格式
>三、Android.mk 深入学习一
>四、 Android.mk 深入学习二
>五、 Android.mk 深入学习三
>六、 Android.mk 判断语句





#  一 、Android.mk 简介

Android.mk 是Android 提供的一种makefile 文件,注意用来编译生成（exe，so，a，jar，apk）等文件。

![Android.mk生成文件](https://upload-images.jianshu.io/upload_images/5851256-408979984ea26ff3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 二、Android.mk 的基本格式

Android.mk 基本格式如下

```
# 定义模块当前路径
LOCAL_PATH := $(call my-dir)  
#清空当前环境变量
include $(CLEAR_VARS)  
................  
# 引入头文件等
LOCAL_xxx       := xxx
#编译生成的文件名  
LOCAL_MODULE    := hello  
#编译该模块所需的源码
LOCAL_SRC_FILES := hello.c  
#引入jar包等
LOCAL_xxx       := xxx  
................  
#编译生成文件的类型 
#LOCAL_MODULE_CLASS  、JAVA_LIBRARIES
#APPS 、 SHARED_LIBRARIES
#EXECUTABLES 、 ETC
include $(BUILD_EXECUTABLE)  
```


# 三、Android.mk 深入学习一

使用Android.mk 可以编译多个目标文件：

![Android.mk 编译多个目标文件](https://upload-images.jianshu.io/upload_images/5851256-07527049ff30d090.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**编译动态库**

C/C++ 文件编译生成静态库.so文件参考如下

```
LOCAL_PATH := $(call my-dir)    
include $(CLEAR_VARS)    
# 生成libhell.so
LOCAL_MODULE = libhello    

LOCAL_CFLAGS = $(L_CFLAGS)    
LOCAL_SRC_FILES = hello.c  
LOCAL_C_INCLUDES = $(INCLUDES) 
LOCAL_SHARED_LIBRARIES := libcutils    
LOCAL_COPY_HEADERS_TO := libhello   
LOCAL_COPY_HEADERS := hello.h   

#编译动态库 BUILD_SHARED_LIBRARY

include $(BUILD_SHARED_LIBRARY)   

```

**编译静态库**

C/C++ 文件编译生成静态库.a文件参考如下
```

#编译静态库    
LOCAL_PATH := $(call my-dir)    
include $(CLEAR_VARS)    
# 生成libhell.a
LOCAL_MODULE = libhello

LOCAL_CFLAGS = $(L_CFLAGS)    
LOCAL_SRC_FILES = hello.c    
LOCAL_C_INCLUDES = $(INCLUDES)    
LOCAL_SHARED_LIBRARIES := libcutils    
LOCAL_COPY_HEADERS_TO := libhello   
LOCAL_COPY_HEADERS := hellos.h   

 # 编译 静态库    BUILD_STATIC_LIBRARY
include $(BUILD_STATIC_LIBRARY) 
```

# 四、 Android.mk 深入学习二

![Android.mk 引用资源](https://upload-images.jianshu.io/upload_images/5851256-ee3dd6cd036987cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**引用静态库**
`LOCAL_STATIC_LIBRARIES += libxxxxx`


```
LOCAL_STATIC_LIBRARIES := \
    ...
    libxxx2 \
    libxxx \
```

**引用动态库**
`LOCAL_SHARED_LIBRARIES += libxxxxx`


```
LOCAL_SHARED_LIBRARIES := liblog libnativehelper libGLESv2
```
**引用第三方库文件**
`LOCAL_LDFLAGS:=-L/PATH -Lxxx`


```
LOCAL_LDFLAGS := $(LOCAL_PATH)/lib/libtest.a
```

**引用第三方头文件**
`LOCAL_C_INCLUDES :=path`

eg:
```
LOCAL_C_INCLUDES = $(INCLUDES)
```

#五、 Android.mk 深入学习三

![Android.mk 深入学习三](https://upload-images.jianshu.io/upload_images/5851256-ff95a3b33b185b07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**编译apk**
```
  LOCAL_PATH := $(call my-dir)
  include $(CLEAR_VARS)
  LOCAL_SRC_FILES := $(call all-subdir-java-files)
  # 生成hello apk
  LOCAL_PACKAGE_NAME := hello
  include $(BUILD_PACKAGE)
```

**编译jar包**

```
  LOCAL_PATH := $(call my-dir)
  include $(CLEAR_VARS)
  LOCAL_SRC_FILES := $(call all-subdir-java-files)
  # 生成 hello
  LOCAL_MODULE := hello
  # 编译生成静态jar包
  include $(BUILD_STATIC_JAVA_LIBRARY)
  #编译生成共享jar
  include $(BUILD_JAVA_LIBRARY)
```
- 静态jar包：
 
` include $(BUILD_STATIC_JAVA_LIBRARY)`
使用.class文件打包而成的JAR文件，可以在任何java虚拟机运行

- 动态jar包：

`include $(BUILD_JAVA_LIBRARY)`
在静态jar包基础之上使用.dex打包而成的jar文件，.dex是android系统使用的文件格式。

**APK 依赖jar**

```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
# 静态jar包
LOCAL_STATIC_JAVA_LIBRARIES := static-library
#动态jar包
LOCAL_JAVA_LIBRARIES := share-library

LOCAL_SRC_FILES := $(call all-subdir-java-files)
LOCAL_PACKAGE_NAME := hello
include $(BUILD_PACKAGE)
```

**预编译jar包**
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
#指定编译生成的文件类型
LOCAL_MODULE_CLASS := JAVA_LIBRARIES
LOCAL_MODULE := hello
LOCAL_SRC_FILES :=  $(call all-subdir-java-files)
# 预编译
include $(BUILD_PREBUILT)
```
预编译文件类型如下：
- 1.LOCAL_MODULE_CLASS：
编译文件类型

- 2.JAVA_LIBRARIES：
dex归档文件

- 3.APPS：
APK文件

- 4.SHARED_LIBRARIES：
动态库文件

- 5.EXECUTABLES：
二进制文件

- 6.ETC：
其他文件格式

# 六、 Android.mk 判断语句

Android.mk 中的判断语句
```
ifeq($(VALUE), x)	#ifneq
  do_yes
else
  do_no
endif

```
ifeq/ifneq：根据判断条件执行相关编译





**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

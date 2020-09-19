 
##### 和您一起终身学习，这里是程序员Android 

`Android `虚拟机 是`app`运行的基石，本篇主要参考`MTK`解决方案，部分方案仅适合`MTK`平台手机。

通过本篇文章阅读，你将收获以下知识点:
>1.Java 语言在Android 上运行流程
>2.虚拟机发展过程
>3.Android Dalvik 模式
>4.Android N 中dex2oat 原理以及模式
>5.如何判断dex2oat 采用的相关参数
>6.如何查看dex2oat 的log
>7.什么时候进行dex2oat
>8.手机反应慢的原因
>9.解决手机反应慢的方法




## 1.Java 语言在Android 上运行流程

![Java语言在android 运行流程](http://upload-images.jianshu.io/upload_images/5851256-fe56a5bc0e404716.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.虚拟机发展过程
* 1.初期Dalvik
* 2.中期Dalvik
* 3.后期ART模式

    为了让APK在不同的虚拟机都可以运行，Google 采取了适配器模式，在让虚拟机运行之前先执行 dexopt ，即将dex 文件优化成odex 文件，可以让虚拟机更加优化的执行。

   在ART 虚拟机中，dexopt 将dex文件优化成二进制格式的问题，从而可以让ART虚拟机执行。
   dexopt 会调用dex2oat 进行优化，dex2oat 的任务是将原来的dex文件进行预翻译，从而可以加快app运行的时间，但是由于某些app比较复杂，所以优化的时间就比较长。
    优化是以dex 文件中的Method 方法为单位，dex2oat 在优化时候，会根据需求优化一定量的Method，即不是所有的Method 都回翻译成oat模式。
## 3.Android Dalvik 模式
根据优化Method量的多少分的模式

![Android Dalvik ](http://upload-images.jianshu.io/upload_images/5851256-7d493ec21cf8f9c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 4. Android  N 中dex2oat 原理以及模式

N版本当中强化了JIT模式。JIT模式是Just in time 的简称。意思是在运行的时候，根据method有使用频率来决定是否要对某一个方法进行优化。虚拟机会统计每一个方法被执行的次数。如果某一个方法执行的次数比较多，达到一定的阈值，就会将升级为hot method，并将其记录在一个profile当中。在系统空闲并且在充电的时候，只将这些方法进行优化。在运行的时候，也会对这些方法进行优化，以方便在运行的时候使用。

* 1.Android  N dex2oat 模式图 

![Android N dex2oat 模式图 ](http://upload-images.jianshu.io/upload_images/5851256-2adcf2ec6bc9388a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 2.Android  N dex2 oat 原理图

![Android  N dex2oat 原理图](http://upload-images.jianshu.io/upload_images/5851256-f1ada531f8cf287c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 5. 如何判断dex2oat 采用的相关参数

dex2oat 的参数会在代码中通过如下函数打印
* 1.  L 版本

 L版本会无条件打印
```
 timings.StartTiming("dex2oat Setup");
1436  LOG(INFO) << CommandLine();
```
* 2 . M 和N版本

在M和N当中会有条件的打印不同的内容。
例如在会在debug版本中打印非常详细的dex2oat参数，而在非debug版本中打印部分内容。
```
if (kIsDebugBuild || dex2oat.IsImage() || dex2oat.IsHost() || !kIsTargetBuild) {
2144    LOG(INFO) << CommandLine();
2145  } else {
2146    LOG(INFO) << StrippedCommandLine();
2147  }
```
##  6.如何查看dex2oat 的log
###  搜索关键字

* dex2oat 
*  dex-file
被优化的dex 文件信息
*  oat-fd
* oat-location
 oat 存放位置
* compiler-filter
判断采用的那种优化模式

eg1:
```
 01-01 04:28:03.471265  5654  5654 I dex2oat : Starting dex2oat.

01-01 04:28:37.548439  5654  5654 I dex2oat : dex2oat took 34.077s (threads: 4) arena alloc=23MB java alloc=14MB native alloc=88MB free=5MB 01-01

其中：dex2oat tokk 34.077s表示 dex2oat的耗时情况，（threads:4）表示，一共用了4个线程做这件事情。
```
eg2:
```
01-01 08:34:27.063 I/dex2oat ( 7714): /system/bin/dex2oat --runtime-arg -classpath 
--runtime-arg --instruction-set=arm --instruction-set-features=default --runtime-arg -Xrelocate --boot-image=/system/framework/boot.art 
--dex-file=/data/data/com.tencent.mm/app_dex/secondary-2.dex.jar --oat-fd=62 --oat-location=/data/data/com.tencent.mm/app_cache/secondary-2.dex.dex 
--runtime-arg -Xms64m --runtime-arg -Xmx512m  --compiler-filter=speed

M 和 N 版本，可以通过会打印出较多的dex2oat参数。

上面的Log中，可以得出很多信息来：
1.被优化的dex文件信息，可以通过 --dex-file获取，形如上面我们就知道优化的是微信里面的内容，因为--dex-file参数的dex文件位于
   com.tencent.mm目录下面的。
2.优化之后oat存储的位置，可以看到--oat-location其是保存在包名下面的app_cache目录下面的。
3.是对插件优化还是对APK本身优化。在L的时候所有的PMS优化的内容都会放在data/dalvik-cache目录下面，所以如果oat放的位置是data/
  dalvik-cache目录下面，那么就是优化主APK,否则就是优化插件的。
  而在M和N当中，优化的结果会放在自己的包下面，但是不会是这种类似app_cache这种，根据这个可以判断是对插件的优化。
4.优化采用的模式
   根据--compiler-filter可以判断采用的是哪种模式。

如果我们关注的信息没有打印出来，可以修改上面打印这些Log的条件。
```
## 7.什么时候进行dex2oat 
dex2oat一般发生在如下的时候：

* 第一次开机
``` 
当第一次开机的时间,PackageManagerService会扫描手机中的各个目录，
当检测到APK之后，就会执行安装流程，在安装的时候，就会对这些APP进行dex2oat操作。
如果说APP比较多，或者dex2oat比较慢，那么开机时间就会比较长。
```
* 安装APP
```
当系统开机之后，我们安装APP的时候，也会进dex2oat。
实质上这时候也是走到了PackageManagerService的流程。
由PMS执行安装的运行，并发起dex2oat的动作。
```
* 优化插件
 ```
当APP运行的时候，会根据自己的需要，优化自己需要的插件文件。
例如自己引用到的非系统类的jar包。
```
综上来看，系统执行dex2oat的主解有两类，一类是system server中的Packagemanagerservice发起的并让installd执行的;另一类是APP自身根据需要优化自己的插件。

## 8. 手机反应慢的原因
慢的原因如下
* 工作量
dex2oat发生的时候，会将原文件中的dex文件抽出来，逐个指令的判断，然后进行翻译，并生成大量的中间内容。并占用所有有CPU（目前的策略是有多少个核，就启动多少个线程）

* CPU
如果CPU越多，则启动的线程数越多，这样的话，就会比较快做完。

* EMMC
 如果dex文件比较大，会产生大量的中间内容，这些在memory当中是保存不下的，所以采用了swap机制，会将一些内容交换到EMMC当中，而且有大量的读操作，同时会将结果保存至emmc当中，所以emmc的性能也是非常关键的。

* Memory
如上面所述，memory越少，越容易发生交换，所以就会引起IO上的瓶颈。

* Patch
google之前有一个优化，就是在dex2oat做完的时候，不进行析构，因为析构会将交换出去的内容再读进来，从而导致耗时较长。
   1.M版本的patch为：  
     ALPS02736915
   2.N版本当中，暂时没有

## 9. 解决手机反应慢的方法
遇到慢的问题的时候，先从上面几个方面找原因，如果无法解决，那么就只好调整APP的安装策略。例如原来为speed模式，就改为interpreter-only模式。
参考修改方法如下：
  * 1. N版本(L /M 不适用) 在PMS中添加白名单
/frameworks/base/services/core/java/com/android/server/pm/PackageDexOptimizer.java

```
private int performDexOptLI(PackageParser.Package pkg, String[] sharedLibraries,String[] targetInstructionSets, boolean checkProfiles, String targetCompilerFilter) {


//在这函数中，可以判断传递下来的pkg是否是我们需要的package，如果是的话，将targetCompilerFilter设置为speed-profile。

//speed-profile会在安装的时候采用interperter-only，然后，运行一段时间之后，会将那些常用的方法优化成为speed模式。也就是说是有选择性的优化。

 if (isProfileGuidedFilter && isUsedByOtherApps(pkg)) {
 checkProfiles = false;

 targetCompilerFilter = getNonProfileGuidedCompilerFilter(targetCompilerFilter);
 if (DexFile.isProfileGuidedCompilerFilter(targetCompilerFilter)) {
 throw new IllegalStateException(targetCompilerFilter);
 }
 isProfileGuidedFilter = false;
}

// If we're asked to take profile updates into account, check now.
boolean newProfile = false;


###  mtk add begin
 if(pkg.contains("com.tencent.mm")){
 targetCompilerFilter="interpret-only";
 }
### mtk add end   

```
* 2.在Installd中进行白名单处理
适用于 L M N 版本
实现思路

```
//1.通过 property来增加白名单
//注意Facebook是apk的名字，后面不用带后缀，但是大小写一定要匹配。
adb shell setprop ro.mtk.dex2oat_white_list Facebook:

//2.如果需要增加到开机启动当中，可以如下修改在device.mk中加入
//多个APK名字之间用冒号隔开。
PRODUCT_PROPERTY_OVERRIDES += ro.mtk.dex2oat_white_list=facebook:Facebook:youtube:skype:twitter: 
```
修改点
 \frameworks\native\cmds\installd\commands.c
```
static int shouldUseInterpretonly(const char* input_file_name);

static void run_dex2oat(int zip_fd, int oat_fd, const char* input_file_name,
const char* output_file_name, int swap_fd, const char *pkgname, const char *instruction_set,
bool vm_safe_mode)
{

......

######## MTK add start
//适当的时候，进行调用

if(shouldUseInterpretonly(input_file_name))
{
strcpy(dex2oat_compiler_filter_arg, "--compiler-filter=interpret-only");
have_dex2oat_compiler_filter_flag = true;   //关键是设置这个参数的值
ALOGW("%s is in whitelist from property so set interpret-only",input_file_name);
}
######## MTK add end
ALOGV("Running %s in=%s out=%s\n", DEX2OAT_BIN, input_file_name, output_file_name);

.....

}
######MTK add start
/**
*函数功能，根据input的file name判断，当前来优化的APK名字判断是否在白名单当中。
*白名单来自于property
*/
static int shouldUseInterpretonly(const char* input_file_name)
{
char prop_buf[PROPERTY_VALUE_MAX];
memset(prop_buf,0,PROPERTY_VALUE_MAX);
bool have_whitelist = property_get("ro.mtk.dex2oat_white_list", prop_buf, NULL) > 0;
if(!have_whitelist)
return false;
char *str = prop_buf;
char appname[128],*ptrname = appname;
memset(appname,0,128);

while(*str)
{
if(*str != ':')
{
*ptrname = *str;
ptrname ++;
str++;
}
else{

str++;

if(*appname != 0)
{
if(strstr(input_file_name,appname)) //found
{
return 1; 
}
else
{
memset(appname,0,sizeof(appname));
ptrname = appname;
}
}
}
}
return 0;
}
### MTK add end
```
* 3 . 在dex2oat.cc 中进行白名单处理

/art/dex2oat/dex2oat.cc
 由于插件不会走PMS和installd这一路，所以，只能想办法在dex2oat中进行拦截。在这里面需要解决的问题是：如何判断来优化的就是我们关注的插件。

 * 通过判断dex2oat进程的父pid来判断是插件
如果是插件进行的dex2oat，有一个特点就是这个进程的父进程不是installd。所以可以根据一点进行判断。但是有一个特点，会将所有的插件全部优化成interpreter-only。

以N版本为例，建议的代码如下：
```
/*
 *调整编译的模式，规则：如果当前进程的父进程是installd，不做修改，否则修改为interpreter-only。
 *因为插件类的，都是进程本来fork的dex2oat.
 *only for device end
 *
 */
 void adjustCompilerOptions(const CompilerOptions* compiler_options){
 
 int ppid = getppid(); //获取当前进程的parent pid
 char cmdline_path[256];
 char contents[256];
 memset(cmdline_path,0,sizeof(cmdline_path));
 memset(contents,0,sizeof(contents));
 snprintf(cmdline_path,"/proc/%d/cmdline");
 FILE *fd = fopen(cmdline_path,"r"); // 打开proc/pid/cmdline文件，以读取对应ppid进程的名字
 
 if(fd == NULL){
 LOG(ERROR) < return;
 }
 
 fgets(contents,sizeof(contents),fd); //读取文件中的内容
 fclose(fd);
 if(strstr(contents,"installd")){ //判断是否是installd
 return;
 }
 
 Log(INFO)<<"parent is not installd,may be is a plugin adjust it to interpret-only";
 compiler_options->SetCompilerFilter(kInterpretOnly); //如果是的话，强行调整编译模式为interpreteronly模式
 
 
 }

在ParseArgs函数中对dex2oat的模式进行调整：
 ProcessOptions(parser_options.get());

 // Insert some compiler things.
 InsertCompileOptions(argc, argv);
 
 if(!IsHost())  //注意一定要判断非host端，否则会有问题
 adjustCompilerOptions(compiler_options_.get());
 
 }
```
* 根据dex文件名字或者oatlocation名字来判断

```
L版本：
/art/dex2oat/dex2oat.cc

方法1：

// mtk add begin

if ((!oat_filename_.empty() && (oat_filename_.find("facebook") != std::string::npos))||(!zip_location_.empty() && (zip_location_.find("facebook") !=std::string::npos))||((!oat_location_.empty()) && oat_location_.find("facebook") != std::string::npos)){  // mtk modify 2016-06-06

                                    compiler_filter_string = "interpret-only";

                                    LOG(INFO) <<" This apk is in whitelist from property so set interpret-only ";

            }

// mtk add end

if (compiler_filter_string = nullptr){

      compiler_filter_string = "speed";

}

 注意：CTS测项用的包不能使用interpret-only模式，因为一些CTS测项是性能相关的，如果改为interpret模式，会导致CTS不过。之前有客户为了追求更快的安装速度，将所有的包都改为interpret-only的，结果CTS就不过了，针对这种情况，可以把CTS的包再放到speed模式的白名单当中。cts的关键字为test.

 

方法2（若上述方法无效）：

1420 if (num_methods <= compiler_options_->GetNumDexMethodsThreshold()) {
1421 compiler_options_->SetCompilerFilter(CompilerOptions::kSpeed);
1422 VLOG(compiler) << "Below method threshold, compiling anyways";
1423 }
1424 }
1425
1426//mtk_add_begin
1427 static constexpr size_t kMinCompileDexSize = 4;
1428 if (!image_ && dex_files_.size() > kMinCompileDexSize) {
1429 compiler_options_->SetCompilerFilter(CompilerOptions::kInterpretOnly);
1430 LOG(INFO) << "Enable Whitelist Rules. Current Dex File Sizes:" << dex_files_.size();
1431 }
1432//mtk_add_end
1433
1434 return true;
1435 }
1436
1437 // Create and invoke the compiler driver. This will compile all the dex files.
1438 void Compile() {
1439 TimingLogger::ScopedTiming t("dex2oat Compile", timings_);
1440 compiler_phases_timings_.reset(new CumulativeLogger("compilation times"))
```
或者如下

```
[SOLUTION L版本]

    在device.mk中加入PRODUCT_PROPERTY_OVERRIDES += ro.mtk.dex2oat_white_list=com.tencent.mm: （注意 包名后又冒号“:”
 
[添加source code](抱歉不能排版)
 /art/dex2oat/dex2oat.cc添加红色部分：
 
 #ifdef HAVE_ANDROID_OS
 extern "C"{
                  static int shouldUseInterpretonly(const char* filename){
                       char prop_buf[92];
                       memset(prop_buf,0,92);
                       bool have_whitelist = property_get("ro.mtk.dex2oat_white_list", prop_buf, NULL) > 0;
                       if(!have_whitelist)
                       return false;
                       char *str = prop_buf;
                       char appname[128],*ptrname = appname;
                       memset(appname,0,128);

                       while(*str)
{
         if(*str != ':'){
                                     *ptrname = *str;
                                      ptrname ++;
                                      str++;
              } else{
                                       str++;
                                      if(*appname != 0){
                                                   if(strstr(filename,appname)) //found
                                                     {
                                                              return 1; 
                                                      } else{
                                                              memset(appname,0,sizeof(appname));
                                                              ptrname = appname;
                                                      }
                                         }
                         }
}

return 0;
}
}
#endif

static int dex2oat(int argc, char** argv) {

          std::string dex_filename; //mtk_add
           ......
         if (option.starts_with("--dex-file=")) {
                    dex_filenames.push_back(option.substr(strlen("--dex-file=")).data());
                    dex_filename = option.substr(strlen("--dex-file=")).data(); //mtk_add
         } else if......

         
         if (compiler_filter_string == nullptr) { 
         if (instruction_set == kMips64) {
         // TODO: fix compiler for Mips64.
                     compiler_filter_string = "interpret-only";
         } else if (image) {
                     compiler_filter_string = "speed";
         } else {
                    #if ART_SMALL_MODE
                    compiler_filter_string = "interpret-only";
        #else
          #ifdef HAVE_ANDROID_OS
                 if(shouldUseInterpretonly(zip_location.c_str())){
                              compiler_filter_string = "interpret-only";
                              LOG(INFO) <<" This apk is in whitelist from property so set interpret-only";
                  }else if(shouldUseInterpretonly(dex_filename.c_str())){
                              compiler_filter_string = "interpret-only";
                              LOG(INFO) <<" This jar is in whitelist from property so set interpret-only";
                   }else{
          #endif
                               compiler_filter_string = "speed";
           #ifdef HAVE_ANDROID_OS
                    } 
        #endif
#endif 
    }
}

CHECK(compiler_filter_string != nullptr);
.......

}
```


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

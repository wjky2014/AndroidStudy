 

##### 和您一起终身学习，这里是程序员Android


本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、启动流程概述
>二、Android启动分析
>三、init 进程启动分析
>四、init 启动脚本分析
>五、init 进程分析
>六、init 脚本执行
>七、init 进程守护
>八、init rc 脚本启动Zygote
>九、启动分析小结



# 一、 启动流程概述

`Android `启动流程跟 `Linux `启动类似，大致分为如下五个阶段。
- 1.开机上电，加载固化的`ROM`。
- 2.加载`BootLoader`，拉起`Android OS`。
- 3.加载`Uboot`，初始外设，引导`Kernel`启动等。
- 4.启动`Kernel`，加载驱动，硬件。
- 5.启动`Android`，挂载分区，加载驱动、服务，`init `进程等。

## 1.Android系统启动大致过程如下 
![Android 启动过程](https://upload-images.jianshu.io/upload_images/5851256-cbced23172ad9287.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于水平有限，无法深入了理解驱动层代码，本文主要对 `Android `上层启动流程进行分析。

# 二、Android启动分析

`Uboot`启动`Kernel`完成系统设置后，会首先在系统中寻找`init.rc`文件,并启动`init`进程。


![Android 启动分析](https://upload-images.jianshu.io/upload_images/5851256-51fed1d88c582420.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 三、init 进程启动分析

`Init`进程是`Android `启动的第一个进程，进程号为`1`，是`Android `的系统启动的核心进程，主要用来创建`Zygote`、属性服务等。 `init.cpp` 中的`main` 函数，是`init`进程的入口函数，源码主要存在`\system\core\init`目录下。

## 1.常见init.xxx.rc 进程 
![常见init.xxx.rc 进程](https://upload-images.jianshu.io/upload_images/5851256-15e275fc54ef3c90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 2.init 进程主要作用
![ init 进程主要作用](https://upload-images.jianshu.io/upload_images/5851256-b679ca4c063b6a2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



##3./system/core/init 部分内容如下：

![/system/core/init 部分内容](https://upload-images.jianshu.io/upload_images/5851256-c8f8cf9de9535acd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 4.main 函数主要做的事情 

1.创建挂载启动所需的文件系统`（tmpfs、 devpts、 proc、 sysfs、 selinuxfs等）`。
2.初始化并启动属性服务
3.解析`init.rc` 脚本配置文件，并启动`Zygote` 进程。

## 5.init.cpp main 函数实现代码 

```
int main(int argc, char** argv) {
   ... ...
    if (!strcmp(basename(argv[0]), "watchdogd")) {
		//启动看门狗函数
        return watchdogd_main(argc, argv);
    }
   ... ...

    //启动第一阶段
    if (is_first_stage) {
        boot_clock::time_point start_time = boot_clock::now();

        // 清理 umask.
        umask(0);

        clearenv();
        setenv("PATH", _PATH_DEFPATH, 1);

        // 在RAM内存上获取基本的文件系统，剩余的被 rc 文件所用
        mount("tmpfs", "/dev", "tmpfs", MS_NOSUID, "mode=0755");
        mkdir("/dev/pts", 0755);
        mkdir("/dev/socket", 0755);
        mount("devpts", "/dev/pts", "devpts", 0, NULL);
        #define MAKE_STR(x) __STRING(x)
        mount("proc", "/proc", "proc", 0, "hidepid=2,gid=" MAKE_STR(AID_READPROC));
        // 非特权应用不能使用 Android 命令行
        chmod("/proc/cmdline", 0440);
        gid_t groups[] = { AID_READPROC };
        setgroups(arraysize(groups), groups);
        mount("sysfs", "/sys", "sysfs", 0, NULL);
        mount("selinuxfs", "/sys/fs/selinux", "selinuxfs", 0, NULL);

        mknod("/dev/kmsg", S_IFCHR | 0600, makedev(1, 11));

        if constexpr (WORLD_WRITABLE_KMSG) {
            mknod("/dev/kmsg_debug", S_IFCHR | 0622, makedev(1, 11));
        }

        mknod("/dev/random", S_IFCHR | 0666, makedev(1, 8));
        mknod("/dev/urandom", S_IFCHR | 0666, makedev(1, 9));

        // Mount staging areas for devices managed by vold
        // See storage config details at http://source.android.com/devices/storage/
        mount("tmpfs", "/mnt", "tmpfs", MS_NOEXEC | MS_NOSUID | MS_NODEV,
              "mode=0755,uid=0,gid=1000");
        //创建可供读写的 vendor目录
        mkdir("/mnt/vendor", 0755);

        // 在/dev目录下挂载好 tmpfs 以及 kmsg 
        // 这样就可以初始化 /kernel Log 系统，供用户打印log
        InitKernelLogging(argv);

        LOG(INFO) << "init first stage started!";

        if (!DoFirstStageMount()) {
            LOG(FATAL) << "Failed to mount required partitions early ...";
        }

        SetInitAvbVersionInRecovery();

        // Enable seccomp if global boot option was passed (otherwise it is enabled in zygote).
        global_seccomp();

        // 优先加载selinux log系统， 紧接着初始化selinux
        SelinuxSetupKernelLogging();
        SelinuxInitialize();

        // 添加 selinux 是否启动成功的log
        if (selinux_android_restorecon("/init", 0) == -1) {
            PLOG(FATAL) << "restorecon failed of /init failed";
        }

        setenv("INIT_SECOND_STAGE", "true", 1);

        static constexpr uint32_t kNanosecondsPerMillisecond = 1e6;
        uint64_t start_ms = start_time.time_since_epoch().count() / kNanosecondsPerMillisecond;
        setenv("INIT_STARTED_AT", std::to_string(start_ms).c_str(), 1);

        char* path = argv[0];
        char* args[] = { path, nullptr };
        execv(path, args);

        // execv() only returns if an error happened, in which case we
        // panic and never fall through this conditional.
        PLOG(FATAL) << "execv(\"" << path << "\") failed";
    }

    //启动第二阶段
    InitKernelLogging(argv);
    LOG(INFO) << "init second stage started!";

    // Set up a session keyring that all processes will have access to. It
    // will hold things like FBE encryption keys. No process should override
    // its session keyring.
    keyctl_get_keyring_ID(KEY_SPEC_SESSION_KEYRING, 1);

    // Indicate that booting is in progress to background fw loaders, etc.
    close(open("/dev/.booting", O_WRONLY | O_CREAT | O_CLOEXEC, 0000));
    //初始化属性
    property_init();

    // If arguments are passed both on the command line and in DT,
    // properties set in DT always have priority over the command-line ones.
    process_kernel_dt();
    process_kernel_cmdline();

    // Propagate the kernel variables to internal variables
    // used by init as well as the current required properties.
    export_kernel_boot_props();

    // Make the time that init started available for bootstat to log.
    property_set("ro.boottime.init", getenv("INIT_STARTED_AT"));
    property_set("ro.boottime.init.selinux", getenv("INIT_SELINUX_TOOK"));

    // Set libavb version for Framework-only OTA match in Treble build.
    const char* avb_version = getenv("INIT_AVB_VERSION");
    if (avb_version) property_set("ro.boot.avb_version", avb_version);

    // 清空设置的环境变量
    unsetenv("INIT_SECOND_STAGE");
    unsetenv("INIT_STARTED_AT");
    unsetenv("INIT_SELINUX_TOOK");
    unsetenv("INIT_AVB_VERSION");

    // 设置第二阶段的selinux
    SelinuxSetupKernelLogging();
    SelabelInitialize();
    SelinuxRestoreContext();
    //创建 epoll 句柄
    epoll_fd = epoll_create1(EPOLL_CLOEXEC);
    if (epoll_fd == -1) {
        PLOG(FATAL) << "epoll_create1 failed";
    }
    //设置 子进程处理函数
    sigchld_handler_init();

    if (!IsRebootCapable()) {
        // If init does not have the CAP_SYS_BOOT capability, it is running in a container.
        // In that case, receiving SIGTERM will cause the system to shut down.
        InstallSigtermHandler();
    }

    LoadRscRoProps();
    property_load_boot_defaults();
    export_oem_lock_status();
	//启动属性服务
    start_property_service();
	//为USB存储设置udc Contorller, sys/class/udc
    set_usb_controller();

    const BuiltinFunctionMap function_map;
    Action::set_function_map(&function_map);

    subcontexts = InitializeSubcontexts();

    ActionManager& am = ActionManager::GetInstance();
    ServiceList& sm = ServiceList::GetInstance();

    LoadBootScripts(am, sm);

    // Turning this on and letting the INFO logging be discarded adds 0.2s to
    // Nexus 9 boot time, so it's disabled by default.
    if (false) DumpState();

    am.QueueEventTrigger("early-init");

    // Queue an action that waits for coldboot done so we know ueventd has set up all of /dev...
    am.QueueBuiltinAction(wait_for_coldboot_done_action, "wait_for_coldboot_done");
    // ... so that we can start queuing up actions that require stuff from /dev.
    am.QueueBuiltinAction(MixHwrngIntoLinuxRngAction, "MixHwrngIntoLinuxRng");
    am.QueueBuiltinAction(SetMmapRndBitsAction, "SetMmapRndBits");
    am.QueueBuiltinAction(SetKptrRestrictAction, "SetKptrRestrict");
    am.QueueBuiltinAction(keychord_init_action, "keychord_init");
    am.QueueBuiltinAction(console_init_action, "console_init");

    // Trigger all the boot actions to get us started.
    am.QueueEventTrigger("init");

    // Repeat mix_hwrng_into_linux_rng in case /dev/hw_random or /dev/random
    // wasn't ready immediately after wait_for_coldboot_done
    am.QueueBuiltinAction(MixHwrngIntoLinuxRngAction, "MixHwrngIntoLinuxRng");

    // Don't mount filesystems or start core system services in charger mode.
    std::string bootmode = GetProperty("ro.bootmode", "");
    if (bootmode == "charger") {
        am.QueueEventTrigger("charger");
    } else {
        am.QueueEventTrigger("late-init");
    }

    // Run all property triggers based on current state of the properties.
    am.QueueBuiltinAction(queue_property_triggers_action, "queue_property_triggers");

    while (true) {
        // By default, sleep until something happens.
        int epoll_timeout_ms = -1;

        if (do_shutdown && !shutting_down) {
            do_shutdown = false;
            if (HandlePowerctlMessage(shutdown_command)) {
                shutting_down = true;
            }
        }

        if (!(waiting_for_prop || Service::is_exec_service_running())) {
            am.ExecuteOneCommand();
        }
        if (!(waiting_for_prop || Service::is_exec_service_running())) {
            if (!shutting_down) {
                auto next_process_restart_time = RestartProcesses();

                // If there's a process that needs restarting, wake up in time for that.
                if (next_process_restart_time) {
                    epoll_timeout_ms = std::chrono::ceil<std::chrono::milliseconds>(
                                           *next_process_restart_time - boot_clock::now())
                                           .count();
                    if (epoll_timeout_ms < 0) epoll_timeout_ms = 0;
                }
            }

            // If there's more work to do, wake up again immediately.
            if (am.HasMoreCommands()) epoll_timeout_ms = 0;
        }

        epoll_event ev;
        int nr = TEMP_FAILURE_RETRY(epoll_wait(epoll_fd, &ev, 1, epoll_timeout_ms));
        if (nr == -1) {
            PLOG(ERROR) << "epoll_wait failed";
        } else if (nr == 1) {
            ((void (*)()) ev.data.ptr)();
        }
    }

    return 0;
}

```


## 6.基于MTK 平台 init.cpp 源码分析 

![基于MTK 平台 init.cpp 主要作用](https://upload-images.jianshu.io/upload_images/5851256-e809ae8f56e921c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 四、 init 启动脚本分析
`init.rc` 路径 一般在`system/core/rootdir`下，`init `脚本是有`Android` 初始化语言编写。

##1.Android Init Language 语句类型 
- 1.Action
- 2.Command
- 3.Service
- 4.Option
- 5.Import



![init 进程分析](https://upload-images.jianshu.io/upload_images/5851256-9abcf42315db687b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![init.rc on](https://upload-images.jianshu.io/upload_images/5851256-bfba7c18cee4b51d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![init.rc services](https://upload-images.jianshu.io/upload_images/5851256-44b8811c44b0b7e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![init.rc import ](https://upload-images.jianshu.io/upload_images/5851256-b76da61ed79e17d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 五、init 进程分析

![init 进程分析](https://upload-images.jianshu.io/upload_images/5851256-ae11460fd6cd0b45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![init 解析脚本分析](https://upload-images.jianshu.io/upload_images/5851256-34952dccd178a424.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![init 事件列表](https://upload-images.jianshu.io/upload_images/5851256-6f447164a0aecda3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![init 事件结构](https://upload-images.jianshu.io/upload_images/5851256-18cbe200bf712604.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 六 、init 脚本执行

![init 进程解析和执行](https://upload-images.jianshu.io/upload_images/5851256-2400fd9bfb039c58.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![启动脚本解析结果](https://upload-images.jianshu.io/upload_images/5851256-301292896aad83d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![整理事件列表](https://upload-images.jianshu.io/upload_images/5851256-1a29520ad360dc51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![init 构建事件](https://upload-images.jianshu.io/upload_images/5851256-b91809c8b63ea552.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Service 事件分类](https://upload-images.jianshu.io/upload_images/5851256-29d8ad5682f0bc03.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![init 进程执行命令和启动服务](https://upload-images.jianshu.io/upload_images/5851256-d46421c8e16b01b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 七、init 进程守护

`init`进程处理消息事件
- 1. 根据`Shell `或者系统中消息设置系统`prop`
- 2. 守护系统服务，如果服务退出，重启退出的服务。

![init守护进程](https://upload-images.jianshu.io/upload_images/5851256-e5620d8dce32caa4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![init 处理 prop 消息分析](https://upload-images.jianshu.io/upload_images/5851256-ee110ae64550fa2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![init 守护服务分析](https://upload-images.jianshu.io/upload_images/5851256-af78d7e03e6f5112.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 八、init rc 脚本启动Zygote

`Zygote` 的 `classname` 为`main `.
`init.rc `文件配置代码如下：
```
... ... 
on nonencrypted
    class_start main
    class_start late_start

on property:sys.init_log_level=*
    loglevel ${sys.init_log_level}

... ...
```
# 九、启动分析小结


![启动分析小结](https://upload-images.jianshu.io/upload_images/5851256-a5cbd638d73879a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

xmind 小结文件下载地址如下：
[小结源文件下载地址](https://pan.baidu.com/s/1-RJmRR3JwRI32HuYZbpRzg)




**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

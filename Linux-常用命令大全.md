


##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、cd 命令
>二、--help 命令
>三、 ls 命令
>四、touch 命令
>五、mkdir命令
>六、pwd 命令
>七、echo 命令
>八、cat 命令
>九、Tab 键
>十、vi 或者vim 命令
>十一、rm 命令
>十二、mv 命令
>十三、cp 命令
>十四、find 命令
>十五、grep 命令
>十六、chmod 命令
>十七、压缩与解压命令
>十八、Top命令
>十九、在某一类文件中(比如：*.xml）查找字符串 
>二十、 字符串替换命令
>二十一、Tree 命令
> 二十二、显示目录下的所有文件
> 二十三、统计目录下文件个数

# 一、cd  命令 

`cd ` 命令是 `Linux`常用命令之一，主要用于进入目录`（相当于windows的文件夹）`。
比如我想进入`TestLinuxCommond`这个目录。
可以使用`cd TestLinuxCommond/`
```
wangjie@ubuntu:~$ cd TestLinuxCommond/
wangjie@ubuntu:~/TestLinuxCommond$ 
```
比如我想退出`TestLinuxCommond`这个目录。
可以使用`cd ..`
```
wangjie@ubuntu:~/TestLinuxCommond$ cd ..
wangjie@ubuntu:~$ 
```
如果想退两层目录，可以使用`cd ../..`
```
wangjie@ubuntu:~/TestLinuxCommond/testmv/test$ ls
wangjie@ubuntu:~/TestLinuxCommond/testmv/test$ cd ../..
wangjie@ubuntu:~/TestLinuxCommond$ 

```
# 二、--help 命令
当我们不知道一个命令如何使用时候，可以使用 `--help` 查看这个命令是干嘛的。
`--help`相当于用户命令帮忙的手册。
比如 我想看看 `cd`命令是如何使用的，可以使用`cd ---help`
```
wangjie@ubuntu:~/TestLinuxCommond$ cd ---help
-bash: cd: --: invalid option
cd: usage: cd [-L|[-P [-e]] [-@]] [dir]
wangjie@ubuntu:~/TestLinuxCommond$ 
```
比如 我想看看 `ls`命令是如何使用的，使用参数如何，可以使用`ls ---help`
```
wangjie@ubuntu:~/TestLinuxCommond$ ls --help
Usage: ls [OPTION]... [FILE]...
List information about the FILEs (the current directory by default).
Sort entries alphabetically if none of -cftuvSUX nor --sort is specified.

Mandatory arguments to long options are mandatory for short options too.
  -a, --all                  do not ignore entries starting with .
  -A, --almost-all           do not list implied . and ..
      --author               with -l, print the author of each file
  -b, --escape               print C-style escapes for nongraphic characters
      --block-size=SIZE      scale sizes by SIZE before printing them.  E.g.,
                               '--block-size=M' prints sizes in units of
                               1,048,576 bytes.  See SIZE format below.
  -B, --ignore-backups       do not list implied entries ending with ~
  -c                         with -lt: sort by, and show, ctime (time of last
                               modification of file status information)
                               with -l: show ctime and sort by name
                               otherwise: sort by ctime, newest first
  -C                         list entries by columns
      --color[=WHEN]         colorize the output.  WHEN defaults to 'always'
                               or can be 'never' or 'auto'.  More info below
  -d, --directory            list directory entries instead of contents,
                               and do not dereference symbolic links
  -D, --dired                generate output designed for Emacs' dired mode
  -f                         do not sort, enable -aU, disable -ls --color
  -F, --classify             append indicator (one of */=>@|) to entries
      --file-type            likewise, except do not append '*'
      --format=WORD          across -x, commas -m, horizontal -x, long -l,
                               single-column -1, verbose -l, vertical -C
      --full-time            like -l --time-style=full-iso
  -g                         like -l, but do not list owner
      --group-directories-first
                             group directories before files.
                               augment with a --sort option, but any
                               use of --sort=none (-U) disables grouping
  -G, --no-group             in a long listing, don't print group names
  -h, --human-readable       with -l, print sizes in human readable format
                               (e.g., 1K 234M 2G)
      --si                   likewise, but use powers of 1000 not 1024
  -H, --dereference-command-line
                             follow symbolic links listed on the command line
      --dereference-command-line-symlink-to-dir
                             follow each command line symbolic link
                             that points to a directory
      --hide=PATTERN         do not list implied entries matching shell PATTERN
                               (overridden by -a or -A)
      --indicator-style=WORD  append indicator with style WORD to entry names:
                               none (default), slash (-p),
                               file-type (--file-type), classify (-F)
  -i, --inode                print the index number of each file
  -I, --ignore=PATTERN       do not list implied entries matching shell PATTERN
  -k, --kibibytes            use 1024-byte blocks
  -l                         use a long listing format
  -L, --dereference          when showing file information for a symbolic
                               link, show information for the file the link
                               references rather than for the link itself
  -m                         fill width with a comma separated list of entries
  -n, --numeric-uid-gid      like -l, but list numeric user and group IDs
  -N, --literal              print raw entry names (don't treat e.g. control
                               characters specially)
  -o                         like -l, but do not list group information
  -p, --indicator-style=slash
                             append / indicator to directories
  -q, --hide-control-chars   print ? instead of non graphic characters
      --show-control-chars   show non graphic characters as-is (default
                             unless program is 'ls' and output is a terminal)
  -Q, --quote-name           enclose entry names in double quotes
      --quoting-style=WORD   use quoting style WORD for entry names:
                               literal, locale, shell, shell-always, c, escape
  -r, --reverse              reverse order while sorting
  -R, --recursive            list subdirectories recursively
  -s, --size                 print the allocated size of each file, in blocks
  -S                         sort by file size
      --sort=WORD            sort by WORD instead of name: none -U,
                             extension -X, size -S, time -t, version -v
      --time=WORD            with -l, show time as WORD instead of modification
                             time: atime -u, access -u, use -u, ctime -c,
                             or status -c; use specified time as sort key
                             if --sort=time
      --time-style=STYLE     with -l, show times using style STYLE:
                             full-iso, long-iso, iso, locale, +FORMAT.
                             FORMAT is interpreted like 'date'; if FORMAT is
                             FORMAT1<newline>FORMAT2, FORMAT1 applies to
                             non-recent files and FORMAT2 to recent files;
                             if STYLE is prefixed with 'posix-', STYLE
                             takes effect only outside the POSIX locale
  -t                         sort by modification time, newest first
  -T, --tabsize=COLS         assume tab stops at each COLS instead of 8
  -u                         with -lt: sort by, and show, access time
                               with -l: show access time and sort by name
                               otherwise: sort by access time
  -U                         do not sort; list entries in directory order
  -v                         natural sort of (version) numbers within text
  -w, --width=COLS           assume screen width instead of current value
  -x                         list entries by lines instead of by columns
  -X                         sort alphabetically by entry extension
  -Z, --context              print any SELinux security context of each file
  -1                         list one file per line
      --help     display this help and exit
      --version  output version information and exit

SIZE is an integer and optional unit (example: 10M is 10*1024*1024).  Units
are K, M, G, T, P, E, Z, Y (powers of 1024) or KB, MB, ... (powers of 1000).

Using color to distinguish file types is disabled both by default and
with --color=never.  With --color=auto, ls emits color codes only when
standard output is connected to a terminal.  The LS_COLORS environment
variable can change the settings.  Use the dircolors command to set it.

Exit status:
 0  if OK,
 1  if minor problems (e.g., cannot access subdirectory),
 2  if serious trouble (e.g., cannot access command-line argument).

Report ls bugs to bug-coreutils@gnu.org
GNU coreutils home page: <http://www.gnu.org/software/coreutils/>
General help using GNU software: <http://www.gnu.org/gethelp/>
For complete documentation, run: info coreutils 'ls invocation'
wangjie@ubuntu:~/TestLinuxCommond$ 

```
#  三、 ls 命令

`ls`用来显示当前目录下都有哪些文件、或者目录`（Window下叫文件夹）`。
备注：
使用 `ls -l` 可以将当前目录下的所有内容以列表形式显示，同时并显示文件的详细信息。
比如我想看看当前目录下有什么文件，可以使用`ls`。
```
wangjie@ubuntu:~/TestLinuxCommond$ ls
test.txt
wangjie@ubuntu:~/TestLinuxCommond$
```
比如我想看看当前目录下文件的详细信息，可以使用`ls -l`
```
wangjie@ubuntu:~/TestLinuxCommond$ ls -l
total 0
-rw-rw-r-- 1 wangjie wangjie 0 Jul 18 16:24 test.txt
wangjie@ubuntu:~/TestLinuxCommond$ 
```
# 四、touch 命令

`touch` 用来创建文本文件，可以有后缀，也可以没有，完全看个人喜好。
比如 我想创建一个`testlinux.txt`文件,可以使用`touch testlinux.txt`
```
wangjie@ubuntu:~/TestLinuxCommond$ touch testlinux.txt
wangjie@ubuntu:~/TestLinuxCommond$ ls
testlinux.txt  test.txt
wangjie@ubuntu:~/TestLinuxCommond$ 
```
# 五、mkdir 命令
`mkdir` 用来创建目录，`linux`下的目录相当于`Windows `下的文件夹。
比如我想创建一个`test`目录，可以使用` mkdir test`
```
wangjie@ubuntu:~/TestLinuxCommond$ mkdir test
wangjie@ubuntu:~/TestLinuxCommond$ ls
test  testlinux.txt  test.txt
wangjie@ubuntu:~/TestLinuxCommond$ 

```

# 六、pwd 命令
`pwd` 用来显示 当前所在目录的路径。
比如我想看看我当前目录所在的路径是什么,可以使用`pwd`
```
wangjie@ubuntu:~/TestLinuxCommond$ pwd
/home/wangjie/TestLinuxCommond
wangjie@ubuntu:~/TestLinuxCommond$ 

```

# 七、echo 命令

`echo`命令 主要用来向文本文件中追加内容。
比如我要将`hello Linux` 写入到`testlinux.txt `文件中,可以使用`echo "hello Linux" > testlinux.txt `
```
wangjie@ubuntu:~/TestLinuxCommond$ echo "hello Linux" > testlinux.txt 
wangjie@ubuntu:~/TestLinuxCommond$ cat testlinux.txt 
hello Linux
wangjie@ubuntu:~/TestLinuxCommond$ 
```
# 八、cat 命令
`cat`命令主要用来显示文本文件中的内容。
比如我要显示 `testlinux.txt` 文本中的内容,可以使用`cat testlinux.txt`
```
wangjie@ubuntu:~/TestLinuxCommond$ cat testlinux.txt 
hello Linux
wangjie@ubuntu:~/TestLinuxCommond$ 

```
# 九、Tab 键
`Tab 键` 在Linux 下主要用来辅助输入，快速补全的功能。 
比如我们想看当前目录下以test 开头的目录或者文件时，但是我们有忘记文件或者目录的全称，此时可在输入开头后按下`Tab键`，系统就会列出以`test`开头的所有文件或者目录。
举例如下：
```
wangjie@ubuntu:~/TestLinuxCommond$ ls test
test/          testlinux.txt  test.txt       
wangjie@ubuntu:~/TestLinuxCommond$ 

```
# 十、vi 或者vim 命令
`vi`或者`vim`命令，主要是通过`vim`编辑器对 文本文件进行编辑。
比如我们想编辑 `testlinux.txt`这个文件，此时可以使用 `vim testlinux.txt`或者`vim testlinux.txt`。
输入命令后需要 按`i` 或者 `a `对文本文件进行输入操作。
#### 1.按`i` 或者 `a `对文本文件进程插入操作
![](https://upload-images.jianshu.io/upload_images/5851256-fd59825ef04a75f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 2.按 Esc 键，输入`：(冒号)`保存文件

![](https://upload-images.jianshu.io/upload_images/5851256-a07923358a288a0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

命令|意义
----|----
`w `|保存 
`q`|退出
`!`|强制
`wq`|保存并退出
`q!`|放弃修改，强制退出

#### 3.set nu 显示行号 set nonu 取消显示行号
使用格式如下：
`set nu `或者` set nonu`
![set nu](https://upload-images.jianshu.io/upload_images/5851256-69be70add29a434f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 4.`/查找的字符串` 用来查找字符串
使用格式如下：
`/查找的字符串` 
比如查找`e`,此时 按`N键`可以全局切换查找上下字符串。
![/查找的字符串](https://upload-images.jianshu.io/upload_images/5851256-87016555c175c608.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 十一、rm 命令
`rm`命令主要用来执行删除文件或者目录的操作，注意两个是有区别的。
##### 删除文件
使用格式如下：`rm 文件名`
比如我想删除`test.txt`,可以使用 `rm text.txt`
```
wangjie@ubuntu:~/TestLinuxCommond$ ls
test  testlinux.txt  test.txt
wangjie@ubuntu:~/TestLinuxCommond$ rm test.txt 
wangjie@ubuntu:~/TestLinuxCommond$ ls
test  testlinux.txt
wangjie@ubuntu:~/TestLinuxCommond$ 

```
##### 删除目录，
使用格式如下：
`rm  -r 文件名`
删除目录时候需要添加参数 `-r`,这样主要是用来递归删除目录下的所有内容（包含 目录、文件等）。
比如我想删除`test`目录
```
wangjie@ubuntu:~/TestLinuxCommond$ ls
test  testlinux.txt
wangjie@ubuntu:~/TestLinuxCommond$ rm test
rm: cannot remove ‘test’: Is a directory
wangjie@ubuntu:~/TestLinuxCommond$ ls
test  testlinux.txt
wangjie@ubuntu:~/TestLinuxCommond$ rm -r test
wangjie@ubuntu:~/TestLinuxCommond$ ls
testlinux.txt
wangjie@ubuntu:~/TestLinuxCommond$ 
```
# 十二、mv 命令
`mv` 主要用来移动、或者重命名文件。
#### 重命名
使用格式如下：
`mv 老名字文件 新名字`
假如我想将`testlinux.txt` 重命名为`test.txt`,可以使用 ` mv testlinux.txt  test.txt`
举例如下：
```
wangjie@ubuntu:~/TestLinuxCommond$ mkdir testmv
wangjie@ubuntu:~/TestLinuxCommond$ ls
testlinux.txt  testmv
wangjie@ubuntu:~/TestLinuxCommond$ mv testlinux.txt  test.txt
wangjie@ubuntu:~/TestLinuxCommond$ ls
testmv  test.txt
wangjie@ubuntu:~/TestLinuxCommond$ 

```

#### 移动
使用格式如下：
`mv 要移动的文件  要移动文件的目的地`
比如我想将当前目录下的`test.txt `移动到`testmv `目录下，可以使用`mv test.txt testmv/`
```
wangjie@ubuntu:~/TestLinuxCommond$ ls
testmv  test.txt
wangjie@ubuntu:~/TestLinuxCommond$ mv test.txt testmv/
wangjie@ubuntu:~/TestLinuxCommond$ ls
testmv
wangjie@ubuntu:~/TestLinuxCommond$ cd testmv/
wangjie@ubuntu:~/TestLinuxCommond/testmv$ ls
test  test.txt
wangjie@ubuntu:~/TestLinuxCommond/testmv$ 

```
# 十三、cp 命令
使用格式如下：
`mv  要复制的文件  要复制文件的目的地`
`cp` 命令用来复制文件。
比如我想复制text.txt 到test 目录下，可以使用 `cp test.txt test`
```
wangjie@ubuntu:~/TestLinuxCommond/testmv$ ls
test  test.txt
wangjie@ubuntu:~/TestLinuxCommond/testmv$ cp test.txt test
wangjie@ubuntu:~/TestLinuxCommond/testmv$ cd test/
wangjie@ubuntu:~/TestLinuxCommond/testmv/test$ ls
test.txt
wangjie@ubuntu:~/TestLinuxCommond/testmv/test$ 
```
复制并包含文件详细目录的方法
```
wangjie@wangjie:$  cp --parents -av   tools/releasetools/build_image.py   /wangjie/log/ 
```

# 十四、find 命令
`find` 用来查找搜索文件。
比如我要查找当前目录下名为`test.txt`的文件
可以使用`find . -name test.txt` 命令查找。
举例如下：
```
wangjie@ubuntu:~/TestLinuxCommond$ find . -name test.txt
./testmv/test.txt
./testmv/test/test.txt
wangjie@ubuntu:~/TestLinuxCommond$ 
```
# 十五、grep 命名
`grep` 命令用来查找文件中的字符串资源。
#####区分大小写
比如我想查找字符串 `Sprocomm `，可以使用`grep "Sprocomm" -r .`
此时的查找是按照大小写严格意思上查找
```
wangjie@ubuntu:~/TestLinuxCommond$ grep "Sprocomm" -r .
./testmv/test.txt:Welcome to Sprocomm
./testmv/test/test.txt:Welcome to Sprocomm
wangjie@ubuntu:~/TestLinuxCommond$ 

```
##### 不区分大小写
 添加 `-i` ,`ingore` 忽略大小写。
 比如我想查找字符串 `Sprocomm `，可以使用`grep "Sprocomm" -ri .`
```
wangjie@ubuntu:~/TestLinuxCommond$ grep "sprocomm" -r .
wangjie@ubuntu:~/TestLinuxCommond$ grep "sprocomm" -ri .
./testmv/test.txt:Welcome to Sprocomm
./testmv/test/test.txt:Welcome to Sprocomm

```
#十六、chmod 命令
`chmod `命令 主要用来修改目录或文件权限。

```
wangjie@ubuntu:~/TestLinuxCommond$ ls -l
total 4
drwxrwxr-x 3 wangjie wangjie 4096 Jul 18 17:56 testmv
wangjie@ubuntu:~/TestLinuxCommond$ 
```

名称|简称|代表意义|数值
----|----|----|----
`d`|dir|目录|-
`r`|read|读文件权限|4
`w`|write|写文件权限|2
`x`|-|可执行权限`（通常是以下sh脚本，库文件等）`|1

加入我想改变 `testmv`下所有文件的`读、写执行`权限。
可以使用`chmod 777 -R .`
```
wangjie@ubuntu:~/TestLinuxCommond$ ls -l
total 4
drwxrwxr-x 3 wangjie wangjie 4096 Jul 18 17:56 testmv
wangjie@ubuntu:~/TestLinuxCommond$ chmod 777 -R .
wangjie@ubuntu:~/TestLinuxCommond$ ls -l
total 4
drwxrwxrwx 3 wangjie wangjie 4096 Jul 18 17:56 testmv
wangjie@ubuntu:~/TestLinuxCommond$ 

```
# 十七、压缩与解压命令

##### tar 压缩解压命令
压缩命令格式 ：
`tar -cvf *.tar  要压缩的文件`
比如我想压缩一个`testmv.tar` ,可以使用 `tar -cvf test.tar testmv/`
```
wangjie@ubuntu:~/TestLinuxCommond$ tar -cvf test.tar testmv/
testmv/
testmv/test.txt
testmv/test/
testmv/test/test.txt
wangjie@ubuntu:~/TestLinuxCommond$ ls
testmv  test.tar
wangjie@ubuntu:~/TestLinuxCommond$ 

```
解压命令格式
` tar -xvf test.tar `
比如我想解压缩一个`test.tar` ,可以使用 ` tar -xvf test.tar `

```
wangjie@ubuntu:~/TestLinuxCommond$ rm -rf testmv/
wangjie@ubuntu:~/TestLinuxCommond$ ls
test.tar
wangjie@ubuntu:~/TestLinuxCommond$ tar -xvf test.tar  
testmv/
testmv/test.txt
testmv/test/
testmv/test/test.txt
wangjie@ubuntu:~/TestLinuxCommond$ ls
testmv  test.tar
wangjie@ubuntu:~/TestLinuxCommond$ 

```
#### zip 压缩命令

```
wangjie@wangjie:/wangjie/log$ zip  -r test.zip test.bat 
  adding: test.bat (stored 0%)
wangjie@wangjie:/wangjie/log$ 

```

# 十八、Top 命令

Top 命令使用方法
```
adb shell top -n 30 -m 10  -k -%CPU > top1.txt
```
# 十九、在某一类文件中(比如：*.xml）查找字符串 

```
find . -type f -name "*.xml" | xargs grep "xxx"
```
# 二十、字符串替换命令
在某个目录下替换说有的字符串的命令

比如 将 WLAN 替换为WiFi
```
sed -i 's/WLAN/WiFi/g' `grep WLAN . -rl`
```
#二十一、Tree 命令

```
E:\AOSP\android_10_0_0_r40\frameworks\av\camera>tree
卷 文档 的文件夹 PATH 列表
卷序列号为 00000233 65F3:3762
E:.
├─aidl
│  └─android
│      └─hardware
│          └─camera2
│              ├─impl
│              ├─params
│              └─utils
├─camera2
├─cameraserver
├─include
│  └─camera
│      ├─android
│      │  └─hardware
│      └─camera2
├─ndk
│  ├─impl
│  ├─include
│  │  └─camera
│  └─ndk_vendor
│      ├─impl
│      └─tests
└─tests

E:\AOSP\android_10_0_0_r40\frameworks\av\camera>
```
# 二十二、显示目录下的所有文件
```
find -type f -print
```
使用举例：
```
$ find -type f -print
./Android.bp
./Barrier.h
./BufferLayer.cpp
./BufferLayer.h
./BufferLayerConsumer.cpp
./BufferLayerConsumer.h
./BufferQueueLayer.cpp
./BufferQueueLayer.h
./BufferStateLayer.cpp
./BufferStateLayer.h

```
# 二十三、统计目录下文件个数

命令如下
` ls -lR |grep '^-' |wc -l`
使用举例：
```
/e/AOSP/android_10_0_0_r40/frameworks/native/services/surfaceflinger (master)
$ ls -lR |grep '^-' |wc -l
294

```


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


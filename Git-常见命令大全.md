# 1.Git 概念
 分布式版本控制系统
# 2.Git文件三种状态
* 1.committed(已提交)
    该文件已经被安全的保存在本地数据库中
* 2.modified（已修改）
    已修改了某个文件，但是还没提交保存
* 3.staged（已暂存）
    把已修改的文件放在下次提交时要保存的清单中

# 3.Git 的三个工作目录
* 1.工作目录
   从项目中取出某个版本的所有文件和目录，用以开始后续工作的
* 2.暂存区域
   只不过是个简单的文件，一般都放在 Git 目录中。有时候人们会把这个文件叫做索引文件，不过标准说法还是叫暂存区域。
* 3.git 仓库
    远程服务器上大家共同维护的版本存放库

# 4.Git 常用方法

* 1.初始化Git 仓库
```
git init
```
* 2.克隆 git 服务器上的代码
```
语法如下：
git clone [url]
 ``` 
[url] 支持：git:// 协议，http(s):// 或者 user@server:/path.git 

 ```
//example 1
git clone   https://github.com/wjky2014/spromcomm.git
//example 2
// 克隆代码并创建CreateFileName目录
git clone   https://github.com/wjky2014/spromcomm.git  CreateFileName
```
* 3.查看工作目录下文件的状态
```
git status
```
```
//example
wangjie@ubuntu:~/testgit/spromcomm$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
nothing to commit, working directory clean
wangjie@ubuntu:~/testgit/spromcomm$ 
```

* 4.跟踪新文件
```
git add  文件/目录
```
```
//example
wangjie@ubuntu:~/testgit/spromcomm$ git add addtest.txt 
wangjie@ubuntu:~/testgit/spromcomm$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
	new file:   addtest.txt
wangjie@ubuntu:~/testgit/spromcomm$ 
```


 * 5.撤销添加跟踪的新文件

```
git reset HEAD <file>
```

* 6.提交修改文件到暂存区

```
git commit -m "修改记录信息"
```
* 7.查看修改点差异

```
//工作目录中的文件与暂存区文件之间的差异，即修改之后还没有暂存的内容
git diff 
//已经暂存的文件与上次提交时候的快照之间的差异 
//这两个命令暂未搞懂
git diff --staged 
git diff --cached

```
* 8.放弃修改，回复原来样子
```
git checkout <file>
```
* 9.提交更新

提交更新之前要执行一下git status 查看一下是否所有的修改都git add上

```
git commit 
or
git commit -m "提交记录信息"
or
git commit -a -m "提交记录信息"   = git add + git commit 

//撤销刚才的提交操作
git commit --amend 
git commit -m 'initial commit'
git add forgotten_file
git commit --amend
//上面的三条命令最终只是产生一个提交，第二个提交命令修正了第一个的提交内容。
```
* 10.移除文件

```
git  rm  
//如果当前文件已经修改且存在暂存区
git  rm -f 
```
* 11.重命名文件

```
git mv file_from file_to
//     git mv = 
               mv file_from file_to 
               + git rm    file_from 
               + git add  file_to
```
* 12.查看提交记录
```
git log
```

* 13.查看当前远程库

```
git remote 
git remote -v (verbose)
//example
wangjie@ubuntu:~/testgit/spromcomm$ git remote
origin
wangjie@ubuntu:~/testgit/spromcomm$ git remote -v
origin	https://github.com/wjky2014/spromcomm.git (fetch)
origin	https://github.com/wjky2014/spromcomm.git (push)
wangjie@ubuntu:~/testgit/spromcomm$ 
```
* 14.更新工作目录与服务器同步
```
git pull
```
* 15 .将修改同步到服务器
```
git push 
```
* 16 .新建标签
1.打含附注的标签
```
git tag -a <标签名> -m  " 提交记录"
//example 
wangjie@ubuntu:~/testgit/spromcomm$ git tag -a v1.0 -m " eg: git tag -a "
wangjie@ubuntu:~/testgit/spromcomm$ git tag
v1.0
wangjie@ubuntu:~/testgit/spromcomm$ 
```
* 17.分享标签
默认情况下git push 并不会把标签传送到远端服务器上
```
git push origin [tagname]
//example 
```

- 18.打包某次commit 修改点的方法

使用命令如下：
```
git format-patch  commit_id -1
```

- 19.git 缓存修改到本地的方法
```
```



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

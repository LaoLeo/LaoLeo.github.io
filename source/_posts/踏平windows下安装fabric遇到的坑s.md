---
title: 踏平windows下安装fabric遇到的坑s
date: 2018-04-15 22:41:22
tags: [fabric]
categories: [web开发,工具]
---

![我是图片](https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=2127917617,591166268&fm=27&gp=0.jpg)

# 前言
本文主要介绍python环境的文件上传部署工具，和在windows平台上安装fabric的过程遇到的坑和一些解决方案

<!--more-->

# fabric
fabric是一个用python开发的部署工具，最大的好处就是不用登陆服务器，在本地运行python脚本就可以轻松做到远程部署

fabric提供了几个简单的api来完成所有的部署  
* local() 本地执行命令
* run() 服务器运行命令
* put() 把本地文件上传到服务器
* with cd('/path/to/dir/') 指定服务器的目录

在项目根目录下增加fabfile.py文件，这个文件就是控制部署的脚本

详细请参考廖雪峰老师的fabric介绍：[https://www.liaoxuefeng.com/article/001373892650475818672edc83c4c978a45195eab8dc753000](https://www.liaoxuefeng.com/article/001373892650475818672edc83c4c978a45195eab8dc753000)

# 安装python
所以这个工具依赖python环境，在下是windows系统，需要装python环境（不得不吐槽windows，安装这些环境过程中会遇到很多坑，不想mac系统，可以跟liunx流畅的打交道）

python下载地址：[https://www.python.org/getit/](https://www.python.org/getit/)  
最稳地还是建议安装2.7版本，因为后面安装的一些依赖包依赖2.7版本

# 安装pip
pip可以理解为python的一个包管理工具,用pip工具去安装fabric
```
pip install fabric
```

pip下载地址：[https://pypi.python.org/packages/source/p/pip/pip-8.0.2.tar.gz#md5=3a73c4188f8dbad6a1e6f6d44d117eeb](https://pypi.python.org/packages/source/p/pip/pip-8.0.2.tar.gz#md5=3a73c4188f8dbad6a1e6f6d44d117eeb)

进入pip的安装目录，运行：
```
python setup.py build
python setup.py install
```
若此时出现错误说，not found module 'setuptools',需要补装一下  
setuptools下载地址：[https://pypi.python.org/packages/source/s/setuptools/setuptools-19.6.tar.gz#md5=c607dd118eae682c44ed146367a17e26](https://pypi.python.org/packages/source/s/setuptools/setuptools-19.6.tar.gz#md5=c607dd118eae682c44ed146367a17e26)

进入安装目录执行
```
python setup.py build
python setup.py install
```

或者参考另一个方法：[https://blog.csdn.net/newjueqi/article/details/47446965](https://blog.csdn.net/newjueqi/article/details/47446965)

此时还有一步，就是把pip的安装目录添加进环境变量(windows就是有点麻烦，有钱还是买个macbook)

这时就可以用前面说的命令安装fabric了

# 运行fabfile.py

```
fab test
```
test是fabfile.py里定义的一个方法
```
fabfile.py

def test(remote='121.41.121.230', make=True):
    print '开始同步代码...'
    if make:
        local('make test')
    remote_dir = '/home/lukou/app/paidan/test2'
    sync_str = 'rsync --delete-after -r %s %s@%s:%s'
    local(sync_str % (os.getcwd() + '/dist/*', env.user, remote, remote_dir))
    print '同步完毕'
```
这时候window上会报错，说make test这命令执行出错  

前面已经说了，local()是执行本地命令，那么问题就是windows没有make这个命令，所以要安装make命令的工具

其实不安装也可以，make test就是运行package.json里面的test脚本，直接把它改为npm run test就好

在次运行fabfile脚本

还是会出错，说没有rsync命令

## 安装rsync工具
cwrsync下载地址：[https://www.itefix.net/cwrsync](https://www.itefix.net/cwrsync)
记得把安装目录添加到PATH

再次运行fabfile脚本

还是会出错，错误信息如下：
```
dup() in/out/err failed
rsync: connection unexpectedly closed (0 bytes received so far) [sender]
rsync error: unexplained error (code 255) at io.c(226) [sender=3.1.2]
```
百度了一下，找个了一个解决方案，原因是本地有两个ssh，而正在使用的ssh不是cwrsync下的ssh.exe，在windows下环境变量是从前到后查找的，假如找到ssh就不会往后找  

解决方案参考：[https://stackoverflow.com/questions/7261029/why-is-this-rsync-connection-unexpectedly-closed-on-windows](https://stackoverflow.com/questions/7261029/why-is-this-rsync-connection-unexpectedly-closed-on-windows)

所以要把path里cwrsync的安装目录提前，像如下
```
λ where ssh
E:\ProgramFiles\cwRsync_5.5.0_x86_Free\bin\ssh.exe
C:\Program Files (x86)\Git\usr\bin\ssh.exe
```

再次运行fab test

也还是会出错，就是这么皮。要不然这标题就不会在坑后面加s了

这次报的错误主要是语法错误
```
The source and destination cannot both be remote.
rsync error: syntax or usage error (code 1) at main.c(1274) [Receiver=3.1.2]
```

这下简单了，百度一下rsync命令在windows下的用法就行了，最后改成如下：
```
fabfile.py

def test(remote='121.41.121.230', make=True):
    print '开始同步代码...'
    if make:
        local('npm run test')
    remote_dir = 'app/paidan/test2'
    sync_str = 'rsync --delete-after -r %s %s@%s:%s'
    local(sync_str % ('/cygdrive/E/work/lukou/mirror/dist/*', env.user, remote, remote_dir))
    print '同步完毕'
```

最后运行，输入密码就成功了！！！

![](http://p2.so.qhimgs1.com/bdr/_240_/t01b985b931a308278e.jpg)









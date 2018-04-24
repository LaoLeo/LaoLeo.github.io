---
title: 聊聊git的分支和rebase命令
date: 2018-04-02 20:46:00
tags: [git]
categories: [工具]
---

![我是图片](https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=246425395,1170785252&fm=27&gp=0.jpg)

# 前言
进入公司已经一周的时间，在项目的协作开发中，根据公司对代码提交的要求，对使用git所遇到的一些问题情况，做了一个简单的总结，以便帮助遇到同样问题的实习生，提高的解决问题效率，进而更快的适应公司的开发节奏。

<!--more-->

> 目录
> * git简介
> * 分支
> * rebase的两个用法

# git简介

git是个分布式和渐进式的版本管理工具。分布式是指代码可以克隆很多份，不像svn（集中式版本管理工具）集中对一份代码进行版本管理，而git允许你将项目fork成自己一份。而渐进式是指git会智能的diff代码的改动情况，但不是记录文件的具体变化，而是保存一系列不同时刻的文件快照。

具体参考廖雪峰老师的[git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/)和[git官网](https://git-scm.com/book/zh/v2/)

# 分支
为了更好的使用git去解决一些复杂的版本冲突问题，有必要先理解git的分支。


**快照**  

其实要理解git的分支，首先要了解git是如何保存文件的  
git以文件快照的方式保存文件的更改情况，而不是cope每一份代码  
那么如何理解文件快照的？  
> 快照，我们可以把它抽象为照片，可以想象一下用相机拍下桌子上物体，照片记录着每个时刻物体的位置和状态，git就是把这每个照片保存起来，等需要的时候指定一张照片来把物体摆设成这个样子


**暂存**  

在暂存时，git把每个文件看成一个blob对象，把快照保存进blob对象里面，然后就加进暂存区域等待提交

**提交**

commit的时候，git会生成一个数对象来记录这个目录结构（包括子目录）和blob对象的索引，还会生成一个commit对象来保存一个指向这个树对象的指针和其他一些提交信息（msg，time，author，email）  
有了这个指针，就可以方便追溯到这次commit的文件快照，来重现这个时刻的文件状态  

![首次提交简介图](https://git-scm.com/book/en/v2/images/commit-and-tree.png)

以上的是首次提交的情况，在第二次提交时  
在原有的情况下，git再生成一个指针去指向上一次的commit对象，我们称之为父对象  
![再次提交简介图](https://git-scm.com/book/en/v2/images/commits-and-parents.png)
现在情况就明了了，git是以单链表结构来追溯文件状态的

**分支指针**


既然git是通过指针去追溯文件快照的，那么是不是可以创建多几个指针去多拿几张照片，进而多重现几个状态呢？  
所以分支就是所谓的指针，那么git又是怎么知道我在哪个分支上呢？  
没错，再创建一个指针去指向这个分支指针，来表示我当前所在的分支。
![分支简介图](https://git-scm.com/book/en/v2/images/branch-and-history.png)
所有的提交都是以单链表结构连接起来的，通过这个指针来追溯到所需的commit对象  

但是冲突是如何产生的呢？  
看个图就明白了
![分支简介图](https://git-scm.com/book/en/v2/images/advance-master.png)
其实就是多个commit对象中指向父对象的指针指向了同一个父对象。而两个照片中物体在同一个时刻有两个状态，所以就产生了冲突。  

使用命令查看分支的指向情况，或者说项目的分叉情况
```
git log --oneline --decorate --graph --all  
```

# rebase的两种用法
**1. 合并commit**  

 > 你肯定也试过这样的场景，leader交给你一个小需求，一个小时不到就解决了，提交一个commit，但过了一会，觉得代码还有可以改进的地方，作为一个努力上进的人，你肯定不会只求完成需求这么简单，所以再提了一版...
 
 但这个几个commit都是为了同一个需求开发的，leader去review代码的时候，几个commit还好，但实际开发中有时是十几个commit，这样对于review代码和追踪版本时，就显的尤为困难了
 
 所以需要将为同一个需求开发的这些commit合并成一个
 
 ```
 git rebase -i HAED~n<commitID>
 ```
 上面的n代指数字，表示包括当前以前的几个commit  
或者把HAED~n替换为n的commitID，效果一样

这是会出现一个编辑界面
```
pick 5e187c7dbe8    add center style indent  
pick 6d577eb3440    add center style  
pick f9b9508a3ab    add center style  
pick 111ab9cc261    update templates  
# Rebase 150a643..2fad1ae onto 150a643  
#  
# Commands:  
#  p, pick = use commit  
#  r, reword = use commit, but edit the commit message  
#  e, edit = use commit, but stop for amending  
#  s, squash = use commit, but meld into previous commit  
```
把第二到四行的pick更改为s，就会使用这个commit，并且合并前个commit

若出现冲突，解决冲突，应用最新的版本，使用
```
git add . //提交索引
git rebase --continue //继续rebase操作
```
若在合并commit的过程中想放弃，可以使用命令
```
git rebase --abort
```
此时会出现一个编辑窗口，保存退出就行

最后把本地推上远程
```
git push -f //必须带上-f，以本地覆盖远程
或者指定远程和分支
git push (origin master) -f
```
因为本地的几个commit已经合并成了一个，而若远程还有这几个commit，需要以本地覆盖远程

**2. 合并分支**  

有如下分支图
```
graph LR
C2-->C1
C3-->C2
C4-->C2
master((master))-->C3
dev((dev))-->C4
```
在master上合并dev我们一般用
```
git merge dev
```
此时分支图就变为
```
graph LR
C2-->C1
C3-->C2
C4-->C2
C5-->C3
C5-->C4
master((master))-->C5
dev((dev))-->C4
```
这样分支图就显的很乱，使用rebase能够让合并后的分支图变得简洁  
在最新的分支dev上
```
git rebase master
```
```
graph LR
C2-->C1
C3-->C2
C4-->C2
'C4'-->C3
master((master))-->C3
dev((dev))-->'C4'
```
dev上的C4会被cope成'C4'并且dev指向'C4',保存在rebase文件夹，然后'C4'指向C3，最后C4会当成垃圾清除，最后分支图就变为了
```
graph LR
C2-->C1
C3-->C2
'C4'-->C3
master((master))-->C3
dev((dev))-->'C4'
```
若有冲突,解决后
```
git add .
git rebase --continue
```
随时放弃
```
git rebase --abort
```

**在拉去远程更新时也可以使用rebase**
```
git pull --rebase
或者指定远程和分支
git pull upstream master --rebase
```

> ps:  
倘若通过git pull --rebase导致在gitlab上发merge request时，出现了其他人的本已经提交了的commit信息，不妨用使用
```
git reset --soft <commitID> //保留更改回退
```
再次commit，在gitlab再发merge request，应该就干净了（只是自己的提交信息）

**回到最初**  

解决git复杂问题的前提还是得先看的明白git的分支图，清晰每条命令对git分支的影响。








---
title: github使用心得
date: 2016-05-10 23:46:00
tags: [github,基础]
---

> 仅以此文提醒下次不会再犯之错
<!--more-->

* 在notepad上编辑md文件，首先要改编码（UTF-8 without BOM）
***
* hexi上commit后的内容，直接个git push origin hexi，再hexo d -g 就可以发布出去了。
第二种可行方法：checkout到master分支上，merge hexi分支后，
再push到远程库的hexi分支（git push origin master:hexi） 括号里master是
本地master分支，hexi是远程的hexi分支

* hexo d -g 要在hexi分支下才能起作用，让远程库的内容发布出去
***
* git remote add （自取的主机名） （地址）
目录下有了该本机，才能pull和push

* 更新fork的远程库
add主机 》》 pull到本地库 》》push到远程库
命令如下：
git remote add itxiehui https://itxiehui/itxiehui.github.io
git pull itxiehui develop
git remote add IT https://github.com/LaoLeo/itxiehui.github.io.git
git push IT develop
解析：
IT表示push到自己的远程仓库，该目录下要有该主机
itxiehui是指owner的主机，IT是member即我fork到我仓库的主机




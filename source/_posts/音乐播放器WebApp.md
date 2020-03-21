---
title: 音乐播放器WebApp有感
date: 2017-5
tags: [layout,animation,clear,audio,进度条, 播放按钮, QQ音乐API]
---


## 前言
花了差不多一周的时间，用html5的新标签audio制作了一个音乐播放器，感觉最多坑的地方在于前期的布局，逻辑代码的规划也很重要。

在布局上填了很多坑，比如资源税浮动父元素没了高度，用clear也解决不了，不得不利用js设置父的高度等之类的，发现布局也有大学问，这一点要注重。

第二点就是代码规划，因为刚学了前端js的MVC框架（将控制逻辑，模型，视图的js划分开），所以想尝试，在项目里我的mvc框架为：
* controller/appController.js   控制器总开关，程序的入   口，将controllers连接起来
* controller/playerController.js  播放器的控制器
* view/pageStyle.js 页面样式的改变逻辑
* view/scale.js  进度条对象
* songs.js  全局的songs模型

结合面向对象的方法，我在点击播放器按钮的时候，进度条，图片旋转和audio同时开始工作，所以把他们捆绑起来会比较好。

下面就这个项目里一些零散的知识点做些记录，以便一下使用。

## 圣杯布局
圣杯布局是三栏布局的经典布局方式，两侧固定宽度中间自适应。  
```
<div class="both">
    <div class="middle">中间</div> //优先加载
    <div class="left">左边</div>
    <div class="right">右边</div>
</div>

<style>
    .box{
        padding-left: 250px;  //关键
        padding-right: 250px;  //关键
        line-height: 200px;
        text-align: center;
        color: #fff;
    }
    .box:before,.box:after{ //伪类清除浮动，父元素有高度
        content: '.';
        display: block;
        clear:　both;
        height: 0；
        visibility: hidden; //元素不可见,但占据空间
    }
    .middle,.left,.right{
        float: left;
        height: 200px;
    }
    .middle{
        width: 100%;
        background-color: red;
    }
    .left{
        width: 250px;
        background-color: blue;
        margin-left: -250px; //关键
    }
    .right{
        width: 250px;
        background-color: yellow;
        margin-right: -250px; //关键
    }
</style>
```
三栏布局的另一经典是双飞翼布局，原理是middlediv下内嵌类为inner的div，利用inner的margin-left和margin-right。左右侧栏div分别float左右。  
另一种方法是利用定位：
```
  <style>
   body{margin:0; padding:0;}
   #left{float:left;width:150px;background:red;}
   #right{float:right;width:200px;background:green;}
   #middle{
   position:absolute; //关键
   left:150px;
   right:200px;
   word-wrap:break-word;  //让字体打断，不会超出div
   background:blue;
   }
</style>

<div id="middle">
        middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;
        middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;middle;
</div>
<div id="left">left</div>
<div id="right">right</div>
```
关于布局，还有很多种，请参考：
[http://blog.csdn.net/andiqu/article/details/50045609](前端布局)
关注弹性布局（flex），和rem

## animition的播放和暂停
```
-webkit-animation-play-state: running | paused
```
```
animation-fill-mode: backwards | forwards | both; //播放前显示动画第一帧，播放完显示动画的最后一帧，向前和向后模式都被应用
animation-direction: reverse | alternate; //播放顺序，resverse反向
animation-iteration-count: num | infinite // 2 | 循环 
```

## 进度条的制作
```
<ul id="lanren">
	<li class="curr">00:00</li>
	<li class="progress">
		<div class="scale" id="bar">
			<div></div>
			<span id="btn"></span>
		</div>
	</li>
	<li class="total">00:10</li>
</ul>

<style>
body{margin:0;padding:0;font-size:12px;}
ul#lanren{ margin:50px auto; overflow: hidden; padding: 0 50px 0 50px;}
#lanren li{font-size:12px;line-height:24px; height:24px;list-style:none; position: relative; float: left;}
li.progress{
	width: 100%;
}
li.curr{
	width: 50px;
	margin-right: -50px;
	left: -50px;
	text-align: center;
}
li.total{
	width: 50px;
	margin-left: -50px;
	right: -50px;
	text-align: center;
}
.scale{ background-color: #E4E4E4; border-left: 1px #83BBD9 solid;  width: 100%; height: 3px; position: relative; font-size: 0px; border-radius: 3px; top:11px;}

.scale span{width:8px;height:8px;position:absolute;left:-2px;top:-2.5px;cursor:pointer; background-color: #000; display: inline-block; -webkit-border-radius: 50%;}

.scale div{ background-color: #3BE3FF; width: 0px; position: absolute; height: 3px; width: 0; left: 0; bottom: 0; }
</style>
```

## 播放按钮的制作和发光动画
```
<div class="playContainer">
	<li class="rewindBtn">  
	    <a href="javascript:;" title="rewind">Rewind</a>  
	    <a href="javascript:;" title="rewind">Rewind</a>  
	</li>
	<li class="playBtn BtnBig">  
	    <a href="javascript:;" title="start">Start</a>  
	</li>
	<!--
	<li class="pauseBtn">  
	    <a href="#" title="pause">Pause</a>  
	</li> 
	--> 
	 
	<li class="forwardBtn playBtn">  
	    <a href="javascript:;" title="forward">Forward</a>  
	    <a href="javascript:;" title="forward">Forward</a>  
	</li>  
</div>

<style>
.playContainer li { 
	position: relative; 
	float: left; 
	border: 50px solid #404040; 
	border-color: rgba(119, 119, 119, 0.3);
	color: transparent; 
	height: 0; 
	width: 0; 
	-webkit-border-radius: 100%; 
	-moz-border-radius: 100%; 
	-o-border-radius: 100%; 
	border-radius: 100%; 
	margin: 0 50px; 
}  
.playContainer a { 
	border-style: solid; 
	text-indent: -9999px; 
	position: absolute; 
	top: -16px; 
	left: -6px;  
}  
li.BtnBig{
	border-width: 70px;
	margin-top: -15px;
}
.playBtn a { 
	border-color: transparent transparent transparent #fff;  
	width: 0; 
	height: 0; 
	border-width: 24px 0 24px 36px;
	left: -10px;
	top: -22px;
}  
.pauseBtn a { 
	border-color: transparent white;  
	border-width: 0 14px;
    height: 36px;
    width: 12px;
    left: -20px;
}    
.forwardBtn a { 
	border-left-width: 16px; 
	left: 2px;
	border-width: 16px 0 16px 16px;
	top: -15px;
}  
.forwardBtn a:first-child { 
	margin-left: -14px;  
}  
.rewindBtn a { 
	border-width: 16px 16px 16px 0; 
	border-color: transparent #fff transparent transparent; 
	width: 0; 
	height: 0; 
}  
.rewindBtn a:first-child { 
	margin-left: -14px; 
} 

//发光动画
@-webkit-keyframes bs {
  0% {
    box-shadow: inset -1px 1px 3px 2px #444444, inset 1px -1px 3px 2px #222222, 0 0 0px 0 #787776;
  }

  50% {
    box-shadow: inset -1px 1px 3px 2px #444444, inset 1px -1px 3px 2px #222222, 0 0 40px 0 #ffffff;
  }

  100% {
    box-shadow: inset -1px 1px 3px 2px #444444, inset 1px -1px 3px 2px #222222, 0 0 0px 0 #b2ff1a;
 }
}
</style>   


```

## URI编码
url里的中文服务器是识别不了的，要经过URI编码成Unicode编码，统一资源标识符，浏览器才能准确的找到资源。
```
encodeURI("春节") //%DD%DD%DD
decodeURI("%DD%DD%DD")  //解码：春节
相同：
escape()
unescape()
```

## ajax跨域
不涉及后台的跨域，直接用ajax发http请求：
[https://bird.ioliu.cn/#interface](http://note.youdao.com/)
其他的跨域都要设计到后台
与jsonp有关

## QQ音乐的API
```
songsAPI = http://s.music.qq.com/fcgi-bin/music_search_new_platform?t=0&n=10&aggr=1&cr=1&loginUin=0&format=json&inCharset=GB2312&outCharset=utf-8&notice=0&platform=jqminiframe.json&needNewCode=0&p=1&catZhida=0&remoteplace=sizer.newclient.next_song&w={0}

n= 10 ,搜索的数据条目
w= content ,搜索内容
返回的数据中有个f属性，
DATA.data.song.list[i].f: xxx|xxx|xxx.....
其中第一个是songID,第五个是imgID


imgAPI = http://imgcache.qq.com/music/photo/album_500/imgID后面两个数/500_albumpic_imgID_0.jpg

500是图片的大小，支持300和500



srcAPI = http://210.38.1.134:9999/ws.stream.qqmusic.qq.com/songID.m4a?fromtag=46
```

## 计时器
当在页面要多次用到定时器时，可以考虑把它挂载到全局。  
当用到多个计时器是可以把它放在一个数组里，方便管理。

## 百度到的资源
专门说进度条的网站：[https://usablica.github.io/progress.js/](https://usablica.github.io/progress.js/)  
js,jq获取div高度：[http://www.cnblogs.com/xiaopin/archive/2012/03/26/2418152.html](http://www.cnblogs.com/xiaopin/archive/2012/03/26/2418152.html)  
[获取div的margin，border，padding，content的宽高](http://blog.csdn.net/zy1281539626/article/details/61913770)  
js控制audio标签：[http://blog.csdn.net/u014520745/article/details/52412427](http://blog.csdn.net/u014520745/article/details/52412427)
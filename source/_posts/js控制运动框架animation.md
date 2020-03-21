---
title: js控制运动框架animation
date: 2017-7-20
tags: [js,css,animation]
---

## js控制运动框架animation

目标：做一个通过按钮控制旋转的动画，下面的是效果图：
![image](http://note.youdao.com/yws/api/personal/file/221639B83D0B4AD3A49229F58A468023?method=download&shareKey=fb6cd7994e20ce6c6ea7f94a1609b490)

想想都很简单，就写个css框架，然后用js给img节点添加animation样式。
所以一开始是这样的的：
```css
/*旋转动画*/
@keyframes rotation{
	from{
		transform: rotate(0deg);
		-webkit-transform: rotate(0deg);
		-moz-transform: rotate(0deg);
		-o-transform: rotate(0deg);
	}
	to{
		transform: rotate(360deg);
		-webkit-transform: rotate(360deg);
		-moz-transform: rotate(360deg);
		-o-transform: rotate(360deg);
	}
}
```
```js
$positer.css('animation', 'rotation 30s linear infinite'); //开始旋转动画

$positer.css('animation', ''); //停止旋转动画
```

然而，问题就来了。停止的时候图片直接回归原始状态。  
这得咋办呢？难道要写一个动画框架，用定时器去使transform的rotate去转动吗？
好说干就干
```js
var deg = 0;  //记录旋转角度
var rotateTimer;

function startRotation($obj, speed) {
	clearInterval(rotateTimer);
	rotateTimer = setInterval(function(){
		deg++;
		if(deg >= 360){
			deg = 0
		}
		
		$obj.css('-webkit-transform', 'rotate('+ deg +'deg)');
	}, speed)

}
function stopRotation() {
	clearInterval(rotateTimer);
}

//调用
var $positer = $('#playerPage .positer img');

startRotation($positer, 80); //转
stopRotation() //停
```
哈哈，就是这么简单。  
。。。  
阿西吧，旧的问题解决了，只是新的问题来的开始而已。这时，播放的动画没有想animation那么流畅，一卡一卡的。天啊，这怎么解决，假如有个属性控制animation的播放不就好了嘛，那用搞这么多事情。

一百度，果真有！
```css
animation-play-state: paused || running;
```

代码：
```css
#playerPage .positer img{
    animation: rotation 30s linear infinite;
    -webkit-animation: rotation 30s linear infinite;
    -moz-animation: rotation 30s linear infinite;
    animation-play-state: paused;
    -webkit-animation-play-state: paused;
    -moz-animation-play-state: paused;
}
```
```js
$positer.css('animation-play-state', 'running'); //开始旋转动画

$positer.css('animation-play-state', 'paused'); //停止旋转
```
---
title: 饼状loading效果
date: 2017-9-20
tags: [html5,css3]
---

###   垂直和水平居中 
	
水平居中是最简单的，在设置了大小的元素上加margin：0 auto
关键是垂直居中
使用定位position：absolution;  //脱离文档流
	    top: 50%;	//这时顶部处于中央位置
	   margin-top: -(height/2) px;
或者第三句可以使用位移：transfrom：translateY(-50%)

另一种方法是使用flex布局
在居中元素的父元素上：display：flex;
			       align-items:center; //垂直居中
			       justify: center;	//水平居中
可是flex布局存在兼容性问题  ，慎用！
参考：http://www.cnblogs.com/yugege/p/5246652.html

**让脱离了文档流的p标签垂直和水平居中**  
```
<nav class="mynav">
		<div>
			<p><span></span><span></span><span></span></p>
		</div>		
	</nav>
	
<style>
    .mynav{
    	height: 100px;
    	width: 100%;
    	margin-top: 70px;	
    }
    .mynav div{
    	height: 100%;
    	position: relative; //垂直
    	text-align: center; //水平
    }
    .mynav p{	
    	width: 100%; //水平
    	position: absolute; //垂直
    	top: 50%;   //垂直
    	margin-top: -12px;  //垂直
    }
    .mynav p span{
    	display: inline-block;
    	width: 24px;
    	height: 24px;
    	border-radius: 50%;
    	-webkit-border-radius: 50%;
    	background-color: #000;
    	margin-right: 20px;
    
    }
</style>
```

### 思路

[查看效果](https://ww2.sinaimg.cn/large/006tNbRwly1fcr4ycdb9cg30d80dm0tf.gif)

这个loading效果分两部份，第一部分是四分之三圆，第二部份是中间的圆饼。
首先如何画圆呢？
一般是在一个正方形的div上设置border-radius=圆的半径
可是现在是用另外的方法，用border来画圆
```
width:0;
height:0;
border:50px solid #000;
border-radius:50px;
```
可以用border-top-color来设置圆的颜色，这样一个圆就可以有四种颜色，那么四分之三圆就是一部分的颜色设为透明transparent。
中间的圆饼其实是一个红色和粉红色的底圆和一个红半圆，一个粉红半圆实现的，半圆其实就是两部分的border颜色设为transparent

一个红色和粉红色的底圆：
```
width: 0;
height: 0;
border-radius: 50px;
border:50px solid;
border-top-color: rgb(251,139,189);
border-left-color: rgb(255,41,140);
border-right-color: rgb(251,139,189);
border-bottom-color: rgb(255,41,140);
```

一个红半圆：
```
width: 0;
 height: 0;
border-radius: 50px;
border: solid rgb(251,139,189) 50px  ;
border-right-color: transparent;
 border-bottom-color: transparent;
```
分析这个效果，其实是一开始是个红色的圆，然后粉色圆在慢慢出来，等整个圆是粉红色时，红圆慢慢出来。
我们可以这样两个半圆重叠，红色在其上，都放在底圆上，在动画框架的第一个过程上红圆转180deg，第二过程粉红圆转180deg，红圆继续保持位置，这个过程开始事时粉红圆的z - index大于红圆。第三个过程，红圆z-index还是小于粉红圆，红圆在转。第四个过程开始，红圆z-index大于粉红圆，粉红圆在转。

对比下面的两个动画框架：
```
@keyframes red-rotate{
		    0% {
		    	transform: rotate(-45deg);
		    }
		    25% {
		    	transform: rotate(-225deg);
		    }
		    50% {
		    	transform: rotate(-225deg);
		    	z-index: 2;
		    }
		    75% {
		    	transform: rotate(-405deg);
		    	z-index: 3;
		    }
		    100% {
		    	transform: rotate(-405deg);
		    }		
		}
		@keyframes pink-rotate{
			0% {
		    	transform: rotate(-45deg);		    	
		    }
		    25% {
		    	transform: rotate(-45deg);
		    }
		    50% {
		    	transform: rotate(-225deg);
		    	z-index: 3;
		    }
		    75% {
		    	transform: rotate(-225deg);
		    	z-index: 3;
		    }
		    75.001% {
		    	transform: rotate(-225deg);
		    	z-index: 2;
		    }
		    100% {
		    	transform: rotate(-405deg);
		    } 
		}
```
最后两个动画框架同时开始就大功告成了！

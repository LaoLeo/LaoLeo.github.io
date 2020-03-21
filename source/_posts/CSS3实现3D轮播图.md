---
title: CSS3实现3D轮播图
date: 2017-9-25
tags: [html5,css3]
---

### CSS3 3D旋转图片轮播

想学好3D变换，必须先理解表示空间方向的XYZ轴，这个其实很容易理解，毕竟高中做了辣么多的函数方程题，要注意的是Z轴，向着自己的方向就是Z的方向。

#### 变形transform
旋转：transform : rotateX()/rotateY()/rotateZ()
这个项目很明显是用rotateY(angle)

#### 关键的perspective属性
这个单词的意思是视角，故名思义，就是眼睛的位置，有两种写法：
1.写在舞台元素上，也就是动画元素的父元素上
```
.stage{
		perspective: 800px;
}
```
2.写在动画元素上，写在transform属性里
```
.stage .box{
		transform:perspective(800px) rotate(60deg);
}
```
视角的位置不一样，看到的景物也是有区别的，换个角度思考也是这个说法。
其实上面的两种写法是有区别的，前者视角在舞台的中心，后者则是在动画元素的中心。

#### 定义视角位置
```
.stage{
		perspective: 800px;
		perspective-origin:25% 250px;
}
```

#### 拉开与中心的距离
当所有图片都以同一个旋转中心旋转后，其实是没有立体感的，这时候就需要用到这个属性translateZ
```
.stage .box{
		transform: rotate(60deg) translateZ(500px); 
}
```

#### js控制transform属性
```
aImg[i].style.webkitTransform = "rotateY("+ ( - (count*60 - i*60) )+  "deg) translateZ(" + 483+"px)";
```
需要注意的是，aImg[i].style.webkitTransform只能用来设置css的transform属性，并不能用alert打印出来。同offsetLeft也left一样。

[参考](http://www.zhangxinxu.com/wordpress/2012/09/css3-3d-transform-perspective-animate-transition/）


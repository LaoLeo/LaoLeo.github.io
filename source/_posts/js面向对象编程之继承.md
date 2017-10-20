---
title: js面向对象编程之继承
date: 
tags: [js, 继承]
categories: [js]
---

![我是图片](https://i.loli.net/2017/07/20/596f9bc03c484.jpeg)

# 前言
> javascript本身不是静态语言，是一门动态语言，变量只有在运行的时候才知道是什么类型，严格上也没有像Java静态语言的类的概念((当然在ES6中提出了类的语法)，那么它是如何实现类的继承的呢？

<!--more-->

# 什么是继承？

继承就是在原类的基础上，略作修改，得到一个新的类，并且不影响原有类的功能。

我们都知道js中对象的实现，是基于它定位原型链系统。类也是一种对象，只不过类比一般的对象更完备，具有自身的属性和方法。所以在js中，类的继承必然少不了原型对象的概念。

而且类可以new出实例对象，通过构造函数的方式可以实现类的实例，也可以这样说，构造函数就是js的类。

# 继承的方式

## 构造函数伪装继承属性

我们以实现一个拖拽的demo来说明类的继承。现有一个拖拽类，这个拖拽类可以在页面中移动，并且能移出屏幕。

```js
function Drag(id)
{
	var _this=this;
	
	this.disX=0;
	this.disY=0;
	this.oDiv=document.getElementById(id);
	
	this.oDiv.onmousedown=function (ev)
	{
		_this.fnDown(ev);
		
		return false;
	};
}

Drag.prototype.fnDown=function (ev)
{
	var _this=this;
	var oEvent=ev||event;
	this.disX=oEvent.clientX-this.oDiv.offsetLeft;
	this.disY=oEvent.clientY-this.oDiv.offsetTop;
	
	document.onmousemove=function (ev)
	{
		_this.fnMove(ev);
	};
	
	document.onmouseup=function ()
	{
		_this.fnUp();
	};
};

Drag.prototype.fnMove=function (ev)
{
	var oEvent=ev||event;
	
	this.oDiv.style.left=oEvent.clientX-this.disX+'px';
	this.oDiv.style.top=oEvent.clientY-this.disY+'px';
};

Drag.prototype.fnUp=function ()
{
	document.onmousemove=null;
	document.onmouseup=null;
};
```

现在我们实现一个限制在窗口内的拖拽类，可以使用call来实现构造函数伪装。
```js
function LimitDrag(id)
{
	Drag.call(this, id); //(1)
}
```

要清楚，函数只不过是在某一环境下执行的在代码，这个环境就是js中的上下文，call()可以改变函数执行的上下文，也就是可以改变函数中this的指向。这就相当于将Drag类构造函数的代码，放在limitDrag类中执行了一遍。这个方式叫做构造函数伪装，可以实现类属性的继承，想要覆盖它父类的属性，只需要在(1)下面写属于自己的属性即可。

## 原型对象实现方法继承

我们先用这种方式实现方法的继承

```js
LimitDrag.prototype=Drag.prototype;
```

然后我们发现，LimitDrag实例可以使用Drag类的方法，但是要是我们先重构一个属于limitDrag类的fnMove()，发现Drag类的fnMove()也被覆盖了，这就影响了前面所说的，“并且不影响原有类的功能”。这个其实跟对象的引用类型有关，因为LimitDrag.prototype和Drag.prototype在内存中指向的是同一块堆内存。

解决这个问题有两种思路：
* 把原型对象上的方法拷贝一份
* 将子类的原型对象的__proto__属性指向父类原型对象

```js
//方法一
for(var i in Drag.prototype)
{
	LimitDrag.prototype[i]=Drag.prototype[i];
}
```

```js
LimitDrag.prototype.__proto__ = Drag.prototype
LimitDrag.prototype.constructor = LimitDrag;
```

其实方法二才算是js的继承，LimitDrag.prototype instranceof Drag为true，方法一只是使用拷贝实现了类的克隆，并没有通过原型链连接起来。

在js中，一切对象都是通过克隆Object.prototype来克隆的，然后通过动态的设置对象的__proto__属性的指向到另一个构造器的原型对象上，来实现继承的效果。

所以现在就可以改写LimitDrag类的fnMove()了，并且不会影响到Drag类了

```js
LimitDrag.prototype.fnMove=function (ev)
{
	var oEvent=ev||event;
	var l=oEvent.clientX-this.disX;
	var t=oEvent.clientY-this.disY;
	
	if(l<0)
	{
		l=0;
	}
	else if(l>document.documentElement.clientWidth-this.oDiv.offsetWidth)
	{
		l=document.documentElement.clientWidth-this.oDiv.offsetWidth;
	}
	
	if(t<0)
	{
		t=0;
	}
	else if(t>document.documentElement.clientHeight-this.oDiv.offsetHeight)
	{
		t=document.documentElement.clientHeight-this.oDiv.offsetHeight;
	}
	
	this.oDiv.style.left=l+'px';
	this.oDiv.style.top=t+'px';
};
```

# 结语

js是静态语言，在实现类的继承可能没有动态语言严谨，但是增加了它的灵活性。通过构造函数伪装和原型对象继承两种方式可以实现js中类的继承。

继承是编程一个重要的概念，可以适用很多场景，比如以上拖拽类的栗子中，假如项目已经大范围的使用了不限制的拖拽类，茂茂然的改原有的代码一定会非常的痛苦，所以利用继承的特性实现一个限制的拖拽类是一种很好的解决方案。
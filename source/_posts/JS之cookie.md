---
title: JS之cookie.md
date: 2016-06-2
tags: [js]
---

### cookie

页面用来保存信息，比如保存用户名，自动登陆

#### cookie的特性
* 同个网站中所有页面公享一套cookie
* 数量，大小有限
* 过期时间，可用js来控制。给expries赋值设置时间
* cookie存在于客户端，据说只有在狐火上可以单独使用，或者说在火狐上运行的问题很少

#### js中使用cookie
* document.cookie

#### cookie的操作
> 设置cookie
  * 格式：名字 = 值
  * 不会覆盖，等号是添加的意思
  *过期事件：expries = 时间

#### 练习代码
* 设置cookie
```javascript
function setCookie(name ,value, iDay)
{
	var oDate = new Date();
	oDate.setDate(oDate.getDate() + iDay);

	document.cookie = name +'='+value+';expires='+oDate;

}
	/*document.cookie = "user = Leo;expires=" +oDate ;
	document.cookie = "pass = 123";*/

```
* 读取cookie
```javascript
function getcookie(name)
{
	//username=laotuzhu; passwork=123456789; user=Leo;...
	var arr = document.cookie.split('; '); //千万不能少空格，因为cookie中的元素是是以；和空格分开的，少了就只能get第一个元素
	//arr->['username=laotuzhu','passwork=123456789','user=Leo',..]
	var i = 0;

	for(i= 0; i<arr.length; i++)
	{
		var arr2 = arr[i].split('=');
		// arr2->['username','laotuzhu', ...]
		if (arr2[0] == name) {
			return arr2[1];
		}
	
	}
	return '';
}

```
*主要是将字符串分割split*  
*注意： var arr = document.cookie.split('; '); //千万不能少空格，因为cookie中的元素是是以；和空格分开的，少了就只能get第一个元素*
* 删除cookie
```javascript
function removeCookie(name)
{
	setCookie(name,'1',-1);
}
```
*将时间设置为过期时间* 

#### cookie的使用
* 记录div的位置
```javascript
<style>
		#div1{
			width: 100px;
			height: 100px;
			background-color: red;
			position: absolute;
		}
	</style>
	<script>
function setCookie(name ,value, iDay)
{
	var oDate = new Date();

	oDate.setDate(oDate.getDate() + iDay);

	document.cookie = name +'='+value+';expires='+oDate;
}

function getcookie(name)
{
	//username=laotuzhu; passwork=123456789; user=Leo;...
	var arr = document.cookie.split('; '); //千万不能少空格，因为cookie中的元素是是以；和空格分开的，少了就只能get第一个元素
	//arr->['username=laotuzhu','passwork=123456789','user=Leo',..]
	var i = 0;

	for(i= 0; i<arr.length; i++)
	{
		var arr2 = arr[i].split('=');
		// arr2->['username','laotuzhu', ...]
		if (arr2[0] == name) {
			return arr2[1];
		}
	
	}
	return '';
}
function removeCookie(name)
{	setCookie(name,'1',-1);
}

	window.onload =function(){
			var oDiv = document.getElementById('div1');
			var disX =0;
			var disY =0;

			var x = getcookie('x');
			var y = getcookie('y');
			if(x)
			{
				oDiv.style.left =x+ 'px';
				oDiv.style.top = y+ 'px';	
			}

		oDiv.onmousedown = function(ev){
				var oEvent = ev || event;

				disX = oEvent.clientX- oDiv.offsetLeft;
				disY = oEvent.clientY- oDiv.offsetTop;

				document.onmousemove=function(ev){
					var oEvent = ev || event;

					var l = oEvent.clientX-disX;
					var t = oEvent.clientY-disY;

					if (l<0) {  //限制div移出可视区
						l=0;
					}else if(l>document.documentElement.clientWidth- oDiv.offsetWidth){
						l = document.documentElement.clientWidth- oDiv.offsetWidth;
					};
					if (t<0) {
						t = 0;
					}else if(t>document.documentElement.clientHeight- oDiv.offsetHeight){
						t =document.documentElement.clientHeight- oDiv.offsetHeight;
					}

					oDiv.style.left = l+'px';
					oDiv.style.top = t+'px';
				}

				document.onmouseup = function(){
					document.onmousemove = null;
					document.onmouseup = null;

					setCookie('x',oDiv.offsetLeft, 30);
					setCookie('y',oDiv.offsetTop,30);
				}
			}
		}
	</script>
</head>
<body>
	<div id="div1"></div>
</body>
```
*在onload下定好div位置，在ondown下记录*

* 记录用户名
```javascript 
<script>
	function setCookie(name ,value, iDay)
{
	var oDate = new Date();
	oDate.setDate(oDate.getDate() + iDay);

	document.cookie = name +'='+value+';expires='+oDate;

}
	/*document.cookie = "user = Leo;expires=" +oDate ;
	document.cookie = "pass = 123";*/

function getcookie(name)
{
	//username=laotuzhu; passwork=123456789; user=Leo;...
	var arr = new Array();
	var arr = document.cookie.split(';');
	//arr->['username=laotuzhu','passwork=123456789','user=Leo',..]
	var i = 0;

	for(i= 0; i<arr.length; i++)
	{
		var arr2 = arr[i].split('=');
		// arr2->['username','laotuzhu', ...]
		if (arr2[0] ==name) {
			return arr2[1];
		};

	}
	return '';
}

function removeCookie(name)
{
	setCookie(name,'1',-1);
}

window.onload = function()
{
	var form = document.getElementById('form1');
	var oUser = document.getElementsByName('user')[0];
	var a = document.getElementsByTagName('a')[0];

	form.onsubmit = function()
	{
		setCookie('user',oUser.value,2);
	}

	a.onclick = function ()
	{
		removeCookie('user');
		oUser.value = '';
	}
}
</script>
</head>
<body>
	<form id="form1" action="www.baidu.com">
		用户名：<input type="text" name="user">
		密码：<input type="password">
		<input type="submit" value="登陆">
		<a href="javascript:;">清除记录</a>
	</form>
</body>


```


---
title: jsϵ��֮DOM����
date: 
tags: [���,javacript,BOM]
---

�������Ǳ�����ѧϰjsʱ���֪ʶ�ܽᣬ�漰��ѧϰ�����е�һЩ֪ʶ�㣬������ϰ�Լ����ֳ����Ĵ�������������ϣ����js�İ������а�����

<!--more-->

# jsϵ��֮BOM����
***
## open

> window.open(href,target); target����Ϊ��ѡ�����������_blank��(���´��ڴ�)����_self��(�ڵ�ǰ���ڴ�)���з���ֵ�����ش��ڱ���

* ���ı�����д���룬�´��ڴ�ִ��
 ```javascript
 <script>
	window.onload = function(){
		var oBtn = document.getElementById('btn1');
		var oTxt = document.getElementById('txt1');

		oBtn.onclick = function(){
			var oNewWin = window.open('about:blank');
			oNewWin.document.write(oTxt.value);
		}
	}
</script>
<body>
	<textarea name="" id="txt1" cols="30" rows="10"></textarea>
	<br>
	<button id="btn1" value="����">����</button>
</body>
```

## close

> close��ҳ����Ҫ����js�򿪵�ҳ��
���磺
��open.html�� �ؼ�js���룺window.open(��close.html��,��_blank��);
��close.html�� �ؼ�js���룺window.close();
close.html��js����򿪵�ҳ��

## userAgent

> �鿴������汾
alert��window.navigator.userAgent��;

```javascript
<script>
	alert(window.navigator.userAgent)//�鿴������İ汾
</script>
```

## location

> window.location   ��ǰҳ��ĵ�ַ
window.location ='http://www.github.com';   ��һ���µ�ַ


## ϵͳ�Ի���
> �����alert���� ,û�з���ֵ
ѡ���confirm�������ʵ����ݡ���������boolen
�����prompt�������⡱����Ĭ��ֵ����������������ַ�����null��

 
## ���ڳߴ磬�������ߴ�
> �������ߴ�
document.documentElement.clientWidth
document.documentElement.clientHeight
��������
document.body.scrollTop
document.documentElement.scrollTop	*��������Ϊ�˸��õļ����ԣ�chrome��firefoxֻ��ʶǰ��*

* �����
 ```javascript
 <style>
	#div1{
		width: 100px;
		height: 100px;
		position: absolute;
		right: 0;
		background: pink;
	}
</style>
<script>
window.onresize = window.onload = window.onscroll  = function (){
		var oDiv = document.getElementById('div1');
		var scrollTop = document.documentElement.scrollTop || document.body.scrollTop;
		var t = (document.documentElement.clientHeight-oDiv.offsetHeight)/2;

		oDiv.style.top = scrollTop + t +'px';
	}
</script>
<body style="height:1000px;">
	<div id="div1"></div>
</body>
```

* �ص�����
 ```javascript
<style>
	#btn{
		position: fixed;
		bottom: 0;
		right: 0;
	}
</style>
<script>
	window.onload = function(){
		var oBtn = document.getElementById('btn');
		var timer =null;
		var bSys = null;

		//��μ�⵽�û������˹�����
		window.onscroll = function(){
			if (!bSys) {
				clearInterval(timer);
			};
			bSys = false;
		}
		oBtn.onclick = function(){

			timer =setInterval(function(){
				var scrollTop = document.documentElement.scrollTop || document.body.scrollTop;
				var iSpeed = Math.floor(-scrollTop/8);
				if (scrollTop == 0) {
					clearInterval(timer);
				};
				document.documentElement.scrollTop = scrollTop +iSpeed;
				bSys = true;
			},30)
			
		}
	}
</script>
<body>
	<button id="btn">�ص�����</button>
	<p>**</p>
	<p>**</p>
	<p>**</p>
	<p>**</p>
	...
</body>
 ```
 
> **С��**
>������JavaScript��BOM��Browsert object model����һЩ���õĻ������Լ�����������ϰ����


><span style="font-size:12px">���ı���: <a href="{{ permalink }}">  { {title} }  </a>
>��������: <a href="http://itxiehui.github.io/">������</a>  
>���Э��: <img alt="֪ʶ�������Э��" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/80x15.png" /><a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">?����-������-��ͬ��ʽ���� 4.0</a></span>
 

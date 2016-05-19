---
title: JS֮Ĭ����Ϊ
date: 2016-05-19 17:10:00
tags: [���,javacript]
---

�������Ǳ�����ѧϰjsʱ���֪ʶ�ܽᣬ�漰��ѧϰ�����е�һЩ֪ʶ�㣬������ϰ�Լ����ֳ����Ĵ�������������ϣ����js�İ������а�����

<!--more-->

## **Ĭ����Ϊ**
***

> ������Դ����¼������磺�Ҽ��˵���a������ת�����ύ
return false;ȡ��Ĭ����Ϊ

### ��ϰ����
#### 1.�����Ҽ��˵�
> --�����Զ�����Ҽ��˵�
```javascipt
<style>
	*{
		margin: 0;
		padding:0;
	}
		#ul1{
			width: 130px;
			height: 205px;
			background-color: #ccc;
			list-style: none;
			position: absolute;
			text-indent: 10px;
			display: none;
		}
		#ul1 li{
			padding-bottom: 10px;
			border-bottom: 1px solid #000;
			height: 30px;
			line-height: 30px;

		}
	</style>
	<script>
	window.onload = function(){
		var oUl = document.getElementById('ul1');

		document.oncontextmenu = function(ev){
			var oEvent = ev || event;

			oUl.style.display = 'block';
			oUl.style.left = oEvent.clientX +'px';
			oUl.style.top = oEvent.clientY+'px';


			return false;
		}

		document.onclick =function(){
			var oUl = document.getElementById('ul1');
			oUl.style.display = 'none';
			
		}
	}
	</script>
</head>
<body>
	<ul id="ul1">
		<li>����</li>
		<li>ˢ��</li>
		<li>������ģʽ</li>
		<li>�鿴Դ����</li>
		<li>����</li>
	</ul>
</body>
```
#### 2.��קdiv�����װ棩
> ԭ������div���ϽǶ�����벻�䣻����������onmousedown���洢���룩��onmousemove����������ƶ����λ�ã���onmouseup��ȡ�����������������ǰ������
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
		window.onload =function(){
			var oDiv = document.getElementById('div1');
			var disX =0;
			var disY =0;

			oDiv.onmousedown = function(ev){
				var oEvent = ev || event;

				disX = oEvent.clientX- oDiv.offsetLeft;
				disY = oEvent.clientY - oDiv.offsetTop;

				oDiv.onmousemove=function(ev){
					var oEvent = ev || event;

					oDiv.style.left = oEvent.clientX-disX+'px';
					oDiv.style.top = oEvent.clientY-disY+'px';
				}

				oDiv.onmouseup = function(){
					oDiv.onmousemove = null;
					oDiv.onmouseup = null;
				}
			return false�� //����ɰ�����bug
			}
		}
	</script>
</head>
<body>
	<div id="div1"></div>
</body>
```

* ���div�϶�ʱ������Ƴ�����
	> ԭ������ϵ�̫�죬����div�¼�ֹͣ���ã���취�����¼���Χ����div��onmousemove��onmouseup�¼���Ϊdocument��
```javascript
document.onmousemove=function(ev){
					var oEvent = ev || event;

					oDiv.style.left = oEvent.clientX-disX+'px';
					oDiv.style.top = oEvent.clientY-disY+'px';
				}

				document.onmouseup = function(){
					document.onmousemove = null;
					document.onmouseup = null;
				}
```	
* ���div�ϳ�������
```javascript
document.onmousemove=function(ev){
					var oEvent = ev || event;

					var l = oEvent.clientX-disX;
					var t = oEvent.clientY-disY;

					if (l<0) {  //����div�Ƴ�������
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

```

><span style="font-size:12px">���ı���: <a href="{{ permalink }}">  {{title}}  </a>
>��������: <a href="http://itxiehui.github.io/">������</a>  
>���Э��: <img alt="֪ʶ�������Э��" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/80x15.png" /><a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">?����-������-��ͬ��ʽ���� 4.0</a></span>
 
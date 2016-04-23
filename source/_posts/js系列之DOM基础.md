
---
title: H5��ǰ������
date: 2016-04-15 00:00:09
tags: [javacript,DOM]
---

�������Ǳ�����ѧϰjsʱ���֪ʶ�ܽᣬ�漰��ѧϰ�����е�һЩ֪ʶ�㣬������ϰ�Լ����ֳ����Ĵ�������������ϣ����js�İ������а�����
<!--more-->
##DOM�ڵ�

###childNodes��nodeType��children

* childNodes�ӽڵ㣬ֻ�ʺ����ھɰ��ie����������ڻ���ȸ��ϻὫ�ı��ڵ㣨�ո񣩰���������

* nodeType�ڵ����ͣ�������һ�����֡�1��ʾԪ�ؽڵ㣬2ָ���Խڵ㣬3���ı��ڵ㡣

* children����ͬ��childNodes����ǿ�����������ļ��ݰ棬����ֻ����Ԫ�ؽڵ�����ݡ�

###��ϰ���룺
```javascript
<script>

window.onload= function(){

var oUl = document.getElementById('ul1');

var i=0;

alert(oUl.childNodes.length)//�����ڻ������ie��360���ȸ趼��ʾ11

/* for(i=0; i<oUl.childNodes.length;i++)

{

//oUl.childNodes[i].style.background == 'red'

if (oUl.childNodes[i].nodeType == 1)

{

oUl.childNodes[i].style.background = 'red';

};

}

alert(oUl.children.length) //children��childNodes�ļ��ݰ�*/

}

</script>

<body>

<ul id="ul1">

<li>111</li>

<li>222</li>

<li>333</li>

<li>444</li>

<li>555</li>

</ul>

</body>
```


###parentNode
```javascript
<script>

window.onload = function(){

var oUl = document.getElementById('ul1');

var i;

var aA = oUl.getElementsByTagName('a');

for(i=0;i< aA.length; i++){

aA[i].onclick = function(){

this.parentNode.style.display = 'none';

}

}

};

</script>

<body>

<ul id="ul1">

<li>jjjj<a href="javascript:;">����</a></li> <!--����a���ӵ������ˢ����������ʾЧ��-->

<li>����j<a href="javascript:">����</a></li>

<li>��<a href="javascript:">����</a></li>

<li>j�õ�j<a href="javascript:">����</a></li>

</ul>

</body>
```
###offsetParent


* offsetParent���Է���һ����������ã���������Ǿ������offsetParent��Ԫ������ģ��ڰ������������ģ����������ѽ��й�CSS��λ������Ԫ�ء� ����������Ԫ��δ����CSS��λ, ��offsetParent���Ե�ȡֵΪ��Ԫ��(�ڱ�׼����ģʽ��ΪhtmlԪ�أ��ڹ������ģʽ��ΪbodyԪ��)�����á� ������Ԫ�ص�style.display ������Ϊ "none"ʱ����ע��IE��Opera���⣩��offsetParent���� ���� null��һ�㷵�������˶�λԪ�صĸ�Ԫ��

```javascript
<body>

<div id="div1" style="width:100px; height:100px;background:red; position:relative;margin:100px;">

<div id="div2" onclick="alert(this.offsetParent.id)" style="width:100px; height:100px;background:yellow; position:absolute; top:100px;left:100px;"></div></div>

</body>
```

###firstChild��firstElementChild


* firstChild�ڻ���ȸ����涼��undefined��������ǰfirstChildֻ����ie�����ã����ڻ�ȡ�׽ڵ���firstElementChild��ͬ������쵽lastChild��lastElementChild

* �����ֵܽڵ㣬nextSibling��nextElementSibling ��previousSibling��previousElementSibling

```javascript
<script type="text/javascript">

window.onload = function(){

var oUl = document.getElementById('ul1');

var i;

//firstChild�ڻ���ȸ����涼��undefined��������ǰfirstChildֻ����ie������

//oUl.firstChild.style.background = 'red';

//��һ��firstElementChild���ݰ�

oUl.firstElementChild.style.background = 'red';

//��ǰ�����һ����ķ���

oFirst = oUl.firstElementChild || oUl.firstChild

oFirst.style.background = 'red'

//ͬ��Ҳ��lastChild �� lastElementChild

//�ֵܽڵ�

var oLi = document.getElementById('li1');

//oLi.nextSibling.style.background = 'red';

oLi.nextElementSibling.style.background = 'red';

//ͬ��nextSibling nextElementSibling��previousSibling previousElementSibling����ͬ�����÷�

}

</script>

<body>

<ul id="ul1">

<li>111</li>

<li>222</li>

<li id="li1">333</li>

<li>444</li>

<li>555</li>

</ul>
```
##DOM��������Ԫ������
***
```javascript   
<script>

window.onload = function(){

oTxt = document.getElementById('txt1');

//1

//oTxt.value = 'asdd';

//2

//oTxt['value'] = 'aswerr';

//3

oTxt.setAttribute("value", "jhgf");

alert(oTxt.getAttribute('id'));

}

</script>

<body>

<input type="text" id="txt1">

</body>
```
##DOMԪ��������

###className

* ͨ��classѡȡԪ�أ�һ���������

1. �����е���Ԫ��ѡ����

2. ��className��Ϊ����ɸѡ��������
```javascript
<script>

//ͨ��class��ѡԪ��

window.onload = function (){

var oUl = document.getElementById('ul1');

var aLi = oUl.getElementsByTagName('li');

var i;

for (var i = 0; i < aLi.length; i++) {

if (aLi[i].className == 'double') {

aLi[i].style.background = 'green';

};

};

}

</script>

<body>

<ul id="ul1">

<li></li>

<li class="double"></li>

<li></li>

<li class="double"></li>

<li></li>

<li class="double"></li>

</ul>

</body>
```
**��һ���Ľ�**��дһ���������ѡclass�Ĺ���
```javacript
//ͨ��class��ѡԪ�صĺ���

function getByClass(oParent, sClass){

var aEle =oParent.getElementsByTagName('*');

var aResult = new Array();

for (var i = 0; i < aEle.length; i++) {

if (aEle[i].className == sClass) { //ע��ǧ��Ҫд�ɡ�sClass������Ϊ����ʱ���Ѿ����С�����

aResult.push(aEle[i]);

};

};

return aResult;

};
```
�������£�
```javascript
window.onload = function (){

var oUl = document.getElementById('ul1');

var aDou = getByClass(oUl,'double');

var i;

for (i = 0; i < aDou.length; i++) {

aDou[i].style.background = 'yellow';

};

}

</script>

<body>

<ul id="ul1">

<li></li>

<li class="double"></li>

<li></li>

<li class="double"></li>

<li></li>

<li class="double"></li>

</ul>

</body>
```
##���������롢��ɾ��Ԫ��
***
###����DOMԪ��

* creatElement����ǩ����  ����һ���ڵ�
* appendChild���ڵ㣩		׷��һ���ڵ�
	-���ӣ�Ϊul����li
	
###����Ԫ��

* insertBefore���ڵ㣬ԭ�нڵ㣩		������Ԫ��ǰ����
	-���ӣ��������li
	
###ɾ��DOMԪ��

* removeChild���ڵ㣩		ɾ��һ��Ԫ��
	-����ɾ��li
	
###��ϰ���룺
�ٴ�������li 
```javascript
<script>
	window.onload = function(){
		oUl = document.getElementById('ul1');
		oBtn = document.getElementById('btn1');
		oTxt = document.getElementById('txt1');

		oBtn.onclick = function(){
			var oLi = document.createElement('li');
			var aLi = oUl.getElementsByTagName('li');

			oLi.innerHTML = oTxt.value;
			if (aLi.length == 0) {
				oUl.appendChild(oLi);
			}else{
				oUl.insertBefore(oLi, aLi[0]);
			}

			
		}
	}
</script>
<body>
	<label for="txt1"></label><input type="text" id="txt1">
	<button id="btn1">����</button>

	<ul id="ul1"></ul>
</body>
```

**ע��**��appendchild(����Ԫ��)�������ӽڵ㣻insertBefore������Ķ���������֮ǰ���룩
	
��ɾ��Ԫ��
```javascript   
<script>
	window.onload = function(){
		var oUl = document.getElementById('ul1');
		var aA = document.getElementsByTagName('a');
		var i;

		for (i = 0; i < aA.length; i++) {
			aA[i].onclick = function (){
				oUl.removeChild(this.parentNode);
			}
		};
	}
</script>
<body>
	<ul id="ul1">
		<li>li1<a href="javascript:;">ɾ��</a></li>
		<li>li2<a href="javascript:;">ɾ��</a></li>
		<li>li3<a href="javascript:;">ɾ��</a></li>
		<li>li4<a href="javascript:;">ɾ��</a></li>
		<li>li5<a href="javascript:;">ɾ��</a></li>
	</ul>
</body>
```
**ע��**��a��ǩ��href����ֵΪJavaScript���� ����Ϊȡ��Ĭ����Ϊ��������תˢ�£��൱һ��js��ڣ���д�Ļ�js����û��Ч����

>**С��**
 ������JavaScript��DOM��document object model����һЩ�����������治�Ǻܹ㣬���˽��������ѿ��Բο�[wschoolDOM�ο��ֲ�](http://www.w3school.com.cn/htmldom/index.asp)�����Ķ�[JavaScript DOM����������ڶ��棩](http://www.javascriptcn.com/read-42.html)һ��


><span style="font-size:12px">���ı���: <a href="{{ permalink }}">  { {title} }  </a>
��������: <a href="http://itxiehui.github.io/">������</a>  
���Э��: <img alt="֪ʶ�������Э��" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/80x15.png" /><a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">?����-������-��ͬ��ʽ���� 4.0</a></span>
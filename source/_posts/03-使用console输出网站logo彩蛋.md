
---
title: 使用console输出网站logo彩蛋【工作日记03】
date: 2019-11-18
tags: []
categories: [web开发,web,工作日记]
---

![我是图片](http://pic1.win4000.com/wallpaper/2017-11-08/5a029f187ca9f.jpg)

<!-- more -->





> 工作日记是笔者记录在日常工作中对负责的前端项目和任务的总结和提炼，在工作中寻乐趣，在代码中找灵魂，输出工作中有价值有意思的沉淀，分享知识，娱乐自己。

> 下文可能存在图片未显示问题，可异步到 [原文](https://juejin.im/post/5dca2e56518825574d2149e5) 阅读

相信大多数年轻人上班时都会偶尔刷下知乎，吃瓜或者提问题，一天我美若天仙的GF发了一张截图过来，说她知乎刷着刷着感觉遇到了bug，于是打开F12，看到了这个彩蛋。

![zhihu-egg](http://b131.photo.store.qq.com/psb?/V12x89qA2LlAEO/piwEkxKgLCg69fWgJGgwSe9E98LHnzpFwrP938F4ufs!/b/dIMAAAAAAAAA&bo=NANnAQAAAAADB3M!&rf=viewer_4)

除了不得不敬佩她的敬业精神外（她也是前端一枚），我肯定是不能涨他人志气，灭自己威风的。我平静的回了句，这玩意太简单，我也能整出来。

编辑器打开，手在键盘上凝固，不知如何敲出第一个代码...

<html>
<center>
    <img src="http://b338.photo.store.qq.com/psb?/V12x89qA2LlAEO/cRggOlfoPJaxWQWwXauWV1aJi57jspW6RmKORJeW8Xk!/b/dFIBAAAAAAAA&bo=lgCWAAAAAAACFzM!&rf=viewer_4"></img>
</center>
</html>



难道要一个个字符的画上去？那得画到什么何年何月？
<!--【惊讶二级】-->
<html>
<center>
    <img src="http://b182.photo.store.qq.com/psb?/V12x89qA2LlAEO/7vb2fhkTRegac9U7pv.iMHmoIf2hqE2x9ZJIYSpvsZw!/b/dLYAAAAAAAAA&bo=8ADwAAAAAAARBzA!&rf=viewer_4"></img>
</center>
</html>

既然自己没那个美术天赋，那就动脑子吧，何况我本身就靠脑子吃饭的呀。

<!--【恍然大悟】-->
<html>
<center>
    <img width="200" src="http://b318.photo.store.qq.com/psb?/V12x89qA2LlAEO/sxtTcevZyQ55Zqb.olsITndQzMmw4oZwwoQmrGrn*O8!/b/dD4BAAAAAAAA&bo=LAEsAQAAAAACFzM!&rf=viewer_4"></img>
</center>
</html>

既然是靠脑子吃饭的，那就得对得起我这聪明的脑子。先问问google大哥如何解决。


一搜，不出我所料，已经有总结文章了。站在巨人的肩膀也不失为一种小聪明。

* [让console充满情怀](hhttps://aotu.io/notes/2016/06/22/An-interesting-experience-on-console/index.html)

<!--【得意】-->
<html>
<center>
    <img width="200" src="http://b182.photo.store.qq.com/psb?/V12x89qA2LlAEO/jj43nXXEcKXqN2wnHhnyHnLuq6OBaIQmVrpngsNZdmw!/b/dLYAAAAAAAAA&bo=8ADwAAAAAAACJwM!&rf=viewer_4"></img>
</center>
</html>

提取出有用的信息：
1. console的方法可以填入占位符，类似c语言的print方法中的占位符。
```
console.log(object [, object, …])
```
1. 占位符

格式占位符 | 作用
---|---
%s | 字符串
%d or %i | 整数
%f | 浮点数
%o | 可展开的DOM
%O | 列出DOM的属性
%c | 根据提供的css样式格式化字符串

3. 字符画  
打印出的logo图案叫字符画，是可以用软件分析图片生成。亲测下面的软件比较好用：
* [ASCII Generator](http://pan.baidu.com/share/link?shareid=3161588673&uk=3509597415)

从上面第二点得知，原来console可以使用css规则格式化字符串，再结合软件将logo生成字符画，这个网站彩蛋不就搞定了吗，如此简单。


具体步骤：
1. 制作字符画  

使用ASCII Generator上传logo图片，生成字符画，调节大小，抖动，然后选择好看的字符，调整好后可以拷贝到IDE了（粘贴到console.info(``)中，否则会出现换行符），如果不是很像可以增删一些字符微调一下。  

这里要注意的是，字符画在IDE里表现会有点变形，可能打印出来也会变形，这是由字体不同造成的，回到软件中将字体调为微软雅黑，再粘贴回去，即使IDE上有点变形但打印出来不会。

但是打印字符画有点单调，加点文字点缀点缀，会碰撞出更美的火花，加上我们团队的队名和标语。

```
console.log(`
                       7007                       
                 7000000000000007                 
            760000000000000000000000057           
         76000008888888888888880800000005         
       700008888868688868888880000008860006       
      800088886888688888888800067      700002     
    70088888888888868888880007         7088000    
   3008868800000088868888005           70888800   
  7008888808000080888888801       76000088888800  
  008868808      608888808      70000000888888806 
 7088688800      608888807     700000888888688800 
 3088868800      808888007             7088868880                 加班熊工作室
 2088686800      608888807             70888888807          
 2088888800      808888007             70888688007          ——国内精品AI语音游戏开创者——
 1088888880      808888807     000000000888688880 
  0088888007     700888007     000000000888888800 
  0088888806      70000007     808888888888888007 
  70086888803       758007     00888686888888005  
   70088000006                 8088686868888002   
    108057878003               808688868888007    
    70007 1  800087            6088888880006      
    00800  7   700000008888888608888800008        
   8088800 77 7080800000000000808000004          
  400000000000800080808000000000000047            
700000000000000000000000000000007                `);
```

2. 加上样式
```
console.log("%cHello,%cWorld", "color:red;", "color:blue;");
```
%c占位符的样式会影响其字后的字符串，样式之间符合从前到后的覆盖逻辑，后面的样式会覆盖前面的样式。

![image](http://b338.photo.store.qq.com/psb?/V12x89qA2LlAEO/grdHiKfwZrCevmZGh8HoFlQ7X7fVWTd4.Ekyhl2zon8!/b/dFIBAAAAAAAA&bo=OQMxAAAAAAADByk!&rf=viewer_4)

那我们就可以在不同位置插入%c占位符，来控制不同元素字符画、队名和标语之间的样式。代码封装成函数：
```
const consoleEgg = () => {
  const egg = `%c
                       7007                       
                 7000000000000007                 
            760000000000000000000000057           
         76000008888888888888880800000005         
       700008888868688868888880000008860006       
      800088886888688888888800067      700002     
    70088888888888868888880007         7088000    
   3008868800000088868888005           70888800   
  7008888808000080888888801       76000088888800  
  008868808      608888808      70000000888888806 
 7088688800      608888807     700000888888688800 
 3088868800      808888007             7088868880                 %c加班熊工作室%c
 2088686800      608888807             70888888807          
 2088888800      808888007             70888688007          ——%c国内精品AI语音游戏开创者%c——
 1088888880      808888807     000000000888688880 
  0088888007     700888007     000000000888888800 
  0088888806      70000007     808888888888888007 
  70086888803       758007     00888686888888005  
   70088000006                 8088686868888002   
    108057878003               808688868888007    
    70007 1  800087            6088888880006      
    00800  7   700000008888888608888800008        
   8088800 77 7080800000000000808000004          
  400000000000800080808000000000000047            
700000000000000000000000000000007                `;
  const defaultStyle = 'color: rgba(0,0,0,.7);';
  const titleStyle = 'color: #fff;background: rgba(52,55,86,1);padding: 2px 10px; border-radius: 2px;font-family: 微软雅黑;';
  const sloganStyle = 'color: #343756';

  console.log(egg, defaultStyle, titleStyle, defaultStyle, sloganStyle, defaultStyle);
};
```

最终效果：
![image](http://b190.photo.store.qq.com/psb?/V12x89qA2LlAEO/MO7ODEnWcPYkDOJMM8VfeO1LROYS.5QE2NzR1qshQTQ!/b/dL4AAAAAAAAA&bo=KgN8AQAAAAADB3Y!&rf=viewer_4)

把图片发给GF，她立马秒变迷妹，发来表情：

<html>
<center>
    <img width="200" src="http://m.qpic.cn/psb?/V12x89qA2LlAEO/nS8f7*bX*osE9cs9U2D61xGHu5goQ8pwLFaHJnJZHNU!/b/dL8AAAAAAAAA&bo=yADIAAAAAAACJwM!&rf=viewer_4"></img>
</center>
</html>

<br>
<br>
<br>
<br>

<center>--------------------------》》》一条华丽的结束线《《《--------------------------
</center>
<br>
<br>
await!await!!await!!!
<center>
    <img width="200" src="http://b190.photo.store.qq.com/psb?/V12x89qA2LlAEO/R3rpwlihgpSoxQnp92BPYp65Iryts4TmYoZemDikX1U!/b/dL4AAAAAAAAA&bo=8ADwAAAAAAARBzA!&rf=viewer_4"></img>
</center>
好奇的小伙伴肯定注意到我们古怪的队名，下面就是我们团队和业务的简单介绍，欢迎关注：


> 加班熊工作室，国内精品AI语音游戏开创者，打造有趣好玩多元化语音交互游戏，在各大智能音箱平台天猫精灵、小度和小度在家、小爱同学上线语音技能数量达10+，积累玩家达一千多万，部分爆款游戏有女生追求手册、约会大作战、恐怖嫌疑人、恐怖陌生人、喜剧之王...赶快上音箱上玩玩吧！

<center>
    <img src="http://b320.photo.store.qq.com/psb?/V12x89qA2LlAEO/rfwmg.Xm7*..V1vL41h84cYLFHoKAtwazaoIvnueRsI!/b/dEABAAAAAAAA&bo=UQHfAAAAAAADB60!&rf=viewer_4"></img>
</center>


<center>--------------------------》》》一条真正的结束线《《《--------------------------
</center>
---
title: 自定义的checkbox,radio样式 
date: 
tags: [html5]
---

**雪碧图**

雪碧图即CSS Sprite，是一种图像合并技术，将网站用到的小图标和背景图都合并在一张图片上，用background-image和background-position来显示需要的图片部份。

**自定义checkbox,radio样式**

Input标签后面跟一个label标签，然后input标签隐藏，通过label标签的for属性，来与input的id对应起来，这样点击label标签，对应的input标签就会被选中。
在该方法下，checkbox，radio的样式就可以通过设置label的背景图来控制，在结合雪碧图，各种各样的样式都可以弄出来。

**思路：**

1.用visibility：hidden将input标签隐藏
2.准备雪碧图，label的大小设置为图标的大小，通过定位显示需要的图片部分
3.使用：checked伪类选择器改变label的背景图定位，定位为选中的图标

**遇到的问题：**

1.使用伪元素方法时，：after伪类给元素添加content的时候，要给设置定位，：after的样式相对label定位，使两者置于不同的文本流，避免出现节点相拥挤的浮动问题。
2. 样式 .example2 input[type='checkbox']:checked + label{} 要在样式.example2 input[type='checkbox']:checked + label:after之前，否则不起作用。

**雪碧图与伪元素方式对比：**
1.用雪碧图的的方式，虽然在减少网络请求优化性能方面有明显作用外，可是一个小图标的大小应该比几十行的伪元素代码大一点，对文件大小方面可能不利。优点可能就是使用图片可以做出很美观的效果。
2.使用伪类元素的方式就相对复杂一下，遇到的问题也会比较多。但结合css3可以做出很简洁美观的效果。

**总结**
这两种方法各有优劣，采用哪种方式还是要按照项目的具体要求吧，不能一概而论。比如做移动端的话，个人感觉使用后者会计较好，因为请求一张雪碧图的流量会很大，消耗流量。而在pc端的网站，考虑前者也是一个不错的选择。
---
title: node开发公众号踩过的坑 
date: 2017-06-13
tags: [nodejs,bug]
---


> 1

**错误**：
```js
request = Promise.promisify( require( ‘request’ ) )
```
调用
```js
return new Promise(function(resolve, reject){
       request({url: url, json: true} ).then (function ( response ) {
          
                var data = response[1];
                var now = (new Date().getTime());
                var expires_in = now + (data.expires_in - 20) * 1000;
                data.expires_in = expires_in;
                resolve(data);
         
        })
  
   })
```

	
	**报错**：属性expires undefined
	
	本来以为request promise化后可以用then函数，所以一开始没有怀疑then的使用

	分析：expires没有定义，那么就跟data有关，data与response有关，假如没有获取的话，就会出错。那么，response是由request发请求后相应回来的数据，有理由怀疑是request使用的问题。
	分析报错信息，一步步追根溯源，才能找出问题所在。

**解决**：
Request的使用改为：

```js
return new Promise(function(resolve, reject){
       request({url: url, json: true}, function (error, response, body) {
            if (!error && response.statusCode === 200) {
                var data = body;
                var now = (new Date().getTime());
                var expires_in = now + (data.expires_in - 20) * 1000;
                data.expires_in = expires_in;
                resolve(data);
                console.log(data);
            } else {
                reject()
            }
        });     
   })
```

没有使用promise化的then函数


> 2

**问题**：客户端收不到微信服务器发来的消息，并显示”该公众号暂时无法提供服务“
为什么会这样呢？
第一步推断：假如微信服务器没有在5秒内接收到服务器发来的消息，会自动断开连接，重发请求，重复三次。所以可能是服务器或者代码程序的问题
进一步推断： 可是程序没有报错，昨天微信服务器是可以接受到第三方服务器的消息的。
反复的检查：发现发送过去的小xml格式如下：
<xml>
<ToUserName><![CDATA[  o1PAWszjPU3zudiu7yxmNg7hLp00 ]]></ToUserName>
 <FromUserName><![CDATA[  gh_4a832c2991ae ]]></FromUserName>
</xml>

![CDATA][ ]里的内容是无法被服务器解析的，可是有一点很显眼，【】里面userId号的前面多了个空格，难道它连空格也不去除直接导致userId不正确，找不到用户吗？？
二话不说，直接测试一下….
丫的，还果真如此！哈哈，xml格式还真TM的严格
以后一定要注意![CDATA][ ]里首字母不能有空格！



> 3

**错误**：上传永久素材时，回复客户端是出错，“改公众号无法提供服务”
**解决**：不一定是代码的问题，这是由于微信服务器在这一块不稳定，再试多一次就好了

---
title: Web离线应用
date: 2017-06-13
tags: [ApplicationCache,localStorage,webSQL,闭包]
---

## 前言

编写web离线应用，需要把应用文件和数据存在本地，也就是浏览器中。这主要用到了H5的三个重要的知识点，Application Cache(应用程序缓存), localStorage/sessionStorage(web存储), web SQL(web数据库)。

## Application cache

如果要启用应用缓存，就需要在html标签加manifest属性
```html
<!DOCTPYE HTML>
<html manifest="demo.appcache">
...
</html>
```
demo.appcache就是写需要缓存页面信息的manifest文件，文件的后缀名可以任写，但建议统一为.appcache。  
### manifest文件
告知浏览器需要（或不需要）缓存的文件，包括三部分：  
* **CACHE MANIFEST** 此标题下出现的文件将在首次下载后进行缓存
* **NETWORK** 此标题下列出的文件需要在访问服务器，且不被缓存
* **FALLBACK** 此标题下列出的文件是页面无法被访问时退回的页面

第一行写CACHE MANIFEST是必须的，写法为
```
CACHE MANIFEST
# 2017-06-10 v1.0.0
index.html 
jqury.min.js
```
#后面的是注释，写上日期和版本号，当index.html等页面有更新时，改日期或者版本号，浏览器会将页面进行更新和缓存，方便。其中能应用默访问的页面，也就是index.html是默认被缓存的，不写也可以。**一般写修改不频繁的文件**

```
NETWORK:
*
```
一般写星号，也就是其他的页面都要通过访问服务器

```
FALLBACK:
/html/ /offline.html
```
如果无法建立因特网连接，则用 "offline.html" 替代 /html5/ 目录中的所有文件。

### 更新缓存的情况
* 用户清空浏览器缓存
* manifest文件被修改
* 由程序文件来更新应用缓存

### 设置MIME-type
需要给manifest文件设置MIME-type为text/cache-manifest
* 在Apache服务器
可以在根目录下添加.htaccess文件
```
AddType text/cache-manifest manifest
```
或者在manifest文件开头添加（前提是文件是php后缀名）
```
<?php
header("Content-Type: text/cache-manifest");
?>
```
==貌似使用nodejs做后台不用设置MIME-type也可以将程序文件写进缓存==

## localStorage/sessionStorage
在浏览器上（本地）储存用户的浏览数据
两者的区别就在于，localStorage储存的数据没有时间的限制，sessionStorage储存的数据，当关闭浏览器时，数据就没了。

两者的API都是相同的，主要有一下：
* 保存数据：localStorage.setItem(key,value);
* 读取数据：localStorage.getItem(key);
* 删除单个数据：localStorage.removeItem(key);
* 删除所有数据：localStorage.clear();
* 得到某个索引的key：localStorage.key(index);
保存数据还可以直接定义localStorage的属性，比如localStorage.resource = value  
**一般储存变更频繁的文件**


## web SQL
位于浏览器中的关系型数据库
核心的方法就三个：
* openDatabase：这个方法使用现有数据库或创建新数据库创建数据库对象。

* transaction：这个方法允许我们根据情况控制事务提交或回滚，意思是可以划分事务。

* executeSql：这个方法用于执行真实的SQL查询。

### 打开数据库
```
var db = openDatabase('mydb', '1.0', 'Test DB', 2 * 1024 * 1024);
```
openDatebase方法接受五个参数，分别是：
* 数据名称
* 数据库版本（写死就行）
* 数据库描述
* 大小
* 回调（可选）

### 插入和读取数据
```
var db = openDatabase('mydb', '1.0', 'Test DB', 2 * 1024 * 1024);
 
db.transaction(function (tx) {
   tx.executeSql('CREATE TABLE IF NOT EXISTS LOGS (id unique, log)');
   tx.executeSql('INSERT INTO LOGS (id, log) VALUES (1, "菜鸟教程")');
   tx.executeSql('INSERT INTO LOGS (id, log) VALUES (2, "www.runoob.com")');
});
 
db.transaction(function (tx) {
   tx.executeSql('SELECT * FROM LOGS', [], function (tx, results) {
      var len = results.rows.length, i;
      msg = "<p>查询记录条数: " + len + "</p>";
      document.querySelector('#status').innerHTML +=  msg;
    
      for (i = 0; i < len; i++){
         alert(results.rows.item(i).log );
      }
    
   }, null);
});
```

## 采用MVC思想写js代码
* applicationcontroller.js //程序的入口程序主控制器，向外暴露一个start方法，作为程序的入口。其他的私有函数都是对次控制器暴露出的方法的调用

* articlescontroller.js //操作articles模型，调用article暴露出的方法实现增删改查以及调用template的方法渲染页面

* article.js //article模型，调用更底层的database对象（对数据库操作的封装）

* database.js //更底层的M层，封装对数据库的操作

* templates.js //V层，插入html节点

程序的入口就只有一个
```
//applicationController.js
//webapp的入口函数，类似C语言中的main，或者jq中的$(document).ready
    function start(resources, storeResources) {
        APP.database.open(function() {
            //监听hash的变化
            $(window).bind("hashchange", route);

            //往DOM里添加CSS
            $("head").append('<style>' +　resources.css + '</style>');

            //创造app应用名称节点
            $('body').html(APP.templates.application());

            //移除下载提示
            $('#loading').remove();

            route();
        });

        if(storeResources) {
            localStorage.resources = JSON.stringify(resources);
        }
    }
```
在index.html调用
```JavaScript
$(document).ready(function() {
            console.log('ready %o', new Date()); //%o代替javascript对象

            var APP_START_FAILED = "I'm sorry, the app can't start right now."

            function startWithResources(resources, storeResources) {
                //执行加载的js函数
                try {
                    //eval(resources.js)
                    insertScript(resources.js);
                    setTimeout(function() {
                        APP.applicationController.start(resources, storeResources);  //程序入口
                    }, 500); 

                } catch(e) {
                    alert(APP_START_FAILED);
                    console.log('%o', e);
                } 
            }

            function startWithOnlineResources(resources) {
                startWithResources(resources, true);
            }

            function startWithOfflineResources() {
                var resources;

                //假如之前已经访问了并且js文件已经缓存进localStorage，执行以下
                if(localStorage && localStorage.resources) {
                    resources = JSON.parse(localStorage.resources);
                    startWithResources(resources, false);

                    //否则输出提醒信息
                }else {
                    alert(APP_START_FAILED);
                }
            }
            
            function insertScript(script) {
                var node = document.createElement('script');
                node.innerHTML = script;
                document.head.appendChild(node);
            }

            //假如设备离线，则执行离线操作
            if(navigator && navigator.onLine === false) {
                startWithOfflineResources();

                //否则，下载资源并执行，假如成功就把资源添加进local storage。
            }else {
                $.ajax({
                    url: 'api/resources',
                    success: startWithOnlineResources,
                    error: startWithOfflineResources,
                    dataType: 'json'
                });
            }

            
        })
```
程序的设计就类似于树形的结构：

```
graph TD
    A[程序入口]-->B[主控制器]
    B-->C[各种次控制器]
    C-->D[模型]
    C-->E[视图]
    B-->E
```
主控制器的第一步工作是打开数据库

### 对象的封装
全局就一个APP对象，为window的全局变量
```
window.APP = {};
(function(APP){
    APP.applicationController = (function(){
        ...
        
        return {
            start: start
        }
    }());
    
    APP.articleController = (funcion(){
       ...
       
       return {
            synchronizeWithServer:　synchronizeWithServer,
            showArticle: showArticle,
            showArticleList: showArticleList
        }
    }());
    
    
    
    APP.templates = (function(){}(
        ...
        
        return {
            application: application,
            home: home,
            articleList: articleList,
            article: article,
            articleLoading:　articleLoading
        }
        
    )()};
    
    APP.database = (function(){}(
        ...
        
        return {
            open: open,
            runQuery: runQuery
        }
    )()};
    
    APP.article = (function(){}(
        ...
        
        return {
            deleteArticles: deleteArticles,
            insertArticles: insertArticles,
            selectBasicArticles: selectBasicArticles,
            selectFullArticle: selectFullArticle
        }
    )()};
}(APP))
```
这里运用的JS的闭包思想，每一个(function(){...}()()};都是单独的作用域，代码不会被污染，全局就只有一个对象APP.
 






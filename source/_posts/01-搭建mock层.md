---
title: 搭建mock层【工作日记01】
date: 2019-9-10
tags: []
categories: [前端,工作日记]
---

![我是图片](http://pic1.win4000.com/wallpaper/2017-11-08/5a029eda4cc75.jpg)

<!-- more -->

> 什么是mock？mock指的是模拟。

在软件工程中当开发工作不同步情况下为了不断工作链，而模拟其中一环节让工作得以持续下去，符合这种情况的工作我们就叫mock。当然我们常说的mock是指模拟网络请求前后端通讯。

前端和后端的协同工作中，为了不串行工作流加快开发效率，往往需要制定协议，而前端只需要后端提供数据接口，所以这份协议就是接口协议。根据这份协议前端就可以mock模仿数据，达到前后端并行开发。

mock可以分为以下几种方式：
1. 侵入业务代码，设置开关
2. 利用webpack构建工具拦截请求
3. 搭建mock服务层代理响应
4. 利用浏览器插件拦截请求


cms-web是由vue-cli3脚手架搭建的项目，依赖webpack打包工具，可以利用devServer.defore api对请求拦截处理，返回mock数据，引入mock层。

所以选择上面第二种方式，原因总结为：**无侵入，简单快捷易上手**。

## mockjs
借助mockjs实现mock数据逻辑，以下是最简单的实现：
```  
var Mock = require('mockjs');
...
before(app, server) {
    app.get('/api/action/game/list', function (req, res, next) {
      var data = Mock.mock({
        code: 0,
        data: {
          'totalCount|1-100': 1,
          'datas|1-10': [{
            'id|+1': 1,
            'dataName|1-5': 'name',
            'dataType|1-2': 1,
            'dataScope': '1,100',
            'defaultValue|1-5': 'value',
            'dataStatus|0-1': 0,
            'remark|1-5': 'str',
            'record|0-1': 0,
            'series|1-3': [{
              'id|+1': 1,
              'type|1-2': 2,
              'name|1-2': '小云熊奇妙故事',
              'mark': null,
              'status': 0
            }]
          }]
        }
      })
    
      res.json(data)
    })
}
```

这里我们简单介绍一下mockjs的使用规范，详细移至[mockjs文档](https://github.com/nuysoft/Mock/wiki/Syntax-Specification)

数据模板中属性主要由三个部分构成：属性名，生成规则和属性值
```
'属性名|生成规则': 属性值
```
以'dataName|1-5': 'name'为例，dataName是属性值，name是属性值，1-5是生成规则，对应String类型属性代表重复随机次数1到5次，假如对应Number类型代表生成随机数的最小值1和最大值5。'id|+1': 1表示id值会从开始叠加。

## 优化mock层
话题回来mock层搭建的迭代优化，首先分析上面实现存在的问题：
- 所有的请求都会经过这一层，mock层匹配到请求就会被拦截，需要设置开关
- 配置的请求和数据模板会越来越多，逻辑与配置混合会导致代码越来越臃肿，逻辑和配置要分离

把mock层从before函数中抽离，并设置开关机制：
```
var Mock = require('mockjs');

class MockHandler {
  constructor() {}
  
  handle(app, server) {
    app.get('/api/action/game/list', function (req, res, next) {
      if (!!process.env.mock || !!req.query._is_mock) {
        var data = Mock.mock({
          code: 0,
          data: {
            'totalCount|1-100': 1,
            'datas|1-10': [{
              'id|+1': 1,
              'dataName|1-5': 'name',
              'dataType|1-2': 1,
              'dataScope': '1,100',
              'defaultValue|1-5': 'value',
              'dataStatus|0-1': 0,
              'remark|1-5': 'str',
              'record|0-1': 0,
              'series|1-3': [{
                'id|+1': 1,
                'type|1-2': 2,
                'name|1-2': '小云熊奇妙故事',
                'mark': null,
                'status': 0
              }]
            }]
          }
        });
        res.json(data);
      } else {
        next();
      }
    })
  }
}

module.exports = new MockHandler(config);
```
设置了两个并联开关，分别设置环境变量mock或者往请求查询参数加入_is_mock变量都可以通过验证。

逻辑与配置分离：
```
const config = {
    '/api/action/game/list': {
      code: 0,
      data: {
        'totalCount|1-100': 1,
        'datas|1-10': [{
          'id|+1': 1,
          'dataName|1-5': 'name',
          'dataType|1-2': 1,
          'dataScope': '1,100',
          'defaultValue|1-5': 'value',
          'dataStatus|0-1': 0,
          'remark|1-5': 'str',
          'record|0-1': 0,
          'series|1-3': [{
            'id|+1': 1,
            'type|1-2': 2,
            'name|1-2': '小云熊奇妙故事',
            'mark': null,
            'status': 0
          }]
        }]
      }
    }
}

class MockHandler {
  constructor(config) {
      this.config = config
  }
  
  handle(app, server) {
    Object.keys(this.config).forEach(route => {
        app.get(route, function (req, res, next) {
            if (!!process.env.mock || !!req.query._is_mock) {
                var data = Mock.mock({
                  code: 0,
                  data: {
                    'totalCount|1-100': 1,
                    'datas|1-10': [{
                      'id|+1': 1,
                      'dataName|1-5': 'name',
                      'dataType|1-2': 1,
                      'dataScope': '1,100',
                      'defaultValue|1-5': 'value',
                      'dataStatus|0-1': 0,
                      'remark|1-5': 'str',
                      'record|0-1': 0,
                      'series|1-3': [{
                        'id|+1': 1,
                        'type|1-2': 2,
                        'name|1-2': '小云熊奇妙故事',
                        'mark': null,
                        'status': 0
                      }]
                    }]
                  }
                })
                res.json(data);
            } else {
                next();
            }
        })
    })
  }
}
```
革命尚未成功，旧的问题的解决了，但同时又出现的新的问题：
- 只支持get请求，不支持post请求
- get方法的回调函数中重复了很多次对mock开关条件的判断，需要提取出来

针对第一个问题，只需要将get/post请求方法配置到route就好了，本身属于配置逻辑的范围之内。而第二个问题，可以提炼函数，利用函数式编程，利用高阶函数返回回调函数，这样代码会简洁很多，具体优化如下：
```
const config = {
    'GET /api/action/game/list': {
      code: 0,
      data: {
        'totalCount|1-100': 1,
        'datas|1-10': [{
          'id|+1': 1,
          'dataName|1-5': 'name',
          'dataType|1-2': 1,
          'dataScope': '1,100',
          'defaultValue|1-5': 'value',
          'dataStatus|0-1': 0,
          'remark|1-5': 'str',
          'record|0-1': 0,
          'series|1-3': [{
            'id|+1': 1,
            'type|1-2': 2,
            'name|1-2': '小云熊奇妙故事',
            'mark': null,
            'status': 0
          }]
        }]
      }
    }
}

class MockHandler {
  constructor(config) {
    this.init(config);

    this.config = config;
  }
  init(config) {
    Object.keys(config).forEach(route => {
      const model = config[route];

      function callbackWarpper(model) {
        return function (req, res, next) {
          if (isMock || !!req.query._is_mock) {
            var data = Mock.mock({
              code: 0,
              data: model
            });
            res.json(data);
          } else {
            next();
          }
        };
      }
      config[route] = callbackWarpper(model);
    });
  }
  handle(app, server) {
    for (let route in this.config) {
      let [method, path] = route.split(' ')
      let reg = new RegExp(path);
      let callback = this.config[route];
      app[method.toLowerCase()](reg, callback);
    }
  }
}
```

写到这里本以为自己写出了一手漂亮的代码，但是突然想起，app实例上有个use()可以用来忽略请求方法get/post，然后在内部对请求做匹配处理，这样代码会更加的精简和容易被理解：
```
const isMock = !!process.env.mock;
const config = {
    '/api/action/game/list': {
      code: 0,
      data: {
        'totalCount|1-100': 1,
        'datas|1-10': [{
          'id|+1': 1,
          'dataName|1-5': 'name',
          'dataType|1-2': 1,
          'dataScope': '1,100',
          'defaultValue|1-5': 'value',
          'dataStatus|0-1': 0,
          'remark|1-5': 'str',
          'record|0-1': 0,
          'series|1-3': [{
            'id|+1': 1,
            'type|1-2': 2,
            'name|1-2': '小云熊奇妙故事',
            'mark': null,
            'status': 0
          }]
        }]
      }
    }
}

class MockHandler {
  constructor(config) {
    this.config = config;
  }
  handle(app, server) {
    app.use((req, res, next) => {
      if (isMock || !!req.query._is_mock) {
        Object.keys(this.config).forEach(route => {
          const reg = new RegExp(route)
          const model = this.config[route]
          if (reg.test(req.path)) {
            var data = Mock.mock({
              code: 0,
              data: model
            });
            return res.json(data);
          }
        })
        next()
      } else {
        next()
      }
    })
  }
}
```

这个方案原理是利用webpack devServer在发送请求时候做了一层拦截，请求并没有经过真实的网络环境，但是利用webpack devServer其实有个弊端，当我想增加一条记录时，需要再次重启webpack，本身属于对webpack配置的修改，除利用node watch功能监听文件改动运行脚本重启的方法外，几乎想不出有更好的方法了，这个痛点用一个名词来形容就是不支持热插拔。

## 浏览器插件
有没有不涉及到编程的mock方案呢？有，**利用浏览器插件拦截请求或篡改响应。**

这种方案属于利用工具，不用在代码层面引入mock层也可以拦击请求，好处是相对前面介绍的方案，更加的具有普适性，可以让测试小伙伴在后端接口发生异常的情况下依然畅通无阻的测试前端功能。

这样的插件我推荐： **Ajax Interceptor** [下载](https://chrome.google.com/webstore/detail/ajax-interceptor/nhpjggchkhnlbgdfcbgpdpkifemomkpg) [源代码](https://github.com/YGYOOO/ajax-interceptor)

## 工具
不想用浏览器插件还有其他的工具吗？当然有，就是我们熟悉的postman，详细请参考下面文章：

[Postman高级应用（11）：可以开始对接了吗——Mock服务](https://blog.csdn.net/qq598535550/article/details/87924629)


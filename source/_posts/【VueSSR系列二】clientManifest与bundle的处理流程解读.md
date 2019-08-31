---
title: 【VueSSR系列二】clientManifest与bundle的处理流程解读
date: 2019-07-20 19:02:01
tags: [vue]
categories: [前端]
---

![我是图片](https://cdn.pixabay.com/photo/2016/03/26/13/09/cup-of-coffee-1280537__340.jpg)

<!-- more -->

上一节讨论了VueSSR的构建流程，构建出来的clientManifest和serverBundle最终会被转换成html，这一节我们深入vue-server-renderer的核心内容，看看它们都经过了哪些的处理。这一节的内容包括：
- 使用BundleRenderer的原因
- 服务端渲染的大体原理
- 输出html正文过程
- 预加载与预取资源

阅读源码先整体查看写下代码文件结构和入口文件暴露的接口，然后运行一段demo断点来跟踪代码处理数据的细节，下面将以这段demo作为阅读代码的入口：

![image](http://m.qpic.cn/psb?/V12x89qA2LlAEO/SucTp5cC08NnxOrUu6IyCLb8OcB0vE7YqLIWyDYTheY!/b/dLYAAAAAAAAA&bo=yAXeAsgF3gIDByI!&rf=viewer_4)


## 使用BundleRenderer的原因
`vue-server-renderer`提供了两个API，createRenderer和createBundleRender。它们的用法一样，如果你阅读了源码你会知道createBundleRenderer其实是在createRenderer上做了扩展，以提供官网上所说的下面几种特性：
- 支持source map
- 热重载 
- 关键css注入
- 资源注入
使用基本SSR（createRenderer）有一个问题，当源代码更新后需要重启服务器以获取最新代码，所以需要引入热重载。官方的做法就是通过webpack自定义插件将server bundle生成可传入renderer处理的特殊JSON文件，createBundleRender需要具备处理JSON文件能力以便支持热重载。当这一章不打算阐述SSr如何支持热重载，我们先来看看SSR最终是怎样输出html的。

## 服务端渲染的大体原理
源码看到最后，我发现了一段可以可以大致的理解它的脉络的代码：
```javascript
// templateRenderer.render方法内
if (this.inject) {
    return (
        template.head(context) +
        (context.head || '') +
        this.renderResourceHints(context) +
        this.renderStyles(context) +
        template.neck(context) +
        content +
        this.renderState(context) +
        this.renderScripts(context) +
        template.tail(context)
    )
} else {
    return (
        template.head(context) +
        template.neck(context) +
        content +
        template.tail(context)
    )
}

```
转换成图来展示：

![html分段图](http://m.qpic.cn/psb?/V12x89qA2LlAEO/yVo.O8FVlWuISuWAm2H4v27hL5ASz9zXsgZoXNZs5hg!/b/dIQAAAAAAAAA&bo=mwNlApsDZQIDFzI!&rf=viewer_4)

原来bundleRenderer以字符串拼接的方式将html的片段组合成整个html文档，整个html文档会划分为几个部分，这里只列出主要的部分，分别是头部、资源预加载和预取资源、inlineStyle、正文、state、script。这几个主要字符片段跟辅助的片段结合就结合成了输出的服务端渲染的html内容。

前面我们已经知道clientManifest的作用是记录文档资源文件的信息，bundleRenderer利用clientManifest的信息，推理出需要预加载和预取的资源和首屏加载的js资源文件，然后拼接成资源预加载和预取片段和script片段。而server bundle中记录中编译后的源代码，这正是html文档中正文的内容来源。

## 输出html正文过程
那么server bundle是如何被处理成正文的呢？我将bundle的处理过程中的关键步骤画成了流程图，如下：
![vueSSR处理bundle生成正文流程](http://m.qpic.cn/psb?/V12x89qA2LlAEO/whA77ZoC2nwXNlTLdFS5Oo.6f*wotTd.oAOU.Kvsntg!/b/dL8AAAAAAAAA&bo=9AJTA*QCUwMRBzA!&rf=viewer_4)

当执行createBundleRunner()时，在内部会执行compileModule()，生成一个处理编译后源码的函数evaluate。evaluate函数会将编译后文件源码包装成module对象，然后返回module.exports.defualt，它就是封装了文件源码的函数，执行这个函数就就相当于执行文件源码。当这个文件是入口文件时，返回的就是entry入口文件源码的封装函数，也就是runner，那么执行runner(context)相当于执行entry-server.js导出的函数，如下。
```JavaScript
export default context => {
  return new Promise((resolve, reject) => {
    const { app, router, store } = createApp(context.url)
    ...
    resolve(app)
    ...
  })
}
```

app就是执行该函数后，Promise状态为fulfilled时往下传递的单页应用的vue根实例。之后app会传入renderToString方法，该方法内会调用render函数，递归根实例中的每个子组件对象，渲染每个子组件的template然后组装template，最终生成html中的正文片段content。

render函数内其实是执行了vue内部的render函数，执行组件的生命钩子函数，生成虚拟dom节点，只不过最后转化成了template字符串返回。最后templateRenderer.render方法将正文和其他文档片段组件成整个html文档返回。

## 预加载与预取资源
bundle Renderer如何推断出预加载和预取资源的呢？

我们现在已经知道html是在templateRenderer.render方法中组合的，在它里面有这么一句`this.renderResourceHints(context)`，预加载和预取片段就是由它来生成的。以下是renderResourceHints方法的流程图：
![vueSSR预加载和预取片段生成生成流程图](http://m.qpic.cn/psb?/V12x89qA2LlAEO/2Xwnp89zmoTY0t9VTszfYE8SFOtSogJ8e80RwTm6*EA!/b/dLgAAAAAAAAA&bo=sgM7A7IDOwMDByI!&rf=viewer_4)

从图中我们很清晰得知道renderPreloadLinks方法和renderPrefetchLinks方法都调用了getUsedAsyncFiles方法，来拿到组件实例中所依赖到的代码chunk文件列表usedAsyncFiles，它是通过context._registeredComponents得到组件实例依赖的moduleIds，再根据clientManifest中的modules对象（记录modules之间的依赖关系）和all对象（记录着所有编译后的文件），找出moduleIds所对应的文件，这些文件就是初始渲染组件所依赖到的代码 chunk文件，也就是usedAsyncFiles。

![资源加载文件](http://m.qpic.cn/psb?/V12x89qA2LlAEO/zC5fWQVzuwuJjvm3lOThbmfEod0T0.0GIm3LJYchj5E!/b/dL8AAAAAAAAA&bo=5AMJAeQDCQEDByI!&rf=viewer_4)

usedAsyncFiles与preloadFiles（clientManifest中的initial文件数组）合并就是需要预加载的资源列表，usedAsyncFiles与prefetchFiles（clientManifest中的async文件数组）的差集就是预取的资源列表。

## 小结
- 服务端渲染思路：将html划分片段，头部、资源预加载和预取资源、inlineStyle、正文、state、script和其他辅助衔接片段，推断生成这些片段然后组装成整一个html文档。
- 生成正文：将bundle中的编译后代码字符串包装成一个可执行的模块，运行模块得到应用实例app（vue根实例），递归渲染app中的子组件，将data与template组合，最后组装成正文部分。
- clientManifest中记录着资源加载信息，通过运行app得到context对象中_registedComponents拿到moduleIds，然后得到usedAsyncFiles（组件依赖的文件）。其与preloadFiles（clientManifest中的initial文件数组）的并集就是初始渲染的预加载的资源列表，与prefetchFiles（clientManifest中的async文件数组）的差集就是预取的资源列表。




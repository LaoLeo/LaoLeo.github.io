---
title: 【VueSSR系列一】构建流程源码解读
date: 2019-06-13 21:20:01
tags: [vue]
categories: [前端]
---

![首图](https://pixabay.com/photos/home-office-workstation-office-336373/)

<!-- more -->

# 【VueSSR系列一】构建流程源码解读

最近接手维护一个使用VueSSR搭建的项目，于是乎对VueSSR做了一系列的学习和研究。本文以VueSSR所依赖的第三方包`vue-server-renderer@2.6.10`进行讲解，内容包括：
- VueSSR构建流程
- VueSSRClientPlugin
- VueSSRServerPlugin
<!--- 服务端渲染的大体原理-->
<!--- 输出html正文过程-->
<!--- 预加载与预取资源-->
<!--2. 数据流-->
<!--    - 服务端处理请求-->
<!--    - 客户端激活-->

希望本文对你有帮助。

## VueSSR构建流程
`vue-server-renderer`是VueSSR的核心内容，主要作用是在服务端将vue应用程序渲染输出html，同时也支持更高级的特性，例如：结合模板文件，构建clientManifest和server bundle来做到更细粒度的把控，自动注入（资源预加载、关键css）、嵌入vuex状态等特性。为了使用这些高级特性，我们先看一下如何构建出clientManifest和server bundle文件。

简单用图表述了VueSSR的构建流程，如下图展示：

![VueSSR构建流程图](http://m.qpic.cn/psb?/V12x89qA2LlAEO/Kc*lvVVyE6.jGWuENdHZ1NoSFgly4xSG*7r3a3s0VDY!/b/dAgBAAAAAAAA&bo=jgTAAY4EwAEDByI!&rf=viewer_4)

如图构建过程中有两个入口文件，通过`vue-server-renderer`提供的webpack插件`VueSSRClientPlugin`和`VueSSRServerPlugin`，分别构建打包出供客户端和服务端使用的json文件。

## VueSSRClientPlugin
`vue-ssr-client-manifest.json`就是通过`VueSSRClientPlugin`生成的clientManifest文件，称为客户端构建清单。咱们来看看它的内容：
```json
{
    "publicPath": "/dist/",
    "all": ["static/img/d_icon.522f4d11.png"...],
    "initial": ["manifest.0338f6f2cf8625ed7117.js"...],
    "async": ["0.51ec59b64fde13191f79.js"...],
    "modules": {...}
}
```
它将输入到renderer中，为模板的资源加载提供信息，推断和注入资源预加载和数据预取指令和首次渲染需要加载script标签。

深入它的源码，我们看看这个插件是怎样生成这些数据的。
```javascript
// vue-server-renderer/client-plugin.js

var VueSSRClientPlugin = function VueSSRClientPlugin (options) {
  if ( options === void 0 ) options = {};

  this.options = Object.assign({
    filename: 'vue-ssr-client-manifest.json' // 默认文件名
  }, options);
};

VueSSRClientPlugin.prototype.apply = function apply (compiler) {
    ...
};

module.exports = VueSSRClientPlugin;
```
从结构来看，作为webpack插件，VueSSRClientPlugin只不过是一个原型上挂载apply方法的构造函数，核心逻辑就在apply方法中。
```javascript
// vue-server-renderer/client-plugin.js

var onEmit = function (compiler, name, hook) {
  if (compiler.hooks) {
    // Webpack >= 4.0.0
    compiler.hooks.emit.tapAsync(name, hook);
  } else {
    // Webpack < 4.0.0
    compiler.plugin('emit', hook);
  }
};

VueSSRClientPlugin.prototype.apply = function apply (compiler) {
    var this$1 = this;

    onEmit(compiler, 'vue-client-plugin', function (compilation, cb) {
      ...
    })
};
```
在apply方法中，有这样一段我觉得挺有意思的，就是将兼容逻辑的判断语句都封装到onEmit函数中，这样代码的层次感比较清晰，值得我们去学习，将代码根据功能来分层。我们重点关注下onEmit的回调函数，是如何获取和处理数据输出json文件的。下图我用了伪代码来展示该逻辑的处理流程：

![clientManifest处理流程图](http://m.qpic.cn/psb?/V12x89qA2LlAEO/3kRBJQYHa.05qmZ9VEJ0u9G3s9Q3CcrRuqo48ZsG2qk!/b/dL4AAAAAAAAA&bo=PgT2Az4E9gMRBzA!&rf=viewer_4)

简单解析下上面的流程：
1. 获取编译对象中的包含编译信息的stats对象
2. 从stats.assets得到应用用到所有资源数组allFiles（元素是文件名）
3. 从入口文件对象中找出初次渲染加载的js和css文件数组initialFiles
4. allFiles与initialFiles的差集，过滤出js和css，作为可异步加载资源asyncFiles
5. 分别从stats.modules找出每个模块对应的chuck代码文件，在将chunk依赖文件转换为allFiles中的indexArray，然后moduleId做key，indexArray作为value组成map对象。
6. 最后组装成json对象，序列化后挂上compilation.assets对象上，最为json文件输出。

## VueSSRServerPlugin
vue服务端构建插件做的事情跟客户端构建插件一样，都是输出一个json文件，源码中的逻辑几乎也一样，不一样的地方在于输出的文件内容：
```javascript
var bundle = {
  entry: entry,
  files: {},
  maps: {}
};

stats.assets.forEach(function (asset) {
  if (isJS(asset.name)) {
    bundle.files[asset.name] = compilation.assets[asset.name].source();
  } else if (asset.name.match(/\.js\.map$/)) {
    bundle.maps[asset.name.replace(/\.map$/, '')] = JSON.parse(compilation.assets[asset.name].source());
  }
  // do not emit anything else for server
  // 删除编译信息中的源代码的文件信息
  // 保证只输出一个文件vue-ssr-server-bundle.json
  delete compilation.assets[asset.name];
});
```
从json文件的内容来看，files属性包含了源代码编译后的所有内容，这些内容将用于服务端渲染输出html的正文。maps字段记录sourceMap文件信息，用于提供sourceMap支持。

在构建流程中我们知道源代码会经过两次编译，在客户端构建完的时候已经输入打包后的文件了，假如服务端侯建再输出就会重复，造成浪费，所以需要删除`compilation.assets`上所有的文件信息。保证最后只输出一个json文件。


值得注意的是这个插件只允许一个入口js文件，为了避免使用了CommonsChunkPlugin分包导致文件内容分散。因为server bundle文件的作用主要是为了记录编译后的源代码内容，以便后续能在服务端渲染中运行。
```javascript
if (entryAssets.length > 1) {
  throw new Error(
    "Server-side bundle should have one single entry file. " +
    "Avoid using CommonsChunkPlugin in the server config."
  )
}
```

## 总结
构建出clientManifest和server bundle两个json文件主要是为了让vue-server-renderer支持一些高级特性，比如自动注入（预加载和预取，inlineStyle关键css）、state等，已达到对输出html更细粒度的控制能力。

他们的作用分别是：
- clientManifest会为文档的资源加载提供信息，推断和注入资源预加载和数据预取指令和首次渲染需要加载script标签。
- bundle包含了源代码编译后的所有内容，将用于服务端渲染输出html的正文，已经提供sourceMap支持。

> 高手都是从表面看到本质的人

这里面涉及到的知识点主要是webpack插件和compilation对象，可以点击[这里](https://webpack.docschina.org/api/compilation/)拓展下知识面。

另外本文用到的流程图可能在表达和像素上都不是很清晰，大家有比较好的画图工具和画图方法，希望推荐一下，感谢！
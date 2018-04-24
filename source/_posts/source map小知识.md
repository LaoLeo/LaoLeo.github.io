---
title: source map小知识
date: 2018-04-09 19:36:00
tags: [webpack, sourceMap]
categories: [工具]
---

![我是图片](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1524580962476&di=2bb2f9b1075674dbb2b22354e010b724&imgtype=0&src=http%3A%2F%2Fwww.d1net.com%2Fuploadfile%2F2014%2F1215%2F20141215110355156.jpeg)


## 前言
前端项目的上线部署前，都需要经过合并压缩后，才能上线。这样会导致的一个问题就是，经过变换的代码，debug没办法定位到代码出错的准确位置。关于代码要压缩变换的原因：
> 1. 合并能减少http请求数
> 2. 压缩可以减少http请求量
> 3. 经过编译后的得到的js，如es6，ts

<!--more-->

## source map文件
source map文件就是用来定位代码的位置的，实质就是个记录代码位置信息的文件

大概长这个样子
```
{
    "version": 3,
    "sources": ["hello.js"],
    "names": ["sayHello", "greeting", "Name", "console", "log"],
    "mappings": "AAAA,QAASA,KAEL,GACIC,GAAW,UAAYC,IAC3BC,SAAQC,IAAIH,GAGhBD",
    "file": "hello.min.js",
    "sourceRoot": "",
    "sourcesContent": ["function sayHello()\n{\n    var name = \"Fundebug\";\n    var greeting = \"Hello, \" + Name;\n    console.log(greeting);\n}\n\nsayHello();\n"]
}
```
> version：Source Map的版本号。  
sources：转换前的文件列表。  
names：转换前的所有变量名和属性名。  
mappings：记录位置信息的字符串，经过编码。 
file：(可选)转换后的文件名。  
sourceRoot：(可选)转换前的文件所在的目录。如果与转换前的文件在同一目录，该项为空。 
sourcesContent:(可选)转换前的文件内容列表，与sources列表依次对应。

source map文件会在压缩代码里面以url或者dataURK的形式与压缩文件产生关联
```
function o(){var o="Hello, "+Name;console.log(o)}o();
//# sourceMappingURL=hello.min.js.map
```

## webpack配置
webpack会把代码编译压缩，所以debug异常也需要source map文件的支持
```
devtool: '#cheap-module-source-map'
```

要注意的是devtool配置source-map，常用的有一下几种模式：


模式 | 作用
---|---
eval | 每个 module 会封装到 eval 里包裹起来执行，并且会在末尾追加注释 //@ sourceURL.
source-map | 生成一个 SourceMap 文件
hidden-source-map | 和 source-map 一样，但不会在 bundle 末尾追加注释
inline-source-map | 生成一个 DataUrl 形式的 SourceMap 文件
eval-source-map | 每个 module 会通过 eval() 来执行，并且生成一个 DataUrl 形式的 SourceMap 
cheap-source-map | 生成一个没有列信息（column-mappings）的 SourceMaps 文件，不包含 loader 的 sourcemap（譬如 babel 的 sourcemap）
cheap-module-source-map | 生成一个没有列信息（column-mappings）的 SourceMaps 文件，同时 loader 的 sourcemap 也被简化为只包含对应行的。

> 其中eval、inline、hidden这些关键字可以随意组合，如cheap-module-eval-source-map

* 在开发模式下  
选择：cheap-module-source-map
> 这里需要主要的是，官方是建议使用cheap-module-eval-source-map,但是这个模式在chrome下没办法定位到源码，不符合开发要求，所以还是选择cheap-module-source-map，实在不行就应用source-map，肯定能定位到源码，对于调试有很大的作用。

* 生产模式下
* 选择：cheap-module-source-map

原因如下：

1. 使用 cheap 模式可以大幅提高 souremap 生成的效率。大部分情况我们调试并不关心列信息，而且就算 sourcemap 没有列，有些浏览器引擎（例如 v8） 也会给出列信息。
2. 使用 eval 方式可大幅提高持续构建效率。参考官方文档提供的速度对比表格可以看到 eval 模式的编译速度很快。
3. 使用 module 可支持 babel 这种预编译工具（在 webpack 里做为 loader 使用）。
4. 使用 inline-source-map 模式可以减少网络请求。这种模式开启 DataUrl 本身包含完整 sourcemap 信息，并不需要像 sourceURL 那样，浏览器需要发送一个完整请求去获取 sourcemap 文件，这会略微提高点效率。而生产环境中则不宜用 eval，这样会让文件变得极大。
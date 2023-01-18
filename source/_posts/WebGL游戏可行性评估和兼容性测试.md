---
title: WebGL游戏可行性评估和兼容性测试
date: 2023-01-11
tags: [WebGL]
categories: [Unity游戏开发,平台,WebGL]
---

<!-- more -->

## WebGL浏览器兼容性

-   支持主流的桌面浏览器，但不同浏览器的不同版本支持程度和性能不一。
-   不支持移动设备，但在高端的移动设备上仍可以运行，瓶颈是内存。

[各浏览器webgl兼容性表](https://docs.unity3d.com/cn/2018.4/Manual/webgl-browsercompatibility.html "各浏览器webgl兼容性表")

## WebGL支持能力

| 能力     | 支持程度 | 解决方案                                                                                                              |
| ------ | ---- | ----------------------------------------------------------------------------------------------------------------- |
| 多线程    | 不支持  | 不支持System.Threading，将多线程代码替换成异步等方式实现                                                                              |
| 网络系统   | 需调整  | 不支持 [System.Net](http://System.Net "System.Net") 接口，HTTP使用UnityWebRequest，TCP使用 WebSocket 通信替代（UnityWebSocket 插件） |
| 音频     | 支持   | 仅基于Web Audio API实现基础功能，不支持压缩和自动播放，不支持Fmod                                                                         |
| 图形API  | 支持   | 基于OpenGL ES图形库功能，WebGL 1.0 匹配 OpenGL ES 2.0 ，WebGL 2.0 匹配 OpenGL ES 3.0                                           |
| 动态生成代码 | 不支持  | WebGL是AOT平台，不支持C#反射等动态代码方案                                                                                        |
| 文件系统   | 需调整  | PlayerPrefs、Unity Local储存文件，存在PresistetDataPath下的文件会储存进IndexedDB，IndexedDB无容量限制                                   |
| Lua脚本  | 支持   | 支持使用ToLua                                                                                                         |
| 三方库    | 需调整  | C#三方库不能包含System.Threading、[System.Net](http://System.Net "System.Net")、System.Reflection等接口                       |

## 浏览器平台兼容性实测

将打出的WebGL包放在各个浏览器运行测试

| 浏览器            | 测试版本                                 | 运行情况                                                                                                                                                                                 |
| -------------- | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Google Chrome  | 108.0.5359.125（正式版本） （64 位）          | 1.各个模块运行基本正常&#xA;&#xA;2.fmod效果音刚播放时噪音有点大&#xA;&#xA;3.fmod背景音乐偶尔进入界面时未播放，像是没有触发，切换界面回来才会播，像是业务逻辑未兼容好的问题&#xA;&#xA;4.输入框捕获不到中文输入                                                         |
| Microsoft Edge | 108.0.1462.54 (正式版本) (64 位)          | 运行情况跟Chrome相似                                                                                                                                                                        |
| Mozila Firefox | 108.0.1 (64 位)                       | 1.fmod效果音刚播放时没有在chrome上噪音大的问题&#xA;&#xA;2.画质上比chrome高清&#xA;&#xA;3.输入框也捕获不到中文输入                                                                                                        |
| Apple Safari   | pjg的mac开发机上测试&#xA;&#xA;版本14.1.1      | 1.手机Safari访问，加载慢，能够进入到登录页，弹不起输入框，运行不久弹窗报错：**WebGL builds are not supported on mobile devices**. &#xA;&#xA;2.fmod的效果音播放的噪声问题也存在，内存紧张是更加明显&#xA;&#xA;3.pv视频播放不了，报user not allow，像是设置禁用了 |
| 360浏览器         | 版本号: 13.1.6380.01内核版本: 86.0.4240.198 | 进入主界面时报： Could not allocate memory: System out of memory! memory access out of bound，分配的内存不够&#xA;&#xA;有些贴图渲染不出来一片黑                                                                   |
| 360浏览器         | 版本号: 13.1.6380.01内核版本: 86.0.4240.198 | 进入主界面时报： Could not allocate memory: System out of memory! memory access out of bound，分配的内存不够&#xA;&#xA;有些贴图渲染不出来一片黑                                                                   |

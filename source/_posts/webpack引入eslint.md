---
title: webpack引入eslint
date: 2018-04-08 21:50:50
tags: [webpack, eslint]
categories: [工具]
---

![我是图片](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1525174897&di=c0a06449ab438e83c3a8f2a245203535&imgtype=jpg&er=1&src=http%3A%2F%2Fwww.ibeifeng.com%2Fimages%2Fupload%2FImage%2Fc5bbf603918fa0ec91b3bd73259759ee3c6ddbc0.jpg)

# 前言
eslint是前端编码规范的检测工具，本文记录如何在webpack配置eslint和定义前端的编码规范

<!--more-->

在webpack搭建的项目中引入eslint语法检测，分以下两步  

**首先node_modules下有以下包**  
eslint   
eslint-friendly-formatter : 格式化检测输出内容  
eslint-loader : webpack的支持eslint检测的加载器
eslint-config-standard ：标准语法规则  
// 以下都是为eslint提供环境或者语法支持的包  
eslint-plugin-html  
eslint-plugin-import  
eslint-plugin-node  
eslint-plugin-promise  
eslint-plugin-standard


1. 根目录下新建.eslintrc.js

```
npm i -g eslint
eslint --init
选择standard标准
```
.eslintrc.js配置
```
    // 设置为true，eslint就不会再往父级目录找配置文件了
    root: true,
    // 分析器，eslint有默认的分析，babel-eslint算是一个扩展，有些语法只有babel支持而eslint不支持，babel-eslint弥补了这一缺点
    parser: 'babel-eslint',
    // 预定义的一些全局变量，应该在src下可以给eslint用来判断，类似process.env.NODE_ENV，但暂时没用过
    env: {
        browser: true,
        node: true,
        es6: true
    },
    // 标准规则模式
    extends: "standard",
    // eslint-plugin-html插件，在.html和.vue下提供eslint检测
    plugins: [
        'html'
    ],
    /*
        'off'/0 关闭
        'warn'/1 警告级别
        'error'/2 错误级别
    */
    rules: {
        'indent': ['error', 4, {
            'SwitchCase': 1,
            'VariableDeclarator': { 'var': 1, 'let': 1, 'const': 1 }
        }],
        'no-trailing-spaces': 'error',
        'space-before-function-paren': ['error', 'never'],
        'key-spacing': ['error', { 
            'beforeColon': false,
            'afterColon': true
        }],
        'object-curly-newline': ['error', { 
            'ObjectExpression': 'always',
            'ObjectPattern': 'never'
        }],
        'quotes': ['error', 'single'],
        'curly': 'error',
        'object-property-newline': 'error',
        'no-console': process.env.NODE_ENV === 'production' ? 'error' : 'off',
        'no-cond-assign': ['error', 'always'],
        'no-constant-condition': ['error', { 'checkLoops': false }],
        'no-empty':['error'],
        'no-var': ['error'],
        'line-comment-position': ['error', { 'position': 'above' }]
    }
```
可以查看一些常用的配置：[https://www.jianshu.com/p/a4966ddf9b0c](https://www.jianshu.com/p/a4966ddf9b0c)

2. 在webpack中配置
```
module: {
    rules: [
        {
            test: /\.(js|vue)$/,
            loader: 'eslint-loader',
            // 强制在编译前检测
            enforce: 'pre',
            // 不包括的文件，所以可以不用.eslintignore文件
            exclude: [path.resolve(__dirname, '../node_modules')],
            options: {
                formatter: require('eslint-friendly-formatter'),
                // 有错自动修复，注意webpack解决不了会陷入死循环
                fix: true,
                // 有语法错误编译照样通过
                emitWarning: true
            }
        }
    ]
}
```
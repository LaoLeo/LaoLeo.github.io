---
title: MVVM框架双向数据绑定的实现原理
date: 2018-03-06
tags: [MVVM]
categories: [前端框架]
---

![我是图片](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1525175920&di=0a8226ac6efda720e6da74d056b697a3&imgtype=jpg&er=1&src=http%3A%2F%2Fwww.chinairn.com%2FUserFiles%2Fimage%2F20161106%2F20161106162224_9200.jpg)

# 前言
> 目前前端最火的几个框架vue，angular等都是基于MVVM框架思想，那么什么是MVVM框架，和它的双向数据绑定都是如何实现的呢？

<!--more-->

# 什么是MVVM？

MVVM是一种发开代码的组织和设计思想，说白了就是框架，它跟MVC是同一个物种，也可以说是从MVC演化到MVP，再在MVP的基础进化为MVVM。

在了解MVVM之前，我们有必要了解它的祖先。MVC是指Model(数据模型)，View(视图)，Controller(控制器)，这些我们都很好理解，而MVP的P指的是Presenter。它跟Controlller有点相似，不同的是，它是用户触发View上的绑定的Dom事件后，View将修改通知给Presenter来完成后续的操作(更新数据或者视图)。而在MVC模式下，用户是直接操作Controller，以更改url上的hash发送请求的方式。所以很明显，Presenter跟View是双向绑定的。

回到MVVM，VM指的是ViewModel。ViewModel是把Present拆分为多个小的指令步骤(directive)，它将View和Model双向绑定，用户操作修改View，ViewModel驱动Model进行更新，相同的，Model数据被更改，ViewModel检测到并驱动View更新。
那么Model数据更改是如何被ViewModel检测到的呢？

# 数据变更检测

在MVVM模式下，在view→model的方向，用View层通过触发一些元素的事件，例如input的onchange事件，将修改通知给ViewModel，然后ViewModel再操作Model，这很容易理解。

那么Model→view这个方向上的通知如何实现呢？那就需要数据变更检测机制，它能够检测到Model的变化，并通过ViewModel更新View层。它有很多实现形式，特别是ES6 新特性的出现，丰富了实现数据对象的变更检测的方式。

下面介绍四种方式： 
* 手动触发绑定
* 脏检测机制
* 数据对象劫持
* Proxy

# 数据变更检测四种方式

## 手动触发绑定

这种方式比较直接，思路是通过在数据对象上定义get()和set()方法，改变数据后手动触发这两个方法来获取和设置。Angular正是通过这种方式进行view层操作更新的。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>数据变更检测之手动触发绑定</title>
</head>
<body>
    <input type="text" q-value="value" id="input">
    <div>
        <span q-text="value" id="el"></span>
    </div>

    <script>
        /**
        原理：通过事件绑定，手动触发viewmodel更改数据和渲染视图
        */
        //相当于view，通过节点数组获取节点
        let elems = [document.getElementById("el"), document.getElementById("input")];

        //数据对像
        let data = {
            value: 'hello'
        };

        //定义directive：操作逻辑定义
        let directive = {
            text: function (text) {
                this.innerHTML = text;
            },
            value: function(value) {
                // this.setAttribute('value', value); setAttribute一般用于设置自定义属性
                this.value = value;
            }
        };

        //数据绑定监听 view ==> model
        if(document.addEventListener) {
            elems[1].addEventListener('keyup', function(e) {
                ViewModelSet('value', e.target.value);
            }, false);
        }else {
            elems[1].attachEvent('onkeyup', function(e) {
                ViewModelSet('value', e.target.value);
            }, false);
        }

        //开始扫描
        scan();
        //model ==> view
        setTimeout(function() {
            ViewModelSet('value', 'hello AlexL');
        }, 1000)

        function scan() {
            //扫描带指令的节点属性
            for(let elem of elems) {
                elem.directive = [];
                for(let attr of elem.attributes) {
                    if(attr.nodeName.indexOf('q-') > -1) {
                        //调用属性指令
                        directive[attr.nodeName.slice(2)].call(elem, data[attr.nodeValue]);
                        elem.directive.push(attr.nodeName.slice(2));
                    }
                }
            }
        }

        function ViewModelSet(key, value) {
            data[key] =value;
            scan();
        }


    </script>
</body>
</html>
```

## 脏检测机制

脏检测机制的基本原理是，在viewmodel对象的某个属性值发生变化时找到与这个属性值相关的所有元素，然后进行数据比较，如果变化就调用指令，重新扫描并渲染这个元素。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>数据变更检测之脏数据检测</title>
</head>
<body>
    <input type="text" id="input" qg-event="value" qg-bind="value">
    <div>
        <span id="el" qg-event="text" qg-bind="value"></span>
    </div>
<script>
    let elems = [document.getElementById('el'), document.getElementById('input')];
    let data = {
        value: 'hello'
    };

    let directive = {
        text: function(str) {
            this.innerHTML = str;
        },
        value: function(str) {
            this.setAttribute('value', str);
        }
    };

    //初始化扫描节点
    scan(elems);
    // $digest('value');

    //数据绑定监听
    if(document.addEventListener) {
        elems[1].addEventListener('keyup',function(e) {
            data.value = e.target.value;
            $digest(e.target.getAttribute('qg-bind'));
        }, false);
    }else {
        elems[1].attachEvent('onkeyup', function(e) {
            data.value = e.target.value;
            $digest(e.target.getAttribute('qg-bind'));
        }, false);
    }

    setTimeout(function() {
        data.value = 'nice job!';
        $digest('value');
    }, 5000)

    function scan() {
        for(let elem of elems) {
            elem.directive = [];
        }
    }

    //数据劫持监听，只扫描绑定了这个数据（value）的元素
    function $digest(value) {
        let list = document.querySelectorAll('[qg-bind=' + value + ']');
        // console.log(list)
        digest(list); 
    }

    //脏数据循环检测
    function digest(elems) {
        //扫描带指令的节点属性
        for(let i=0, len=elems.length; i<len; i++) {
            let elem = elems[i];
            for(let j=0, len=elem.attributes.length; j<len; j++) {
                let attr = elem.attributes[j];
                if(attr.nodeName.indexOf('qg-event') > -1) {
                    //调用属性指令
                    let datakey = elem.getAttribute('qg-bind') || undefined;
                    //判断数据是否发生变化，是则执行指令重新渲染，否则跳过
                    if(elem.directive[attr.nodeValue] !== data[datakey]) {
                        directive[attr.nodeValue].call(elem, data[datakey]);
                        elem.directive[attr.nodeValue] = data[datakey];
                    }
                }
            }
        }
    }

</script>
</body>
</html>
```

## 数据对象劫持

基本思路是使用Object.defineProperty和Object.defineProperies对viewmodel数据对象进行属性get()和set()的监听，当有数据读取和赋值操作的时候则扫面节点，运行指定对应节点的Directive指令，viewmodel通过等号赋值就可以了。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>数据变更检测之对象劫持</title>
</head>
<body>
    <input type="text" q-value="value" id="input">
    <div>
        <span q-text="value" id="el"></span>
    </div>

<script>
    let elems = [document.getElementById('el'), document.getElementById('input')];
    let data = {
        value: 'hello'
    };

    let directive = {
        text: function(str) {
            this.innerHTML = str;
        },
        value: function(str) {
            this.setAttribute('value', str);
        }
    };

    let bValue;
    scan();

    //数据劫持监听
    defineGetAndSet(data, 'value');

    //数据绑定监听
    if(document.addEventListener) {
        elems[1].addEventListener('keyup',function(e) {
            data.value = e.target.value;
        }, false);
    }else {
        elems[1].attachEvent('onkeyup', function(e) {
            data.value = e.target.value;
        }, false);
    }

    setTimeout(function() {
        data.value = 'hello 木木夕';
    }, 2000);

    function scan() {
        //扫描带指令的节点属性
        for(let elem of elems) {
            elem.directive = [];
            for(let attr of elem.attributes) {
                if(attr.nodeName.indexOf('q-') > -1) {
                    //调用属性指令
                    directive[attr.nodeName.slice(2)].call(elem, data[attr.nodeValue]);
                    elem.directive.push(attr.nodeName.slice(2));
                }
            }
        }
    }

    //定义对象属性并设置劫持
    function defineGetAndSet(obj, propName) {
        Object.defineProperty(obj, propName, {
            get: function() {
                return bValue;
            },
            set: function(newValue) {
                bValue = newValue;
                scan();
            },
            enumerable: true,
            configurable: true
        })
    }



</script>
</body>
</html>
```

## ES6 Proxy

ES6的新特性Proxy，可以在已有对象的基础上重新定义一个对象，并重新定义对象原型上的方法，包括get()和set()。

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>数据变更之proxy对象劫持</title>
</head>

<body>
    <label>请输入<input type="text" q-value="value" id="input"></label>
    <div>
        <span q-text="value" id="el"></span>
    </div>

    <script>
        /**
         * 跟Object.defineProperty()差不多，都是自定义数据对象的get和set方法，达到对象劫持的目的； 
         * 缺点在于，scan方法，假如组件中节点过多，扫描过程就会变长，从而拖慢了代码的执行速度，尤其要避免重复元素的扫描
         */
        let elems = [document.getElementById('el'), document.getElementById('input')];
        let directive = {
            text: function (str) {
                this.innerHTML = str;
            },
            value: function (str) {
                this.setAttribute('value', str);
            }
        };

        //设置data的访问proxy
        let data = new Proxy({
            value: 'my name is AlexL'
        }, {
                get: function (target, key, receiver) {
                    return target.value; //data对象中的value属性
                },
                set: function (target, key, value, receiver) {
                    target.value = value;
                    scan();
                    return target.value;
                }
            });

        // 初始化
        // data['value'] = 'my name is laotuzhu';
        scan();

        //数据绑定监听
        if (document.addEventListener) {
            elems[1].addEventListener('keyup', function (e) {
                data.value = e.target.value;
            }, false);
        } else {
            elems[1].attachEvent('onkeyup', function (e) {
                data.value = e.target.value;
            }, false);
        }

        setTimeout(function () {
            data.value = 'hello 木木夕';
        }, 2000);

        function scan() {
            //扫描带指令的节点属性
            for (let elem of elems) {
                elem.directive = [];
                for (let attr of elem.attributes) {
                    if (attr.nodeName.indexOf('q-') > -1) {
                        //调用属性指令
                        directive[attr.nodeName.slice(2)].call(elem, data[attr.nodeValue]);
                        elem.directive.push(attr.nodeName.slice(2));
                    }
                }
            }
        }
    </script>
</body>

</html>
```

# 结语
> 实现一个MVVC框架远远没这么简单，里面涉及很多性能优化等机制，在上诉的四种方法中，在数据变更时，都把整个Dom结构扫描了，在渲染Dom时连没变化的也重新渲染，这样是很耗性能的，在Angular，vue这样的框架中，用到了virtual dom来最小化的操作dom，这样就可以解决这个问题。    
> 但是，也无可避免的操作了Dom，我们知道浏览器的Dom操作其实是很慢的，有没有在完全不操作dom的情况下，渲染视图呢？其实是有的，MNV\*框架就可以办到，它适用于移动端的hybird app，通过定义的协议来调用原生方法来渲染视图，来达到跟原生体验非常接近的效果 。关于MNV\*框架的讨论不在本章的范畴，有兴趣的同学可以去自行了解。
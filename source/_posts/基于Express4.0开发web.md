---
title: 基于Express4.0开发web 
date: 
tags: [nodejs,express,grunt,router,MVC,jade,mongoDB]
---

### 1.快速搭建
在全局下安装express  
```
npm install -g express  
```
执行  
```
express -v yourProject 	
或者
cd yourProject
express
```
就会快速在目录yourProject中搭建express，-v参数指定模板引擎，没写就默认为jade


### 2.express中间件
```
var app = requrie('express')();
app.use (function (req, res, next) {
    ...
    next()
})  
```
next()作用是交付控制权，前往下一个中间件
可以利用这一点做验证功能

### 3.MVC架构
```
/app
    /controllers
    /models
    /schemas
    /views
/public
    /libs
    /css
    /js
/router
    router.js
app.js
package.json
.bowerrc
bower.json
gruntfile.js
.gitignore

```
以上就是项目的架构，按照MVC架构，后台代码都放在app下，schemas存放mongodb的模式，可以看作是一张张表，models放生成模型的文件，模型与模式对接起来。public存放静态文件，/libs存放前端框架如bootstrap。把router路由模块抽离放在router目录。  
bower模块用来安装项目依赖的前端框架。需要在全局安装bower
```
npm install -g bower
```
安装bootstrap
```
bower init
bower install bootstrap
```
bower init 生成bower.json
然后安装bootstrap时，会根据.bowerrc文件
```
{
    "directory": "public/libs"
}
```
把依赖的框架安装在public/libs下

### 4.项目管理工具grunt
gruntfile.js
```js
module.exports = function(grunt) {

    grunt.initConfig({
        watch: {
            jade: {
                files: ['views/**'],
                options: {
                    livereload: true
                }
            },
            js: {
                files: ['public/js/**', 'app/**/*.js'],
                //tasks: ['jshint'],
                options: {
                    livereload: true
                }
            },
            uglify: {
                files: ['public/**/*.js'],
                tasks: ['jshint'],
                options: {
                    livereload: true
                }
            },
            styles: {
                files: ['public/**/*.less'],
                tasks: ['less'],
                options: {
                    nospawn: true
                }
            }
        },

        nodemon: {
            dev: {
                script: 'bin/www',  //入口文件
                options: {
                    nodeArgs: ['--debug'], //开启开发模式
                    args:　[],
                    ignoredFiles: ['README.md', 'node_modules/**', 'public/libs/**', '.vscode'],
                    watchedExtensions: ['js'],
                    watchedFolders: ['./'], //监听定义的文件夹,根目录
                    debug: true,
                    delayTime: 1000,
                    env: {
                        PORT: 3000
                    },
                    cwd: __dirname
                }
            }
        },

        jshint: {
            options: {
                jshintrc: '.jshintrc',
                ignores: ['public/libs/**/*.js']
            },
            all: ['public/javascripts/*.js', 'test/**/*.js', 'app/**/*.js']
        },

        less: {
            development: {
                options: {
                    compress: true,
                    yuicompress: true,
                    optimization: 2
                },
                files: {
                    'public/build/index.css' : 'public/less/index.less'
                }
            }
        },

        uglify: {
            development: {
                files: {
                'public/build/admin.min.js': 'public/javascripts/admin.js',
                'public/build/detail.min.js': [
                    'public/javascripts/detail.js'
                ]
                }
            }
        },

        mochaTest: {
            options: {
                reporter: 'spec'
            },
            src: ['test/**/*.js']
        },

        concurrent: {
            tasks: ['nodemon', 'watch', 'jshint', 'less', 'uglify'],
            options: {
                logConcurrentOutput: true
            }
        }
    });

    grunt.loadNpmTasks('grunt-contrib-watch'); //监控定义好的静态文件变动
    grunt.loadNpmTasks('grunt-nodemon'); //监控入口文件变动
    grunt.loadNpmTasks('grunt-concurrent'); //优化慢任务的构建时间，比如sass,less，并发执行多个阻塞的任务,比如nodemon和watch
    grunt.loadNpmTasks('grunt-mocha-test');  //测试框架
    grunt.loadNpmTasks('grunt-contrib-less');
    grunt.loadNpmTasks('grunt-contrib-uglify');
    grunt.loadNpmTasks('grunt-contrib-jshint');

    grunt.option('force', true); //避免程序因语法错误而终止执行

    grunt.registerTask('default', ['concurrent']);
    grunt.registerTask('test', ['mochaTest']);
}
```
前期开发只需要前面三个模块包即watch,nodemon,concurrent就行。

###  5.router模块
这里有两种方法  
第一种是将路由分类  
```
app.js
//挂载路由表
app.use('/', index);
app.use('/admin', admin);
app.use('/user', user)

user.js
var express = require('express');
var User = require('../models/user');
var router = express.Router();

//用户注册
router.post('/signup', function(req, res) {
    ...
}
module.exports = router;
```
当url以/user开头时就会进入user.js文件。  
第二种是路由全部写在一个文件  
```
app.js
require('./config/routers')(app); //不能传入express.Router(),它只能作为中间件用,如上

router.js
var Movie = require('../app/controllers/movie');
var Index = require('../app/controllers/index')


module.exports = function(app) {

    //Index
    app.get('/', Index.index);

    //Movie
    app.get('/movie/:id', Movie.detail);
    app.get('/admin/movie/list',User.signinRequired,User.adminRequired, Movie.list);
    
};

```

### 6.数据库模块mongose
```js
app.js
var mongoose = require('mongoose');
var dbURL = 'mongodb://localhost:27017/movieWeb';

//连接数据库
mongoose.Promise = global.Promise; //避免控制台输出Promise错误 
mongoose.connect(dbURL) //数据库名为movieWeb
mongoose.connection.on('connected', function () {    //绑定connected函数，连接成功时触发
  console.log('Connection success!');
});
```
模式schemas(表)
```js
var mongoose = require('mongoose');
var Schema = mongoose.Schema;
var ObjectId = Schema.Types.ObjectId;

var MovieSchema = new mongoose.Schema({  //定义模式
    doctor: String,
    title: String,
    language: String,
    country: String,
    summary: String,
    flash: String,
    poster: String,
    year: String,
    pv: {
        type: Number,
        default: 0
    },
    category: {
        type: ObjectId,
        ref: 'Category'
    },
    meta: {
        createAt: {
            type: Date,
            default: Date.now()
        },
        updateAt: {
            type: Date,
            default: Date.now()
        }
    }
})

//每次存取数据之前，都调用save方法
MovieSchema.pre('save', function(next){
    if(this.isNew) {
        this.meta.createAt = this.meta.updateAt = Date.now()
    } else {
        this.meta.updateAt = Date.now()  
    }

    next() //将流程走下去
})

MovieSchema.statics = {  //实例化以后就会具有以下方法，相当于构造函数的成员方法
    fetch: function(cb) {
        return this
            .find({})
            .sort('meta.updateAt')
            .exec(cb)
    },
    fetchById: function(id, cb) {
        return this
            .findOne({_id: id})
            .exec(cb)
    }
}

module.exports = MovieSchema
```
模型models（模式跟模型对接）
```js
var mongoose = require('mongoose')
var MovieSchema = require('../schemas/movie')
var Movie = mongoose.model('Movie', MovieSchema)  //模式和模型对接

module.exports = Movie

```
值得注意的是，使用mongodb之前，需要在电脑上安装mongodb，并注册成一个服务，方便使用前开启，具体参考笔记：mongodb错误

### 7.将session存放在mongodb里
session是会话支持，作为客户端的id，跟服务器交互
```
//使用mongodb来储存sesion信息
var cookieParser = require('cookie-parser');
var session = require("express-session");
var MongoStore = require('connect-mongo')(session);

//配置session
app.use(cookieParser());
app.use(session({
  secret: 'imoocMovieWeb',
  store: new MongoStore({
    url: dbURL,
    collection: 'sessions'
  }),
  resave: true,
  saveUninitialized: true
}))
```
其中呢，cookieParser()中间件是session和bodyParser的依赖的中间件
bodyParser是解析request请求的中间价
```
配置
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({extended:true})); //extended为true则可以解析字符串和数组以外的数据
```

### 8.前后台传数据
可以用res.render方法
```js
controllers/movie.js

//获取后列表页
exports.list = function(req, res, next) {
  Movie.fetch(function(err, movies) {
    if(err) {
      console.log(err)
    }

    res.render('list', {
      title: '后台列表页',
      movies: movies
    });
  })
};
```
还可以用res.locals，可以传递方法和变量
```js
app.js

//前后台传递数据
app.use(function(req, res, next) {
  res.locals.moment = moment;  //向前台模板发送处理时间的moment方法
  
  var _user = req.session.user;
  res.locals.user = _user; //传递用户转态

  next();
})
```

### 8.各种用到的中间件
* flash  
```
var flash = require('connect-flash') //存储变量的中间件，相当全局变量
app.use(flash())

使用
var error = req.flash('error')
```

*  logger  
```js
var logger = require('morgan'); //处理日志模块 

//日志文件
var fs = require('fs');
var accessLogfile = fs.createWriteStream('access.log', {flags:'a'});
var errorLogfile = fs.createWriteStream('error.log', {flags:'a'});

app.set('env', 'production');
app.use(logger({stream: accessLogfile})); //访问日志

//处理错误日志，处理错误的中间件一定要写在路由后面
if ('production' == app.get('env')) {
  app.use(function(err, req, res, next){
    var meta = '[' + new Date() + '] ' + req.url + '\n';
    errorLogfile.write(meta + err.stack + '\n');
    next();
  });
}
```

* moment  
```
app.js
var moment = require('moment'); //处理时间格式
在中间件里
res.locals.moment = moment;  //向前台模板发送处理时间的moment方法

jade模板文件
moment(item.meta.updateAt).format('MM/DD/YYYY') //item.meta.updateAt是后台传过来的时间数据new Date()
```

* bcrypt-nodejs 加盐模块
```js
var bcrypt = require('bcrypt-nodejs');
var SALL_WORK_FACTOR = 10; //数值越大，破解越复杂
bcrypt.genSalt(SALL_WORK_FACTOR, function(err, salt) {
        if(err) return next(err);

        bcrypt.hash(user.password, salt, null, function(err, hash) {
            if(err) return next(err);

            user.password = hash
            next() //将流程走下去
        })
    })
```

* underscore （用到这模块的继承方法）  
```
var _ = require('underscore');
_movie = _.extend(movie, movieObj)
```

### 9.jade模板引擎
```jade
layout.jade 公用的部份

doctype html
html
  head
    meta(charset="utf-8")
    title #{title} - imoocMovie
    include ./includes/head
  body
    include ./includes/header
    block content
    include ./includes/footer
```
引进用include  
block content是主体内容
```jade
extends ../layout  //-扩展layout

block content
  .container
    .row
      each cat in categories
        .panel.panel-default
          .panel-heading
            h3
              a(href='/results?cat=#{cat._id}&p=0') #{cat.name}
          .panel-body
            if cat.movies && cat.movies.length > 0
              each item in cat.movies
                .col-md-2
                  .thumbnail
                    a(href="/movie/#{item._id}")
                      if item.poster.indexOf('http:') > -1
                        img(src="#{item.poster}", alt="#{item.title}")
                      else
                        img(src="/upload/#{item.poster}", alt="#{item.title}")
                    .caption
                      h4 #{item.title}
                      p: a.btn.btn-primary(href="/movie/#{item._id}", role="button") 观看预告片
    script(src='/build/admin.min.js')
```

* request（发http请求）
```
var request = require('request');

//用法
request({url: url, json: true}, function (error, response, body) {
            if (!error && response.statusCode === 200) {
                var data = body;
                var now = (new Date().getTime());
                var expires_in = now + (data.expires_in - 20) * 1000;
                data.expires_in = expires_in;
                resolve(data);
                //console.log(data);
            } else {
                reject('can not get ticket')
            }
        });
```







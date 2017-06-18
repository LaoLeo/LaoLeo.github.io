---
title: node开发微信公众号 
date: 2017-06-13
tags: [nodejs,微信公众号]
---


## 开发第一步

不是埋头就是实现各种逻辑，而是，划分好逻辑模块，思考各个模块的位置（目录位置），因为总不能将所有代码都放在app.js里，进而构成了整个项目的架构。

开发微信公众号，理解整个交互的流程。客户端 《 == 》微信服务器 《 == 》项目服务器  

开发前的准备工作请详细看微信开发者文档，这里只讨论代码实现。  

## 开始开发
我们使用koa框架。  
app.js
```js
'use strict'

var Koa = require('koa')
var wechat = require('./wechat/g')  //可以看作是程序的第二个流程，实现验证 以及调用回复功能
var config = require('./config') //公众号的配置配置信息
var reply = require('./wx/reply')  //处理客户发过来的消息，回复逻辑
var Wechat = require('./wechat/wechat') //回复各种消息类型获取的api，封装在Wechat对象中

var app = new Koa()

app.use(wechat(config.wechat, reply.reply))

app.listen(1234)
console.log("listening at port 1234...")
```

wechat/g.js
```js
'use strict'

var sha1 = require('sha1')
var getRawBody = require('raw-body')
var Wechat = require('./wechat')
var util = require('./util')

module.exports = function(opts, handler) {
    var wechat = new Wechat(opts)

    return function *(next){

                var token = opts.token
                var signature = this.query.signature
                var nonce = this.query.nonce
                var timestamp = this.query.timestamp
                var echostr = this.query.echostr
                var str = [token, timestamp, nonce].sort().join('')
                var sha = sha1(str)

                if(this.method == 'GET'){
                    if(sha === signature){
                        this.body = echostr + ''
                    }
                    else{
                        this.body = 'wrong'
                    }
                }
                else if(this.method == 'POST'){
                    console.log(this.method)
                    if(sha !== signature){
                        this.body = 'wrong'

                        return false
                    }

                    var data = yield getRawBody(this.req, { //xml格式的数据
                        length: this.length,
                        limit: '1mb',
                        encoding: this.charset
                    })

                    //console.log(data.toString())
                    var content = yield util.parseXMLAsync(data) //转化为json格式的数据
                    //console.log(content)
                    var message = util.formatMessage(content.xml)
                    //console.log(message)

                   this.weixin = message

                   yield handler.call(this, next)

                   wechat.reply.call(this)   //数据转化为最终的xml格式，客户端只能解析xml格式数据
                }
                
            }
}
```

wechat/wechat.js
```js
'use strict'

var Promise = require('bluebird')
var request = require('request')
var util = require('./util')
var fs = require('fs')
var _ = require('lodash')

var prefix = 'https://api.weixin.qq.com/cgi-bin/'
var api = {
    accessToken : prefix + 'token?grant_type=client_credential',
    temporary: {
        upload : prefix + 'media/upload?',
        fetch: prefix + 'media/get?'
    },
    permanent: {
        upload: prefix + 'material/add_material?',
        fetch: prefix + 'material/get_material?',
        uploadNews: prefix + 'material/add_news?',
        uploadNewsPic:prefix + 'material/uploadimg?',
        del: prefix + 'material/del_material?',
        update: prefix + 'material/update_news?',
        count: prefix + 'material/get_materialcount?',
        batch: prefix + 'material/batchget_material?'
    },
    group: {
        create: prefix + 'groups/create?',
        fetch: prefix + 'groups/get?',
        check: prefix + 'groups/getid?',
        update:prefix + 'groups/update?',
        move: prefix + 'groups/members/update?',
        batchupdata: prefix + 'groups/members/batchupdate?',
        del: prefix + 'groups/delete?'
    },
    user: {
        remark: prefix +　'user/info/updateremark?',
        fetch: prefix +　'user/info?',
        batchFetch: prefix + 'user/info/batchget?',
        list: prefix +　'user/get?'
    },
    mass: {
        group: prefix + 'message/mass/sendall?',
        openId: prefix + 'message/mass/send?',
        del: prefix + 'message/mass/delete?',
        preview: prefix + 'message/mass/preview?',
        check: prefix + 'message/mass/get?'
    },
    menu: {
        create: prefix + 'menu/create?',
        get:　prefix +　'menu/get?',
        del: prefix + 'menu/delete?',
        current: prefix + 'get_current_selfmenu_info?'
    },   
    semantic: 'https://api.weixin.qq.com/semantic/semproxy/search?',
    ticket: {
        get : prefix +　'ticket/getticket?'
    }
    
}

function Wechat(opts){
    var that = this

    this.appID = opts.appID
    this.appSecret = opts.appSecret
    this.getAccessToken = opts.getAccessToken
    this.saveAccessToken = opts.saveAccessToken
    this.getTicket = opts.getTicket
    this.saveTicket = opts.saveTicket

    this.fetchAccessToken()
}

//获取ticket
Wechat.prototype.fetchTicket = function(access_token){
    var that = this

    return this.getTicket()
        .then(function(data){
            try{
                data = JSON.parse(data)
            }
            catch(e){
                return that.updateTicket(access_token)
            }
            
            if(that.isValidTicket(data)){
                return Promise.resolve(data)
            }
            else{
                return that.updateTicket(access_token)
            }
        })
        .then(function(data){
            that.saveTicket(data)

            return Promise.resolve(data)
        })
}

Wechat.prototype.updateTicket = function(access_token){

   var url = api.ticket.get + '&access_token=' + access_token + '&type=jsapi'

   return new Promise(function(resolve, reject){
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
   })
   
}

Wechat.prototype.isValidTicket = function(data){
    if(!data || !data.ticket || !data.expires_in){
        return false
    }

    var ticket = data.ticket
    var expires_in = data.expires_in
    var now = (new Date().getTime())

    if(ticket && now < expires_in){
        return true
    }
    else{
        return false
    }
}

//获取access_token
Wechat.prototype.fetchAccessToken = function(data){
    var that = this

    if(this.access_token && this.expires_in){
        if(this.isValidAccessToken(this)){
            return Promise.resolve(this)
        }
    }

    return this.getAccessToken()
        .then(function(data){
            try{
                data = JSON.parse(data)
            }
            catch(e){
                return that.updateAccessToken()
            }
            
            if(that.isValidAccessToken(data)){
                return Promise.resolve(data)
            }
            else{
                return that.updateAccessToken()
            }
        })
        .then(function(data){
            that.access_token = data.access_token
            that.expires_in = data.expires_in

            that.saveAccessToken(data)

            return Promise.resolve(data)
        })
}

Wechat.prototype.isValidAccessToken = function(data){
    if(!data || !data.access_token || !data.expires_in){
        return false
    }

    var access_token = data.access_token
    var expires_in = data.expires_in
    var now = (new Date().getTime())

    if(now < expires_in){
        return true
    }
    else{
        return false
    }
}

Wechat.prototype.updateAccessToken = function(){
   var appID = this.appID
   var appSecret = this.appSecret
   var url = api.accessToken + '&appid=' + appID + '&secret=' + appSecret

   return new Promise(function(resolve, reject){
       request({url: url, json: true}, function (error, response, body) {
            if (!error && response.statusCode === 200) {
                var data = body;
                var now = (new Date().getTime());
                var expires_in = now + (data.expires_in - 20) * 1000;
                data.expires_in = expires_in;
                resolve(data);
                //console.log(data);
            } else {
                reject('can not get access_token')
            }
        });     
   })
   
}

Wechat.prototype.uploadMaterial = function(type, material, permanent){
    var that = this
    var form = {}
    var uploadUrl = api.temporary.upload

    if(permanent){
        uploadUrl = api.permanent.upload

        _.extend(form, permanent)
    }

    if(type === 'pic'){
        uploadUrl = api.permanent.uploadNewsPic
    }

    if(type === 'news'){
        uploadUrl = api.permanent.uploadNews
        form = material
    }
    else{
        form.media = fs.createReadStream(material)
    }
   

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = uploadUrl + 'access_token=' + data.access_token 

                if(!permanent) {
                    url += '&type=' + type
                }
                else{
                    form.access_token = data.access_token
                }

                var options = {
                    method: 'POST',
                    url: url,
                    json: true
                }

                if(type === 'news'){
                    options.body = form
                }
                else{
                    options.formData = form
                }

                request(options, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                        //console.log(_data);
                    } else {
                        throw new Error('upload material fails')
                    }
                }) 

           })

          
   })
   
}

Wechat.prototype.fetchMaterial = function(mediaId, type, permanent){
    var that = this
    var fetchUrl = api.temporary.fetch

    if(permanent){
        fetchUrl = api.permanent.fetch
    }
   
   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = fetchUrl + 'access_token=' + data.access_token 
                var form = {}
                var options = {method:'POST', url: url, json: true}

                if(permanent){
                    form.media_id = mediaId
                    form.access_token = data.access_token
                    options.body = form
                }
                else{
                    if(type === 'video'){
                        url = url.replace('https://', 'http://')
                    }
                    url += '&media_id=' + mediaId
                }

               if(type === 'news' || type === 'video'){
                   request(options, function (error, response, body) {
                        if (!error && response.statusCode === 200) {
                            var _data = body;
                            
                            if(_data){
                                resolve(_data);
                            }                     
                        
                        } else {
                            throw new Error('delete material fails')
                        }
                    })
               }
               else{
                   resolve(url)
               }

                

           })

          
   })
   
}

Wechat.prototype.deleteMaterial = function(mediaId){
    var that = this
    var form = {
        media_id:mediaId
    }
   
   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.permanent.del + 'access_token=' + data.access_token + '&media_id=' + mediaId

                request({method:'POST', url: url, body:form, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                       
                    } else {
                        throw new Error('delete material fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.updateMaterial = function(mediaId, news){
    var that = this
    var form = {
        media_id:mediaId
    }

    _.extend(form, news)
   
   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.permanent.update + 'access_token=' + data.access_token + '&media_id=' + mediaId

                request({method:'POST', url: url, body:form, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                       
                    } else {
                        throw new Error('delete material fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.countMaterial = function(){
    var that = this

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.permanent.count + 'access_token=' + data.access_token

                request({method:'GET', url: url, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                       
                    } else {
                        throw new Error('count material fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.batchMaterial = function(options){
    var that = this

    options.type = options.type || 'image'
    options.offset = options.offset || 0
    options.count = options.count || 1

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.permanent.batch + 'access_token=' + data.access_token

                request({method:'POST', url: url, body:options, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                       
                    } else {
                        throw new Error('batch material fails')
                    }
                }) 
           })         
   })  
}

//用户分组
Wechat.prototype.createGroup = function(name){
    var that = this

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.group.create + 'access_token=' + data.access_token
                var form = {
                    group: {
                        name: name
                    }
                }

                request({method:'POST', url: url, body:form, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                       
                    } else {
                        throw new Error('create group fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.fetchGroup = function(){
    var that = this

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.group.fetch + 'access_token=' + data.access_token

                request({url: url, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                       
                    } else {
                        throw new Error('fetch group fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.checkGroup = function(openId){
    var that = this

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.group.check + 'access_token=' + data.access_token
                var form = {
                    openid : openId
                }

                request({method:'POST', url: url, body: form, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                       
                    } else {
                        throw new Error('check group fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.updateGroup = function(id, name){
    var that = this

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.group.update + 'access_token=' + data.access_token
                var form = {
                    group: {
                        id: id,
                        name: name
                    }
                }

                request({method:'POST', url: url, body: form, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                       
                    } else {
                        throw new Error('update group fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.moveGroup = function(openIds, to_groupid){
    var that = this

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
               var url
               var form = {
                   to_groupid : to_groupid
               }

               if(_.isArray(openIds)){
                   url = api.group.batchupdata + 'access_token=' + data.access_token
                   form.openid_list = openIds
               }
               else{
                   url = api.group.move + 'access_token=' + data.access_token
                   form.openid = openIds
               }  

                request({method:'POST', url: url, body: form, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                       
                    } else {
                        throw new Error('move group fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.deleteGroup = function(id){
    var that = this

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.group.del + 'access_token=' + data.access_token
                var form = {
                    group: {
                        id: id
                    }
                }

                request({method:'POST', url: url, body: form, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                       
                    } else {
                        throw new Error('delete group fails')
                    }
                }) 
           })         
   })  
}

//获取用户信息
Wechat.prototype.remarkUser = function(openId, remark){
    var that = this

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.user.remark + 'access_token=' + data.access_token
                var form = {
                    openid:　openId,
                    remark: remark
                }

                request({method:'POST', url: url, body: form, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                       
                    } else {
                        throw new Error('remark user fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.fetchUser = function(openIds, lang){
    var that = this
    
    var lang = lang || 'zh_CN'


   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
               var options　= {
                   json: true
               }

               if(_.isArray(openIds)){
                   options.url = api.user.batchFetch + 'access_token=' + data.access_token
                   options.method = 'POST'
                   options.body = {
                       user_list : openIds
                   }
               }
               else{
                   options.url = api.user.fetch + 'access_token=' + data.access_token + '&openid=' + openIds + '&lang=' + lang
               }

                request(options, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                       
                    } else {
                        throw new Error('fetch users fails')
                    }
                }) 
           })         
    })  
}

Wechat.prototype.listUsers = function(openId){
    var that = this

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.user.list + 'access_token=' + data.access_token
                
                if(openId){
                    url += '&next_openid=' + openId
                }

                request({method:'GET', url: url, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                       
                    } else {
                        throw new Error('list users fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.sendByGroup = function(type, message, groupId){
    var that = this
    var msg = {
        filter:{},
        msgtype:type
    }
    msg[type] = message
    if(!groupId){
        msg.filter.is_to_all = true
    }else{
        msg.filter.is_to_all = false
        msg.filter.group_id = groupId
    }


   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.mass.group + 'access_token=' + data.access_token

                request({method:'POST', url: url,body:msg, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                       
                    } else {
                        throw new Error('sell by group fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.sendByOpenId = function(type, message, openIds){
    var that = this

    var msg = {
        touser:openIds,
        msgtype:type
    }

   msg[type] = message

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.mass.openId + 'access_token=' + data.access_token

                request({method:'POST', url: url, body:msg, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                        
                    } else {
                        throw new Error('sell by openid fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.deleteMass = function(msgId){
    var that = this

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.mass.del + 'access_token=' + data.access_token
                var form = {
                    msg_id : msgId
                }

                request({method:'POST', url: url,body:form, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                        
                    } else {
                        throw new Error('delete Mass fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.previewMass = function(type, message, openId){
    var that = this
    var msg = {
        touser:openId,
        msgtype:type
    }
    msg[type] = message

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.mass.preview + 'access_token=' + data.access_token

                request({method:'POST', url: url,body:msg, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                        
                    } else {
                        throw new Error('preview mass fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.checkMass = function(msgId){
    var that = this
    var form = {
        msg_id : msgId
    }

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.mass.check + 'access_token=' + data.access_token

                request({method:'POST', url: url,body:form, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                        
                    } else {
                        throw new Error('check mass fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.createMenu = function(menu){
    var that = this

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){  
                var url = api.menu.create + 'access_token=' + data.access_token

                request({method:'POST', url: url,body:menu, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                        
                    } else {
                        throw new Error('create menu fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.getMenu = function(){
    var that = this

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.menu.get + 'access_token=' + data.access_token

                request({url: url, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                        
                    } else {
                        throw new Error('get menu fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.deleteMenu = function(){
    var that = this

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.menu.del + 'access_token=' + data.access_token

                request({url: url, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                        
                    } else {
                        throw new Error('delete menu fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.currentSelfMenu = function(){
    var that = this

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){
                var url = api.menu.current + 'access_token=' + data.access_token

                request({url: url, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                        
                    } else {
                        throw new Error('get current selfmenu fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.semantic = function(semanticData){
    var that = this

   return new Promise(function(resolve, reject){
       that
           .fetchAccessToken()
           .then(function(data){  
                var url = api.semantic + 'access_token=' + data.access_token
                semanticData.appid = data.appID

                request({method:'POST', url: url,body:semanticData, json: true}, function (error, response, body) {
                    if (!error && response.statusCode === 200) {
                        var _data = body;
                        
                        if(_data){
                            resolve(_data);
                        }                     
                        
                    } else {
                        throw new Error('semantic data fails')
                    }
                }) 
           })         
   })  
}

Wechat.prototype.reply = function(){  //实现完整的回复逻辑，放回给客户端xml格式数据
    var content = this.body   //此时的this.body是json数据格式
    var message = this.weixin
    var xml = util.tpl(content, message)  //将content与message中的内容提出整合，此时已经变成了xml格式
    //console.log('xml:'+ xml)

    this.status = 200
    this.type = 'application/xml'
    this.body = xml

}

module.exports = Wechat
```

wx/reply.js  处理客户端消息
```js
'use strict'

var config = require('../config')
var Wechat = require('../wechat/wechat')
var wechatApi = new Wechat(config.wechat)
var menu = require('./menu')

/*wechatApi.deleteMenu().then(function(){
    return wechatApi.createMenu(menu)
}).then(function(msg){
    console.log(msg)
})*/

exports.reply = function *(next){
 
    var message = this.weixin

    if(message.MsgType === 'event'){
        if(message.Event === 'subscribe'){
            if(message.EventKey) {
                console.log('扫二维码进来：'+ message.EventKey +' '+ message.ticket)
            }
            this.body =  '么么哒'
        }
        else if(message.Event === 'unsubscribe'){
            console.log('无情取关！')
            this.body = ' '
        }
        else if(message.Event === 'LOCATION'){
            this.body = '您上报的位置是：' + message.Latitude + '/' + message.Longitude + '-' +message.Precision
        }
        else if(message.Event === 'CLICK'){
            this.body = '您点击了菜单：'+ message.EventKey
        }
        else if(message.Event === 'SCAN'){
            console.log('关注后扫二维码'+ message.EventKey + ' '+ message.Ticket)
            this.body = '您扫了二维码哦'
        }
        else if(message.Event === 'VIEW'){
            this.body = '您点击了菜单中的连接：'+ message.EventKey
        }
        else if(message.Event === 'scancode_push'){
            console.log(message.ScanCodeInfo.ScanType)
            console.log(message.ScanCodeInfo.ScanResult)
             this.body = '您点击了菜单中的：'+ message.EventKey
        }
        else if(message.Event === 'scancode_waitmsg'){
             console.log(message.ScanCodeInfo.ScanType)
             console.log(message.ScanCodeInfo.ScanResult)
             this.body = '您点击了菜单中的：'+ message.EventKey
        }
        else if(message.Event === 'pic_sysphoto'){
             console.log(message.SendPicsInfo.Count)
             console.log(message.SendPicsInfo.PicList)
             this.body = '您点击了菜单中的：'+ message.EventKey
        }
        else if(message.Event === 'pic_photo_or_album'){
             console.log(message.SendPicsInfo.Count)
             console.log(message.SendPicsInfo.PicList)
             this.body = '您点击了菜单中的：'+ message.EventKey
        }
        else if(message.Event === 'pic_weixin'){
             console.log(message.SendPicsInfo.Count)
             console.log(message.SendPicsInfo.PicList)
             this.body = '您点击了菜单中的：'+ message.EventKey
        }
        else if(message.Event === 'location_select'){
             console.log(message.SendLocationInfo.Location_X)
             console.log(message.SendLocationInfo.Location_Y)
             console.log(message.SendLocationInfo.Scale)
             console.log(message.SendLocationInfo.Label)
             console.log(message.SendLocationInfo.Poiname)
             this.body = '您点击了菜单中的：'+ message.EventKey
        }
    }
    else if(message.MsgType === 'text'){
        var content = message.Content
        var reply = '你说的“' + message.Content + '” 我们没有这样的服务'

        if(content === '1'){
            reply = '天下第一'
        }
        else if(content === '2'){
            reply = '天下第二'
        }
        else if(content === '3'){
            reply = '天下第三'
        }
        else if(content === '4'){
            reply = [{
                title: 'nodejs开发微信',
                description: 'nodejs是服务器端的js脚本语言，用最火的技术开发微信公众号，你还等什么',
                picUrl: 'http://www.aseoe.com/images/aseoe_nodejs_txal.jpg',
                url: 'https://nodejs.org'
            },{
                title: '前端三大框架',
                description: 'vue.js React.js angular.js是目前前端最火的三大框架',
                picUrl: 'http://5xruby.tw/uploads/course/image/57/thumb_5x_VueJs-1024x768_5x.png',
                url: 'https://cn.vuejs.org/'
            }
            ]
        }
        else if(content === '5'){
            var data  = yield wechatApi.uploadMaterial('image', __dirname + '/2.jpg')

            reply = {
                type:'image',
                mediaId: data.media_id
            }
            console.log(data.media_id)
        }
        else if(content === '6'){
            var data  = yield wechatApi.uploadMaterial('video', __dirname + '/6.mp4')

            reply = {
                type:'video',
                title: '跪着唱征服',
                description: '试试这是个怎样的感觉',
                mediaId: data.media_id
            }
        }
        else if(content === '7'){
            var data  = yield wechatApi.uploadMaterial('image', __dirname + '/2.jpg')

            reply = {
                type:'music',
                title: '《李白》--李荣浩',
                description: '轻松时刻',
                musicUrl: 'http://link.hhtjim.com/163/27678655.mp3',
                hqMusicUrl: 'http://link.hhtjim.com/163/27678655.mp3',
                thumbMediaId: data.media_id
            }
        }
        else if(content === '8'){
            var data  = yield wechatApi.uploadMaterial('image', __dirname + '/2.jpg',{type:'image'})

            reply = {
                type:'image',
                mediaId: data.media_id
            }
        }
        else if(content === '9'){
            var data  = yield wechatApi.uploadMaterial('video', __dirname + '/6.mp4',{type:'video', description:'{"title":"VIDEO_TITLE","introduction":"INTRODUCTION"}'})

            reply = {
                type:'video',
                title: '跪着唱征服',
                description: '试试这是个怎样的感觉',
                mediaId: data.media_id
            }
        }
        else if(content === '10'){
            var picData  = yield wechatApi.uploadMaterial('image', __dirname + '/2.jpg',{})

            var media = {
                articles:[{
                    title:'angularjs入门',
                    thumb_media_id:picData.media_id,
                    show_cover_pic:1,
                    author:'Leo',
                    digest:'关于angularjs摘要',
                    content:'augular是目前前端最火框架之一',
                    content_source_url:'https://docs.angularjs.org/'
                    }
                ]
                
            }

           var  data = yield wechatApi.uploadMaterial('news', media, {})
           console.log('我要的mediaId：'+ data.media_id)
           data = yield wechatApi.fetchMaterial(data.media_id, 'news', {})

           console.log(data)

           var items = data.news_item
           var news = []

           items.forEach(function(item){
               news.push({
                   title: item.title,
                   description: item.digest,
                   picUrl: picData.url,
                   url: item.url
               })
           })

           reply = news

        }
        else if(content === '11'){
            var counts  = yield wechatApi.countMaterial()

            console.log(JSON.stringify(counts))

            var results = yield [
                wechatApi.batchMaterial({
                    type: 'image',
                    offset: 0,
                    count: 10
                }),
                wechatApi.batchMaterial({
                    type: 'video',
                    offset: 0,
                    count: 10
                }),
                wechatApi.batchMaterial({
                    type: 'vioce',
                    offset: 0,
                    count: 10
                }),
                wechatApi.batchMaterial({
                    type: 'news',
                    offset: 0,
                    count: 10
                })
            ]

            console.log(JSON.stringify(results))

            reply = 'ok'
        }
        else if(content === '12'){
           /* var group1 = yield wechatApi.createGroup('wechat1')
            console.log('新分组wechat1:')
            console.log(group1)

            var groups = yield wechatApi.fetchGroup()
            console.log('加了wechat1的分组:')
            console.log(groups)

            var group2 = yield wechatApi.checkGroup(message.FromUserName)
            console.log('查看自己的分组:')
            console.log(group2)

            var group3 = yield wechatApi.updateGroup(100, 'wechat_update')
            console.log('把wechat1分组改为wechat_update')
            console.log(group3)

            groups = yield wechatApi.fetchGroup()
            console.log('查看把wechat1分组改为wechat_update后的分组:')
            console.log(groups)

            var group4 = yield wechatApi.moveGroup(message.FromUserName, 2)
            console.log('把我移动到2组')
            console.log(group4)

            groups = yield wechatApi.fetchGroup()
            console.log('查看把我移动到2组后的分组:')
            console.log(groups)

            var group5 = yield wechatApi.moveGroup([message.FromUserName], 0)
            console.log('批量移动 把我移动到0组')
            console.log(group5)

            var groups = yield wechatApi.fetchGroup()
            console.log('查看 批量移动 把我移动到0组后的分组:')
            console.log(groups)
*/          
            var groups = yield wechatApi.fetchGroup()
            console.log(groups)
            var group_delete = yield wechatApi.deleteGroup(101)
            console.log('删除101')
            console.log(group_delete)
            groups = yield wechatApi.fetchGroup()
            console.log('删除101后的分组')
            console.log(groups)

            reply = 'Groups done'

        }
        else if(content === '13'){
          /* var remark = yield wechatApi.remarkUser(message.FromUserName, '超级大帅哥')
            console.log(remark)
            */
            var user = yield wechatApi.fetchUser(message.FromUserName)
            console.log(user)

           /* var openIds = [{
                openid : message.FromUserName, 
                lang:'zh_CN'
            }]
            var users = yield wechatApi.fetchUser(openIds)
            console.log(users)
            */
            reply = JSON.stringify(user)

        }
        else if(content === '14'){
            var list = yield wechatApi.listUsers()
            console.log(list)

            reply = list.total
        }
        else if(content === '15'){
            var mpnews = {
                media_id : 'fTn2AjmZW41YaIc0iw-86l9rCE_Ex4LNtT_SXvISNFQ'
            }

            var text = {
                content: '天要下雨了'
            }

            var msgData = yield wechatApi.sendByGroup('text', text, 0)

            console.log(msgData)
            reply = '内容如下'
        }
        else if(content === '16'){
            var mpnews = {
                media_id : 'fTn2AjmZW41YaIc0iw-86l9rCE_Ex4LNtT_SXvISNFQ'
            }
          
           /* var text = {
                content: '天一直在打雷还没下雨'
            }
  */
            var msgData = yield wechatApi.previewMass('mpnews', mpnews, 'okFRawx2UAkxdtKG1i9QNGzgXLA4')

            console.log(msgData)
            reply = '内容如下'
        }
         else if(content === '17'){
             var msgData = yield wechatApi.checkMass('1000000010')
             console.log(msgData)
             reply = 'done'
         }
         else if(content === '18'){
            
                wechatApi.createMenu(menu).then(function(msg){
                console.log('菜单生成状态：'+ JSON.stringify(msg))
            })
             /*var createMenu = yield wechatApi.createMenu(menu)
             console.log(createMenu)
             */
             reply = 'done'
         }
         else if(content === '19'){
            var semanticData = {
                query:"查一下明天从北京到上海的南航机票",
                city:"北京",
                category: "flight,hotel",
                uid: message.FromUserName
            }

            var _semantic = yield wechatApi.semantic(semanticData)
            console.log(_semantic)
             reply = JSON.stringify(_semantic)
         }

        this.body = reply
    }
    

    yield next
}
```


### 在koa中使用ejs模板 与 heredoc模块
heredoc模块是作用是封装模板  
比如：
```
'use strict'

var ejs = require('ejs')
var heredoc = require('heredoc')

var tpl = heredoc(function(){/*
    <xml> 
        <ToUserName><![CDATA[<%= toUserName %>]]></ToUserName>
        <FromUserName><![CDATA[<%= fromUserName %>]]></FromUserName> 
        <CreateTime> <%= createTime %> </CreateTime>
        <MsgType><![CDATA[<%= msgType%>]]></MsgType>
        <% if (msgType === 'text') {%>
            <Content><![CDATA[<%= content%>]]></Content>
        <% } else if(msgType === 'image') { %>
            <Image>
                <MediaId><![CDATA[<%= content.mediaId %>]]></MediaId>
            </Image>
         <% } else if(msgType === 'video') { %>
             <Video>
                <MediaId><![CDATA[<%= content.mediaId %>]]></MediaId>
                <Title><![CDATA[<%= content.title %>]]></Title>
                <Description><![CDATA[<%= content.description %>]]></Description>
            </Video> 
         <% } else if(msgType === 'voice') { %>
             <Voice>
                <MediaId><![CDATA[<%= content.mediaId %>]]></MediaId>
            </Voice>
        <% } else if(msgType === 'music') { %>
            <Music>
                <Title><![CDATA[<%= content.title %>]]></Title>
                <Description><![CDATA[<%= content.description %>]]></Description>
                <MusicUrl><![CDATA[<%= content.musicUrl %>]]></MusicUrl>
                <HQMusicUrl><![CDATA[<%= content.hqMusicUrl%>]]></HQMusicUrl>
                <ThumbMediaId><![CDATA[<%= content.thumbMediaId %>]]></ThumbMediaId>
            </Music> 
        <% } else if(msgType === 'news') { %>
            <ArticleCount><%= content.length %></ArticleCount>           
            <Articles>
            <% content.forEach(function(item){ %>
                <item>
                    <Title><![CDATA[<%= item.title %>]]></Title> 
                    <Description><![CDATA[<%= item.description %>]]></Description>
                    <PicUrl><![CDATA[<%= item.picUrl %>]]></PicUrl>
                    <Url><![CDATA[<%= item.url %>]]></Url>
                </item>
            <% }) %>
            </Articles>
        <% } %>
    </xml>

*/})

var compiled = ejs.compile(tpl)

exports = module.exports = {
    compiled : compiled
}
```

以上暴露了compiled方法，但只是做好了封装模板而已，还缺少json格式数据  
结合json数据
```
var tpl = require('以上文件路径')

var data = {
    msgType : 'text',
    content : 'hello world'
}

var xml = tpl.compiled(data)
```
或者可以使用render方法将模板和数据一并结合
```
var xml = ejs.render(tpl, data)
```

******
这篇文档算不上很好的教程，只是将开发过程的关键代码展示了出来，还有介绍了，heredoc封装xml模板，理解了用法。  

其实做好的教程是，渐进式的从头到尾开发微信公众号，使流程变得清晰杳然，能学到更多。但是鉴于个人微信公众号无法认证，缺失了很有接口权限，申请的测试公众号配置时常出问题，开发困难，时间成本开销很大。所以只展示了关键代码。  

思考了一下，项目的关键点可以写个渐进式的教程，展示开发的优化过程，让自己更深入理解知识点。

这篇笔记就写到这吧，全部代码放在github上。




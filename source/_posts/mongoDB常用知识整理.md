---
title: mongoDB常用知识整理
date: 2017-7-10
tags: [mongoDB]
---


# mongoDB常用知识整理

## 数据库恢复

     cd bin 
进去bin目录，在该目录下有工具`mongorestore`，`mongoimport`，`mongodump`,`mongoexport`
恢复数据库主要用`mongorestore`

使用命令：`mongorestore -d  baifuCommune_db  --noIndexRestore   C://pathname`
`--noIndexRestore`  是忽略数据库版本不同引起的索引问题
`-d`后接数据库名
`C://pathname`是数据库目录


>注入此类请参考：[mongoDB数据库导入、导出、备份][1]

## mongoDB基础命令

    show dbs
    use dbname
    show collections
    db.collectionName.insert({x :  1})
    for(i = 3; i< 100; i++)db.collectionName.insert({ x ： 1})
    db.collectionName.find()
    db.collectionName.find({ x : 1 })
    db.collectionName.find().count()
    
    
    db.collectionsName.find().skip(3).limit(2).sort({ x : 1 })
    db.collectionName.drop()
    db.collectionName.update({ x : 1}, { x : 999 })    第二个json对象替换掉原有的对象
    db.collectionName.update({ z : 100}, { $set : { y : 99} })   只是更新了y值
    db.collectionName.update({ y : 100}, { y : 999 }, true)    假如更新的数据不存在就插入新数据
    db.collectionName.update({ x : 1}, { $set : { x : 999} }, false, true)   将第四个参数设为true，更新多条x为1的数据
    db.collectionName.remove( { x : 2 } )   删除x为2的数据
    
    db.collectionName.getIndexes()    获取索引
    db.collectioname.ensureIndex( { x ： 1 } )     创建索引   
    db,collectionName.ensureIndex( { x : 1  }, { exprieAfterSeconds : 30} )  x为key，1表示升序，相反-1表示降序，exprieAfterSeconds 值为秒数，表示索引失效被删除的时间，到了时间不一定准时被删除
    
    
    db.collectionName.ensureIndex( { "article"  : "text"} )  创建全文索引
    db.collectionName.find( $text : { $search : { " aa  bb  cc "}})  全文查询，默认是或查询
    db.collectionName.find( $text : { $search : { " aa  bb  -cc "}}   包含aa或bb，但不包含cc
    db.collectionName.find( $text : { $search : { "  \"aa\"  \" bb\"   \"cc\" " } } )  反斜杠是转义符， 既包含   
    db.collectionName.find( $text : { $search : { " aa  bb  cc "} }, { score :{$meta : "textScore"}}  ).sort({ score :{$meta : "textScore"})   相识度查询，文档里 会增加一个相识度的值，sort排序
    
    地理位置查询
    db.collectionName.ensureIndex({ "w"  : "2d"})
    db.collectionName.insert( { w : [ 12, 12]})   经纬度  -180--180  -90--90
    db.collectionName.find( { w :{$near : [1,1]}})    返回一百个距离这个点最近的地点
    db.collectionName.find( { w :{$ geowithin : { $box : [ [0,0], [3,3] ]} } })    矩形内的点
    db.collectionName.find( { w :{$ geowithin : { $center : [ [0,0], 5 ]} } })   圆型
    db.collectionName.find( { w :{$ geowithin : { $ ploygon : [ [0,0], [3,3], [5,5] ]} } })  多边形
    db.runCommand({ geoNear : "collectionName", near : [1,3], maxDistance : 10, num: 10})


  [1]: http://www.cnblogs.com/yangxia-test/p/3983271.html
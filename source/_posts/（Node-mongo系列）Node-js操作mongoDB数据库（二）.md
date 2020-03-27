---
title: （Node-mongo系列）Node-js操作mongoDB数据库（二）
---
####简介
>MongoDB is a document database designed for ease of development and scaling。

`mongoDB`是一个文档数据库，旨在简化开发和扩展.

>A record in MongoDB is a document, which is a data structure composed of field and value pairs. MongoDB documents are similar to JSON objects. The values of fields may include other documents, arrays, and arrays of documents.

类似其他关系型数据库，在`mongoDB`中一个记录叫做`document`，是由名称和值对组成的数据结构。类似`json`对象。
####类比mysql关系数据库
类比如下图所示，document 类似 一行record，connection类似一张表table；
![](https://upload-images.jianshu.io/upload_images/5541401-207823dd8b0e1b15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####简单shell操作
管理员身份打开`cmd`，运行`mongo`，前提是将`bin`目录写入到环境变量中。
安装参考：https://www.jianshu.com/p/59079c9832e9
1.显示所有数据库和正在使用的默认数据库`test`
```
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
> db
test
>
```
2.`use`切换数据库，没有的话新建一个数据库，新建一个`collection`,相当于关系数据库中的表，插入一条数据`db.myCollection.insertOne()`,插入多条数据`db.myCollection.insert({x:2},{x:3})`
```
> use myNewDatabase
switched to db myNewDatabase
> db.myCollection.insertOne({x: 1})
{
        "acknowledged" : true,
        "insertedId" : ObjectId("5cbd255be647b2c115594ba6")
}
> db.myCollection.insert({x:2},{x:3})
WriteResult({
        "nInserted" : 0,
        "writeError" : {
                "code" : 13297,
                "errmsg" : "db already exists with different case already have:
[myNewDataBase] trying to create [myNewDatabase]"
        }
})
```

3.删除数据库`db.dropDatabase()`
```
> db.dropDatabase()
{ "dropped" : "myNewDatabase", "ok" : 1 }
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```

####nodejs中操作mongodb
前提是数据库服务已经启动，管理员身份打开 `cmd`
```
net start mongodb  //启动服务
net stop mongodb  //停止服务
```
* ######Init初始化文件目录 
新建一个文件夹例如`node-mongodb`，使用`npm init`初始化`package.json`，`npm install mongodb -S`
`mongoDB`的版本是`3.2`版本，以下用`3.2`版本的写法
新建一个`index.js`文件
```
const MongoClient = require('mongodb').MongoClient;
const assert = require('assert');

//connection url
const url = 'mongodb://127.0.0.1:27017';

//database name
const dbname = 'myNewDataBase';

//create new mongodbClient
const client = new MongoClient(url,{ useNewUrlParser: true });

client.connect((err) => {
    assert.equal(null,err);
    console.log('Connected successfully to server');
    client.close();
})
```
`node index.js`，数据库链接成功
![](https://upload-images.jianshu.io/upload_images/5541401-0916651d61c55783.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* #####操作数据库（insert，find，update，delete，indexes（索引））
```
//index.js
const MongoClient = require('mongodb').MongoClient;
const assert = require('assert');

//connection url
const url = 'mongodb://127.0.0.1:27017';

//database name
const dbname = 'myNewDataBase';

//create new mongodbClient
const client = new MongoClient(url,{ useNewUrlParser: true });

client.connect((err) => {
    assert.equal(null,err);
    console.log('Connected successfully to server');
    //3.0版本新写法
    const db = client.db(dbname);
    insertDocuments(db,() => {
        // find
        // findDocuments(db,() => {
        //     client.close();
        // })
        //update
        // updateDocument(db,() => {
        //     //remove
        //     removeDocument(db,() => {
        //         client.close();
        //     });
        // });
        indexCollection(db,() => {
            client.close();
        })
    })
})

//insert
const insertDocuments = (db,callback) => {
    //get the document collection(相当于拿到一个表，没有的话创建一个表)
    const collection = db.collection('myCollection');
    collection.insertOne(
        {a: 4}
    ,(err,result) => {
        assert.equal(err, null);
        console.log("Inserted 3 documents into the collection");
        callback(result);
    })
}

//find
const findDocuments = (db,callback) => {
    const collection = db.collection('myCollection');
    collection.find({}).toArray((err,docs) => {
        console.log('Found the following records')
        console.log(docs);
        callback(docs);
    });
}

//update
const updateDocument = (db,callback) => {
    //get the document collection
    const collection = db.collection('myCollection');
    //update
    collection.updateOne({a: 2},{ $set: { b: 1} },(err,result) => {
        assert.equal(err, null);
        console.log("Updated the document with the field a equal to 2");
        callback(result);
    })
}

//remove
const removeDocument = (db,callback) => {
    const collection = db.collection('myCollection');
    collection.deleteOne({ a: 3},(err,result) => {
        assert.equal(err, null);
        console.log("Removed the document with the field a equal to 3");
        callback(result);
    })
}

//Index a Collection
const indexCollection = (db,callback) => {
    db.collection('myCollection').createIndex(
        { "a": 1 },
          null,
          function(err, results) {
            console.log(results);
            callback();
        }
    );
}
```
运行`node index.js`，显示如下结果
```
E:\workCode\node-mongodb>node index.js
Connected successfully to server

E:\workCode\node-mongodb>node index.js
Connected successfully to server
Inserted 3 documents into the collection

E:\workCode\node-mongodb>node index.js
Connected successfully to server
Inserted 3 documents into the collection
Found the following records
[ { _id: 5cbd6ee9ef77ab2d6c4a89dd, a: 4 },
  { _id: 5cbd747a716b2b08e463ccee, a: 4 } ]

E:\workCode\node-mongodb>node index.js
Connected successfully to server
Inserted 3 documents into the collection
Updated the document with the field a equal to 2

E:\workCode\node-mongodb>node index.js
Connected successfully to server
Inserted 3 documents into the collection
Updated the document with the field a equal to 2
Removed the document with the field a equal to 3

E:\workCode\node-mongodb>node index.js
Connected successfully to server
Inserted 3 documents into the collection
a_
```
参考：
https://docs.mongodb.com/manual/mongo/#start-the-mongo-shell-and-connect-to-mongodb
http://www.mongoing.com/archives/3818
http://mongodb.github.io/node-mongodb-native/3.2/quick-start/quick-start/

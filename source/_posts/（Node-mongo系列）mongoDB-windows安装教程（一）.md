---
title: （Node-mongo系列）mongoDB-windows安装教程（一）
---
官方下载安装包
https://www.mongodb.com/download-center/community
直接下载msi文件
安装步骤如下：
#####1.install mongoDB
直接双击下载好的`msi`文件，根据提示一步步安装，我是直接安装默认存放位置`C:\Program Files\MongoDB\`；
![](https://upload-images.jianshu.io/upload_images/5541401-aecdb0bbf7150e53.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####2.Create database directory
安装完成之后需要新建一个文件夹用来`存放数据`，因为数据量比较大，安装在`D盘下的data文件夹`里面，在这里建一个`db文件夹`，一个`log的日志文件夹`；
![](https://upload-images.jianshu.io/upload_images/5541401-8fa4183c3accfaac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####3.Start your MongoDB database
以管理员身份运行`cmd`，从官网拷贝命令修改数据的位置`d盘`，以及`版本`是`4.0`，运行命令，
```
"C:\Program Files\MongoDB\Server\4.0\bin\mongod.exe" --dbpath="d:\data\db"
```
出现以下日志说明安装成功了，端口是`27017`
```
2019-04-18T11:28:24.645+0800 I NETWORK  [initandlisten] waiting for connections
on port 27017
```
![](https://upload-images.jianshu.io/upload_images/5541401-bbf70a22f41364b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####4. Connect to MongoDB
运行命令，可以看到`mongodb`已经启动了
```
"C:\Program Files\MongoDB\Server\4.0\bin\mongo.exe"
```
```
C:\windows\system32>"C:\Program Files\MongoDB\Server\4.0\bin\mongo.exe"
MongoDB shell version v4.0.9
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("28fbc481-853c-485f-9c4b-388377327f5f")
}
MongoDB server version: 4.0.9
Welcome to the MongoDB shell.
```
![](https://upload-images.jianshu.io/upload_images/5541401-e14c36bd5e813ed2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####5.查看数据库
`MongoDB Shell`是`MongoDB`自带的交互式`Javascript shell`,用来对`MongoDB`进行操作和管理的交互式环境。
直接输入`show dbs`；
```
---
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
>
```
#####6.启动、停止mongo服务（都是管理员身份打开cmd）
```
net start MongoDB  //启动
net stop MongoDB  //停止
```
![](https://upload-images.jianshu.io/upload_images/5541401-994407d021ad1843.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
卸载MongoDB，通过`sc.exe delete MongoDB`，然后到应用程序里面卸载即可。

最后可以将mongoDB的bin目录写环境变量中，可以全局启动`mongoDB`的` shell`。
```
C:\Program Files\MongoDB\Server\4.0\bin
```
![](https://upload-images.jianshu.io/upload_images/5541401-9a7788b9970d1cbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 管理员身份打开`cmd`，输入`mongo`，即可打开`mongo shell`
![](https://upload-images.jianshu.io/upload_images/5541401-4d7807089fea4e83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
`mongoDB`数据库连接成功，默认是`27017`端口
```
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
```

参考：
https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows-unattended/#start-your-mongodb-database










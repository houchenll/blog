---
layout: default
title: 006 linux服务器mongodb服务环境搭建与连接
---

# linux服务器mongodb服务环境搭建与连接


## 服务器安装配置mongodb服务

### 下载安装包
　　centos上使用的mongodb需要是redhat对应的版本，安装包选择CommunityServer -> Linux -> RHEL 7 Linux 64-bit x64([mongodb-linux-x86_64-rhel70-3.4.7.tgz][1])。

### 从mac上传安装包到centos
　　从mac电脑上，通过shell，使用scp命令上传安装包到centos。  
　　scp(security copy)是unix命令，上传过程中会加密。命令使用格式为：  
　　`scp -i secret_key local_file_path username@ip_address:remote_file_path`。  
　　如`scp -i secret_key ./1.txt root@123.123.123.123:/data/2.txt`  
　　　　-i secret_key指定连接到远程服务器的密钥文件。  
　　　　./1.txt指定要上传的本地文件地路径。  
　　　　`root@123.123.123.123:/data/2.txt`指定要上传的服务器的ip地址、用户名及文件在服务器上存储的路径。  

### 解压
``` shell
cd /data
tar -zxvf mongodb-linux-x86_64-rhel70-3.4.7.tgz
mv mongodb-linux-x86_64-rhel70-3.4.7 mongodb
```

### 安装
　　把mongodb安装包移动到`/usr/local/server`，并添加一些配置文件，供后续使用，如`data`, `logs`。在`/etc`下添加配置文件`mongod.conf`。
``` shell
mkdir /usr/local/server
mv /data/mongodb /usr/local/server
cd /usr/local/server/mongodb/
mkdir data
touch logs
touch /etc/mongod.conf
```

### 添加mongod命令到PATH
　　首先在`/etc/profile`文件最后添加`export PATH=$PATH:/usr/local/server/mongodb/bin`。然后执行`source /etc/profile`使配置生效。这样，在所有路径中都能使用mongodb命令。  

### 编辑配置文件
　　编译mongodb.conf文件，指定数据库存放位置、log文件等。注意，首次启动时，因为还没有添加用户和密码，所以auth配置要设置为false，这样所有用户都能操作数据库，有很大的安全风险，所以随后会介绍如何为数据库添加用户和密码，然后将auth设置为true，这样再次启动mongodb服务后，访问该数据库就需要使用用户和密码登录，否则访问不了数据库，这样就保证了安全性。  
　　注意，不能设置bindip项，如果bindip设置为127.0.0.1，那么远程客户端就不能通过服务器Ip访问服务器mongodb数据库。不设置bindip，那么远程客户端和服务器本身都可以访问服务器数据库。  
```
dbpath = /usr/local/server/mongodb/data
logpath = /usr/local/server/mongodb/logs
logappend = true
port = 27017
fork = true
auth = false
```

### 启动停止服务
　　启动服务：`mongo --config /etc/mongodb.conf`  
　　停止服务：`ps -ef | grep mongo`, `kill -9 pid`  

### 导出数据库文件到json文件中
　　使用mongoexport命令，可以将mongodb数据库导出到json/csv文件中。注意，导出时，是按collection逐个导出。命令格式如下：  
　　`mongoexport -d dbname -c collectionname -o file --type json/csv -f field`  
　　　　-d ：数据库名  
　　　　-c ：collection名  
　　　　-o ：输出的文件名  
　　　　--type ： 输出的格式，默认为json  
　　　　-f ：输出的字段，如果-type为csv，则需要加上-f "字段名"，不指定-f时默认输出所有字段  
　　例：`mongoexport -d mongotest -c users -o /home/python/Desktop/mongoDB/users.json --type json -f  "_id,user_id,user_name,age,status"`  
　　数据库导出完毕后，将json文件打包在一起，通过scp命令传输到服务器。然后服务器解压后，逐个导入到数据库。  

### 导入json文件
　　使用mongoimport命令，可以将json/csv文件中的内容导入到指定数据库指定collection中。命令格式如下：  
　　`mongoimport -d dbname -c collectionname --file filename --headerline --type json/csv -f field`  
　　　　--headerline    如果导入的格式是csv，则可以使用第一行的标题作为导入的字段


## 远程连接准备

### 修改安全组
　　至此，服务器mongodb数据库已经启动起来，并有了数据，服务器能通过`mongo`连接服务器数据库。  
　　但这时客户机不能访问服务器数据库，因为服务器的安全组设置还没有允许TCP:27017访问，需要在服务器后台管理中，在安全组中添加TCP:27017项，客户端才能通过`mongo mongodb://ip:27017/dbname`访问服务器mongodb数据库。  

### 添加用户密码
　　到目前为止，所有人都能访问并操作服务器数据库，连接到数据库时，会打出很多提示，意思是不安全，需要添加安全设置。  
　　但是这样不安全，所以需要做两件事，第一、为指定数据库添加用户和密码，这样，只有在连接时使用这个用户和密码登录，才能操作该数据库；第二、将数据库配置文件中的auth打开为true，并重启服务，这样下次再连接数据库时，就需要指定用户和密码，才能操作数据库。使用用户和密码连接到数据库时，就不会再提示那些当前连接不安全的信息。  
　　添加用户和密码，需要先用mongo连接到数据库，然后指定数据库，再使用添加命令添加，如下：  
``` mongodb
use dbname
db.createUser({user:"root",pwd:"root",roles:[{role:"readWrite",db:"dbname"}]})
```
　　添加用户密码成功后，将数据库配置文件mongodb.conf中的auth修改为true，再次启动服务器。这样，下次再连接数据库，就需要使用用户和密码登录，才能权限操作数据库。  


## 远程连接mongodb数据库

　　在保证服务器安全组`TCP:27017`已添加，数据库已创建用户密码，启动数据库时auth为true等条件后，各客户端就能安全地连接服务器数据库了，且连接成功后不会再提示不安全这样的信息。  
　　各客户端访问服务器mongodb数据库，使用同样的url：`mongodb://username:pwd@ip:port/dbname`  

### 服务器本地shell连接
　　服务器本地连接，也需要指定用户和密码，否则不能访问数据库中内容。  
　　`mongo mongodb://username:pwd@127.0.0.1:port/dbname`  

### 客户端shell连接
　　`mongo mongodb://username:pwd@ip:port/dbname`  

### python连接
``` python
# -*- coding:utf-8 -*-
from pymongo import MongoClient

client = MongoClient('mongodb://username:pwd@ip:27017/dbname')
db = client.dbname
cursor = db.collectionname.find({}, {"_id": 1, "name": 1})
count = cursor.count()
if count > 0:
    for doc in cursor:
        print doc['_id'], doc['name']
```

### node连接
``` javascript
const MongoClient = require('mongodb').MongoClient;
const assert = require('assert');

const mongoUrl = 'mongodb://username:pwd@ip:27017/dbname';

MongoClient.connect(mongoUrl, (err, db) => {
    assert.equal(null, err);
    console.log('connected successfully to server');

    var author = db.collection('author');
    author.find({}, {'_id': 1, 'name': 1}).toArray((err, docs) => {
        assert.equal(err, null);
        console.log('found the following records');
        console.log(docs);
    });
});
```

[1]: https://www.mongodb.com/download-center?jmp=tutorials&_ga=2.103561536.1195125389.1503368882-1420124469.1502781833#community "mongodb-linux-x86_64-rhel70-3.4.7.tgz"

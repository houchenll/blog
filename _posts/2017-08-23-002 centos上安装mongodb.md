---
layout: default
title: 002 centos上安装mongodb
---

# centos上安装mongodb

## 上传安装包到centos
　　下载centos上使用的mongodb安装包时，选择CommunityServer -> Linux -> RHEL 7 Linux 64-bit x64版本([mongodb-linux-x86_64-rhel70-3.4.7.tgz](https://www.mongodb.com/download-center?jmp=tutorials&_ga=2.103561536.1195125389.1503368882-1420124469.1502781833#community))。  
　　下载完成后，使用scp工具上传安装包到centos。 scp是unix命令，是security copy的缩写，上传过程中会加密。 上传文件命令格式如下:  
　　`scp [param] local_file_path username@ip_address:remote_file_path`  
　　一个实际的例子如下：`scp -i secret_key ./1.txt root@115.121.121.231:/data/2.txt`。 `-i secret_key`指定连接到远程服务器的密钥文件。 `./1.txt`指定要上传的本地文件地路径。 `root@115.121.121.231:/data/2.txt`指定要上传的服务器的ip地址、用户名及文件在服务器上存储的路径。  
　　这条命令在本地mac或linux电脑上运行。  

## 安装centos
　　解压。 `tar -zxvf /data/mongodb-linux-x86_64-rhel70-3.4.7.tgz`。 然后修改文件夹名为mongodb。  
　　配置数据库和log存储路径。 创建`/var/log/mongo/mongo.log`文件及`/var/lib/mongo`文件夹。 创建`/etc/mongod.conf`文件，里面填写启动配置，内容如下：  
```
systemLog:
    destination: file
    path: /var/log/mongo/mongo.log
    logAppend: true
storage:
    dbPath: /var/lib/mongo
net:
    bindIp: 127.0.0.1

```
　　启动数据库服务。 `/data/mongodb/bin/mongod --fork --config /etc/mongod.conf`  
　　连接数据库。 `/data/mongodb/bin/mongo`

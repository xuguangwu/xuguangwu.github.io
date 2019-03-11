---
title: 搭建mongo服务
categories:
 - DB
tags: Mongo
---

### 镜像
docker pull mongo

### 宿主机相关目录
cd /mnt
mkdir mongodb
cd ./mongodb
mkdir data
mkdir backup
mkdir logs

### 运行容器
docker run --name mongo -p 27017:27017 -v /Users/clear/data/mongodb/data:/data/db -v /Users/clear/data/mongodb/backup:/data/backup -d mongo --auth

### 进入容器
docker exec -it mongo mongo admin

### 创建用户
db.createUser({ user: 'xuguangwu', pwd: 'xgw123456', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });

db.auth("xuguangwu","xgw123456");

db.createUser({ user: 'spider', pwd: 'spider', roles:[{"readWrite"}] });

db.createUser(
  {
    user: "spider",
    pwd: "spider",
    roles: [
       { role: "readWrite", db: "spider" }
    ]
  }
)


### 常见问题
1. 创建用户提示[main] Error: couldn't add user: not authorized 
docker run --name mongo -p 27017:27017 -v /Users/clear/data/mongodb/data:/data/db -v /Users/clear/data/mongodb/backup:/data/backup -d mongo

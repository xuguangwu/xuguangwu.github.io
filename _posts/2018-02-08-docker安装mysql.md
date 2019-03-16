---
title: docker安装mysql
categories:
 - DB
tags: mysql
---

## simple run
docker run --name mysql -e MYSQL_ROOT_PASSWORD=xuguangwu -d mysql:8

## completed run
docker run --name mysql -v /Users/xuguangwu/server/mysql/conf.d:/etc/mysql/conf.d \
-v /Users/xuguangwu/server/mysql/data:/var/lib/mysql \
-v /Users/xuguangwu/server/mysql/log:/var/log/mysqld.log \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=xuguangwu -d mysql:8

## creating database dumps
$ docker exec mysql sh -c 'exec mysqldump --all-databases -uroot -p"$MYSQL_ROOT_PASSWORD"' > /Users/xuguangwu/server/mysql/dump/all-databases.sql








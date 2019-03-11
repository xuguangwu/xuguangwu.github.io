---
title: docker安装postgres
categories:
 - DevOps
tags: DevOps
---

1. 运行容器
docker stop postgres && docker rm postgres && docker run -d -p 54321:5432  -m 8g --cpus=2 --name postgres -v /mnt/data/pgdata:/var/lib/postgresql/data postgres 

2. 登陆容器
su postgres
psql
create database factoring;

3. 创建角色，用户（角色与用户的区别在于角色无法登陆）
CREATE ROLE r_data_dml;
CREATE ROLE r_data_qry;
CREATE USER dolphindata  PASSWORD 'dol,908';

4. 创建schema
CREATE SCHEMA dolphin;

5. 授权
GRANT r_data_dml,r_data_qry TO dolphindata;
GRANT r_data_qry TO dolphinopr;

grant all on schema dolphin to r_data_dml;
revoke all on schema dolphin from r_data_qry;
grant select on all tables in schema dolphin to r_data_qry;

6. 删除schema
DROP SCHEMA dolphin CASCADE;


## 运行容器
docker run -d -p 54321:5432 --name postgres -v /mnt/data/pgdata:/var/lib/postgresql/data hub.yun.paic.com.cn/library/postgres:9.5.3-1

docker run -d -p 54321:5432 --name postgres -v /Users/clear/data/pgdata:/var/lib/postgresql/data postgres

docker run -d -p 54321:5432 --name postgres -v /mnt/data/pgdata:/var/lib/postgresql/data postgres

## 创建db
create database ipfpms;
create database mytest;

## 创建dml角色
CREATE ROLE r_authdata_dml WITH
  NOLOGIN
  NOSUPERUSER
  INHERIT
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION;
  
## 创建qry角色
CREATE ROLE r_authdata_qry WITH
  NOLOGIN
  NOSUPERUSER
  INHERIT
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION;

CREATE ROLE r_data_dml;
CREATE ROLE r_data_qry; 
  
## 创建authdata用户
CREATE USER authdata WITH
  LOGIN
  NOSUPERUSER
  INHERIT
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION; 
  
 CREATE USER xuguangwu WITH
  LOGIN
  NOSUPERUSER
  INHERIT
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION;

## 创建authopr用户
CREATE USER authopr WITH
  LOGIN
  NOSUPERUSER
  INHERIT
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION;



## 设置用户密码
ALTER USER authdata WITH PASSWORD 'pdata1234';
ALTER USER authopr WITH PASSWORD 'pdata1234';

ALTER USER xuguangwu WITH PASSWORD 'test123';

alter user postgres with password 'Xu,3efd5';


## 测试服务器搭建
docker stop postgres-stg && docker rm postgres-stg && docker run -d -p 64321:5432 --name postgres-stg -v /mnt/test/data/pgdata:/var/lib/postgresql/data postgres

## db数据导入导出
````
drop database factoring;
如果提示有会话连接，就需要将连接的会话进程kill
factoring=# SELECT * FROM pg_stat_activity;
kill -9 ${pid}

pg_dump factoring -c -Ft -f /tmp/factoring.tar
createdb factoring -U postgres
pg_restore -d factoring -c /tmp/factoring.tar
````




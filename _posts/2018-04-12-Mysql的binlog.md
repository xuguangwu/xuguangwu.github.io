---
title: mysql中的BinLog
categories:
 - mysql
tags: 
 - mysql
---

MySQL Server 有四种类型的日志
* Error Log

错误日志，记录 mysqld 的一些错误。

* General Query Log

一般查询日志，记录 mysqld 正在做的事情，
比如客户端的连接和断开、来自客户端每条 Sql Statement 记录信息；
如果你想准确知道客户端到底传了什么给服务端，
这个日志就非常管用了，不过它非常影响性能。

* Slow Query Log

慢查询日志，记录一些查询比较慢的 SQL 语句——这种日志非常常用，主要是给开发者调优用的。

* Binary Log

包含了一些事件，这些**事件描述了数据库的改动，如建表、数据改动等**，
也包括一些潜在改动，比如DELETE FROM users WHERE user_id = 1，
然而一条数据都没被删掉的这种情况,
但是不会记录SELECT和没有实际更新的UPDATE语句。
除非使用 Row-based logging，否则会包含所有改动数据的 SQL Statement。

**那么 Binlog 就有了两个重要的用途——复制和恢复。比如主从表的复制，和备份恢复什么的。**
通常情况 MySQL 是默认关闭 Binlog 的，所以你得配置一下以启用它。
启用的过程就是修改配置文件 my.cnf 了。
```
log-bin=master-bin
log-bin-index=master-bin.index
max_binlog_size=200M --默认1G
```
这里的 log-bin 是指以后生成各 Binlog 文件的前缀，比如上述使用master-bin，
那么文件就将会是master-bin.000001、master-bin.000002 等。
而这里的 log-bin-index 则指 binlog index 文件的名称，这里我们设置为master-bin.index。
**SHOW VARIABLES LIKE '%log_bin%';**
```
+---------------------------------+---------------------------------------+
| Variable_name                   | Value                                 |
+---------------------------------+---------------------------------------+
| log_bin                         | ON                                    |
| log_bin_basename                | /usr/local/var/mysql/master-bin       |
| log_bin_index                   | /usr/local/var/mysql/master-bin.index |
| log_bin_trust_function_creators | OFF                                   |
| log_bin_use_v1_row_events       | OFF                                   |
| sql_log_bin                     | ON                                    |
+---------------------------------+---------------------------------------+
```
mysql提供了mysqlbinlog命令来查看日志文件，在记录每条变更日志的时候，
日志文件都会把当前时间给记录下来，以便进行数据库恢复。

具体的如何恢复数据库，可以参考这篇博文，我就不再赘述了<https://www.ilanni.com/?p=7911>


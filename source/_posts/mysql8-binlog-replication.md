---
title: Mysql8下主从复制、主主复制、多源复制
date: 2019-10-15 07:27:31
tags:
  - mysql
  - replication
  - 主从复制
  - 主主复制
  - 多源复制
---

## 简述
复制的原理如下：在主库上执行的所有`DDL`和`DML`语句都会被记录到二进制日志中，也就是`binlog`。
这些日志由连接到它的从库提取。它们只是被复制到从库，并被保存为中继日志。这个过程由`IO线程`的线程负责。还有一个`SQL线程`，它按顺序执行中继日志中的语句。

## 主要步骤
不管是主主，还是多源，其基础配置都和主从差不多，所以就列出主从的基本步骤：

1. master 开启`binlog`
2. 创建复制用户
3. 设置 master 以及 slave 惟一的`server_id`
4. 在 slave 上执行`change master to`
5. 执行`start slave`

## 主从复制

### 增加binlog配置

如果不想要自定义配置，先看下系统是否开启了binlog即可
```bash
show variables like '%log_bin%'
```
![binlog](/images/mysql8-replication/log_bin.png)

增加或修改`binlog`配置，一同增加`server_id`
```
[mysqld]
log-bin=/var/lib/mysql/bin_logs/master
expire_logs_days=7
max_binlog_size=1GB

server_id=1
```
> 其它更多`binlog`配置请参考官方文档

查看`binlog`其它配置
``` bash
SHOW VARIABLES LIKE '%binlog%';
```
![binlog](/images/mysql8-replication/bin_log.png)

查看所有`binlog`及大小
```bash
show master logs(或者 show binary logs)
```
![binlog](/images/mysql8-replication/logs.png)

查看当前最新`binlog`位置
```bash
show master status;
```
![binlog](/images/mysql8-replication/master_status.png)

查看`binlog`所有位置及其它信息
```bash
show binlog events;
```
![binlog](/images/mysql8-replication/binlog_events.png)

移至下一个日志（开启新日志）
```bash
flush logs;
```
![binlog](/images/mysql8-replication/flush_logs.png)

### 创建复制用户
```bash
create user 'binlog_user'@'%' identified with mysql_native_password by 'binlog_password';
grant replication slave on *.* to 'binlog_user'@'%';
flush privileges;
```
> 需要的权限以及连接的服务器请自行修改

### 修改slave配置，增加`server_id`
```
[mysqld]
server_id=100
```

### 在slave中执行`change master to`
```bash
change master to master_host='172.22.0.2',master_user='binlog_user',master_password='binlog_password',master_log_file='master.000013',master_log_pos=155
```

> 如果配置增加未修改的话，请重启master和slave

### 开启复制
```bash
start slave
```

## 主主复制
![binlog](/images/mysql8-replication/master_master.png)
所谓主主复制，其实就是一台mysql既是主又是从，两台互为复制
需要新建一台mysql和从库开启方式一致，原主服务器再加上
```mysql
CHANGE MASTER TO MASTER HOST, MASTER USER,MASTER PASSWORD,MASTER LOG FILE,MASTER LOG POS
start slave
```

## 多源复制
![binlog](/images/mysql8-replication/master_master_slave.png)
多游、复制可用于将多台服务器备份到单台服务器，合并表分片，以及将多台服务器中的数据整合到单台服务器。多源复制在应用事务时不会执行任何冲突检测或解析，并且如果需要的话，这些任务将留给应用程序来处理。在多源复制拓扑中，从库为每个主库创建一个复制通道，以便从中接收事务。

在slave上修改配置文件：
```mysql
master_info_repository=TABLE
relay_log_info_repository=TABLE
```
如果此时从库还在同步，也需要修改
```mysql
stop slave; 
SET GLOBAL master_info_repository='TABLE';
SET GLOBAL relay_log_info_repository='TABLE';
```
重启slave即可
> 注意：多源复制下，多主对一从，需要注意复制数据冲突问题
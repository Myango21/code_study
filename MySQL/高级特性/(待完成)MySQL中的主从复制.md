[TOC]

## 文章简介

网络上关于MySQL主从复制的文章很多都是讲解如何实现，以及部分实现原理，缺乏对MySQL主从复制的全面介绍。例如主从复制的模式(半同步模式和异步同步模式)、同步的原理(binary log+position，GTID)、主从复制的常见问题都缺乏一个全面的总结。

本文针对这些部分内容做一个全面的分析与总结。本文主要的内容有MySQL**主从复制的原因、实现原理、实现步骤、半同步模式、异步同步模式、GTID同步、常见问题与解决方案**等内容。

![Snipaste_2021-05-11_15-30-41](https://gitee.com/bruce_qiq/picture/raw/master/2021-5-11/1620718267555-Snipaste_2021-05-11_15-30-41.png)

## 模式优势

在了解主从复制之前，我们先了解一下什么是主从复制。说的简单一点就是将一台MySQL服务器的数据库文件同步到其他的MySQL服务上，使得被同步的MySQL服务也能读取到我们的数据。

为什么会有主从复制呢？这里我总结了两点原因：

1. 数据容灾、备份。当我们的数据库只使用一台服务时，如果我们的数据库遭到破坏，例如黑客的攻击、人为操作的失误等等情况。这时候我们就可以在一定程度上**保障我们数据库的恢复**。

如果只为为了防止数据库的丢失，我们可以针对数据进行定时备份，为何还要搞什么主从复制，这样不是更麻烦嘛。如果我们只是定时备份数据库，可以试想一下，万一在某个备份操作还未执行的阶段，数据库出现问题，中间的这一部分数据不就没法恢复了嘛。

2. 缓解MySQL主服务的压力。当我们线上应用用户量小的时候，所有的读与写操作都在一台服务器上，这时候还不会遇到什么问题。当用户量逐渐增加，访问数据库的请求也越来越多，这时候就给MySQL服务器增加了负担，容易导致服务崩溃等问题。因此，**主从复制模式可以缓解单服务器的压力，将写操作给主服务器，读操作给从服务器，从服务器可以部署多台，分摊压力。因为在一个应用中，读的操作肯定是大于写的操作**。

![Snipaste_2021-05-11_15-48-07](https://gitee.com/bruce_qiq/picture/raw/master/2021-5-11/1620719298607-Snipaste_2021-05-11_15-48-07.png)

## 实现原理

下图是MySQL主从复制的一个原理图：

![Snipaste_2021-05-11_15-50-17](https://gitee.com/bruce_qiq/picture/raw/master/2021-5-11/1620719443119-Snipaste_2021-05-11_15-50-17.png)

1. master服务器会将SQL记录通过多dump线程写入到binary log中。

2. slave服务器开启一个io thread线程向服务器发送请求，向master服务器请求binary log。master服务器在接收到请求之后，根据偏移量将新的binary log发送给slave服务器。

3. slave服务器收到新的binary log之后，写入到自身的relay log中，这就是所谓的中继日志。

4. slave服务器，单独开启一个sql thread读取relay log之后，写入到自身数据中。

## 常见模式

常见的主从模式有如下几种，具体的模式也得看实际的业务需要。根据实际的情况，选择合适的一种架构模式。
1. 一主一从模式。

![Snipaste_2021-05-11_16-12-00](https://gitee.com/bruce_qiq/picture/raw/master/2021-5-11/1620720773893-Snipaste_2021-05-11_16-12-00.png)

2. 一主多从模式。

![Snipaste_2021-05-11_16-12-19](https://gitee.com/bruce_qiq/picture/raw/master/2021-5-11/1620720792801-Snipaste_2021-05-11_16-12-19.png)

3. 级联主从模式。

![Snipaste_2021-05-11_16-12-29](https://gitee.com/bruce_qiq/picture/raw/master/2021-5-11/1620720815664-Snipaste_2021-05-11_16-12-29.png)

4. 一主多从模式。

![Snipaste_2021-05-11_16-12-43](https://gitee.com/bruce_qiq/picture/raw/master/2021-5-11/1620720830762-Snipaste_2021-05-11_16-12-43.png)

## 配置流程

在本文演示中，采用一主一从的架构模式。

| 角色 | IP地址 | 端口号 | server-id | 
| --- | --- | --- | --- |
| master | 192.168.0.112 | 3304 | 1 |
| slave | 192.168.0.112 | 3305 | 2 |

要开启主从复制，首先要遵循下面几种条件。
1. master和server都要有自身的server-id，并且每一个server-id不能相同。

2. master开启log_bin选项。推荐将从服务器的该选项也开启。

3. master开启binlog_format=row选项。推荐从服务器也开启该选项，并且log_slave_updates也逐步开启，后期如果slave升级为master也方便扩展。

### master操作

将下面的一段配置添加到master配置文件中，重启服务让配置生效。
```mysql
server_id               = 1
log_bin                 = mysql-bin
binlog_format           = ROW
```
接下来登录到master命令行操作界面，创建一个主从复制的账号。这里创建一个账号为slave_user，密码为123456的账号。
```mysql
grant replication slave on *.* to 'slave_user'@'%' identified by '123456';
flush privileges;
```
查看master的binary log文件和postion。记录下来待会在slave上设置主节点信息时需要使用。
```mysql
mysql root@127.0.0.1:(none)> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000062 | 728      |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set
Time: 0.011s
```

### slave操作

将下面的配置项添加到slave配置文件中，重启服务让配置生效。
```mysql
server_id               = 2
log_bin                 = mysql-bin
binlog_format           = ROW
log_slave_updates		= ON
read_only				= ON
super_read_only			= ON
```
接下来登录到slave命令行操作界面，设置主节点信息。
```mysql
change master to master_host='192.168.0.112',master_port=3304,master_user='slave_user',master_password='123456',master_log_file='mysql-bin.000062',master_log_pos=728;
start slave;
stop slave;
```
1. master_host：master的IP地址。

2. master_port：master的服务端口。

3. master_user：master创建的主从复制用户。

4. master_password：master创建的主从复制用户密码。

5. master_log_file：master的binary log文件(上面在master上执行 `show master status`获取到的File值)。

6. master_log_pos：master的复制偏移量(上面在master上执行 `show master status`获取到的Position值)。

> 配置主节点的信息之后，还不能进行主从复制，还需要我们开启主从复制。

### 开启主从复制

```mysql
start slave;
```

### 断开主从复制

```mysql
stop slave;
```

### 重置主从复制信息

```mysql
reset slave all;
```

## 效果演示

在配置之前，在master上实现有这样一张表，表结构如下：
```mysql
CREATE TABLE `mysql_test`.`master_slave_demo`  (
  `id` int(10) UNSIGNED ZEROFILL NOT NULL AUTO_INCREMENT,
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 10 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;
```
在配置主从复制之前，我们在从服务器上也同样创建一张这样的表，库名和表名与master保持一致。

假设你都做好了这些工作，此时我们在master上insert一条数据，然后去slave查看是否有新的数据。
```mysql
mysql root@127.0.0.1:mysql_test> insert into `master_slave_demo` (`name`) value ('FFFFFF');
Query OK, 1 row affected
Time: 0.016s
mysql root@127.0.0.1:mysql_test> select * from `master_slave_demo`;
+----+--------+
| id | name   |
+----+--------+
| 1  | 张三   |
| 2  | 李四   |
| 3  | 王五   |
| 4  | 赵六   |
| 5  | AA     |
| 6  | BB     |
| 7  | CC     |
| 8  | DD     |
| 9  | EE     |
| 10 | FFFFFF |
+----+--------+
10 rows in set
Time: 0.011s
```
此时登录到slave查看是否有新的数据。
```mysql
mysql root@127.0.0.1:mysql_test> select * from master_slave_demo;
+----+--------+
| id | name   |
+----+--------+
| 1  | 张三   |
| 2  | 李四   |
| 3  | 王五   |
| 4  | 赵六   |
| 7  | CC     |
| 8  | DD     |
| 9  | EE     |
| 10 | FFFFFF |
+----+--------+
8 rows in set
Time: 0.013s
```
此时发现我们的数据已经自动同步过来了，到此我们的主从复制也配置完成了。
> 对于主从复制的数据库表不推荐使用该方式，在实际的过程中会遇到各种问题。这里只是为了文章演示采用的一种模式，后面针对MySQL的数据备份会单独总结一下如何正确的去操作。

## 同步模式

上面总结了同步的原理、同步的配置流程和同步的实际效果，下面针对主从复制的同步模式进行深一步的探索。



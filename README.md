# MySQL
----


## 基于日志点的主从复制实战 ##

1.mysql复制基础
1.1 异步复制，且主从之间存在时间延时，示意图如下：
![异步复制](https://i.imgur.com/6gZtMpd.jpg)  
1.2 复制基于binlog日志文件，其格式分为以下三种：
``` java

Statement: 在binlog文件中存储执行的sql语句[基于段]
row: binlog中存储event数据（记录每行变更的数据）[基于行]
mixed:
	 主从无差异时，使用statemt模式
	 主从有差异时，使用row模式

```

2.基于日志点的主从同步配置步骤：
>master端建立复制用户  
>备份数据库文件。在slave端恢复文件  
>slave端使用 `change master` 命令 配置主从同步命令  

``` java
//-- 建立同步用户 并授权
create user 'dba'@'192.168.80.%' identified by 'root'；
grant replication slave on *.* to 'dba'@'192.168.80.%';
//-- 建立同步的测试数据库
create database dba;
create table t(id int, name varchar(10), primary key(id));
insert into t values(1,'aa'),(2,'bb'),(3,'cc');
//-- 退出数据库 使用dump命令备份主数据库文件数据
mysqldump --single-transaction --master-data=2 --triggers --routines --all-databases -uroot -p  > all.sql
/**
 mysqldump --single-transaction  --master-data=2 命令
master-data=2 : log日志中注释change mater命令
查看general-log文件可知：
（1）FLUSH TABLES
（2）FLUSH TABLES WITH READ LOCK
（3）SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ
（4）START TRANSACTION /*!40100 WITH CONSISTENT SNAPSHOT */
（5）SHOW MASTER STATUS
（6）UNLOCK TABLES
上面六行可以保证不会锁表的情况下获得一致性快照、又可以精确地记下binlog位置!

*/

//考机器拷贝
scp all.sql root@192.168.80.101:/root/apps/mysql_slave;
//在81机器上恢复备份的数据库文件
mysql -uroot -p <all.sql

/**
cat all.sql
 CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.002799', MASTER_LOG_POS=1687;
*/

//从机器上主从复制配置
change master to master_host='192.168.80.100', msater_user='dba', master_password='123456', msater_log_file='mysql-bin.002799', master_log_pos='1687';

show slave status /G

//Slave_IO_Running: no
//Slave_IO_Runniong:no

//启动主从复制 相关功能
start slave  //需要关闭防火墙过滤规则
```

----

## mysql索引 ##

[索引数据结构博客参考：](https://blog.csdn.net/jacke121/article/details/78268602)
![B+树](https://i.imgur.com/ccOkrZL.jpg)










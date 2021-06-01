# Mysql运维知识

这里总结下mysql数据库常用的运维操作

## 登录MySQL

登录MySQL的命令是mysql， mysql 的使用语法如下：

mysql [-u username] [-h host] [-p[password]] [dbname]

username 与 password 分别是 MySQL 的用户名与密码，mysql的初始管理帐号是root，没有密码，注意：这个root用户不是Linux的系统用户。MySQL默认用户是root，由于初始没有密码，第一次进时只需键入mysql即可。

```
[root@test1 local]# mysql
Welcome to the MySQL monitor.　Commands end with ; or \g.
Your MySQL connection id is 1 to server version: 4.0.16-standard
Type 'help;' or '\h' for help. Type '\c' to clear the buffer.
mysql>
出现了“mysql>”提示符，恭喜你，安装成功！
增加了密码后的登录格式如下：
mysql -u root -p
Enter password: (输入密码)
其中-u后跟的是用户名，-p要求输入密码，回车后在输入密码处输入密码。
注意：这个mysql文件在/usr/bin目录下，与后面讲的启动文件/etc/init.d/mysql不是一个文件。
```

## MySQL的几个重要目录

MySQL安装完成后不象SQL Server默认安装在一个目录，它的数据库文件、配置文件和命令文件分别在不同的目录，了解这些目录非常重要，尤其对于Linux的初学者，因为Linux本身的目录结构就比较复杂，如果搞不清楚MySQL的安装目录那就无从谈起深入学习。

下面就介绍一下这几个目录。
* 数据库目录
/var/lib/mysql/
* 配置文件
/usr/share/mysql（mysql.server命令及配置文件）
* 相关命令
/usr/bin(mysqladmin mysqldump等命令)
* 启动脚本
/etc/rc.d/init.d/（启动脚本文件mysql的目录）

## 修改登录密码

MySQL默认没有密码，安装完毕增加密码的重要性是不言而喻的。

### 命令

usr/bin/mysqladmin -u root password ‘new-password’
格式：mysqladmin -u用户名 -p旧密码 password 新密码

### 例子

例1：给root加个密码123456。
键入以下命令 ：
[root@test1 local]# /usr/bin/mysqladmin -u root password 123456
注：因为开始时root没有密码，所以-p旧密码一项就可以省略了。

### 测试是否修改成功

* 不用密码登录

```
[root@test1 local]# mysql
ERROR 1045: Access denied for user: ‘root@localhost’ (Using password: NO)
显示错误，说明密码已经修改。
```

* 用修改后的密码登录

```
[root@test1 local]# mysql -u root -p
Enter password: (输入修改后的密码123456)
Welcome to the MySQL monitor.　Commands end with ; or \g.
Your MySQL connection id is 4 to server version: 4.0.16-standard
Type 'help;' or '\h' for help. Type '\c' to clear the buffer.
mysql>
```
成功！
这是通过mysqladmin命令修改口令，也可通过修改库来更改口令。

## 启动与停止

* 启动

MySQL安装完成后启动文件mysql在/etc/init.d目录下，在需要启动时运行下面命令即可。

```
[root@test1 init.d]# /etc/init.d/mysql start
```

* 停止

```
/usr/bin/mysqladmin -u root -p shutdown
```
* 自动启动

```
1）察看mysql是否在自动启动列表中
[root@test1 local]#　/sbin/chkconfig –list
2）把MySQL添加到你系统的启动服务组里面去
[root@test1 local]#　/sbin/chkconfig　– add　mysql
3）把MySQL从启动服务组里面删除。
[root@test1 local]#　/sbin/chkconfig　– del　mysql
```

## 更改MySQL目录

MySQL默认的数据文件存储目录为/var/lib/mysql。假如要把目录移到/home/data下需要进行下面几步

1.home目录下建立data目录

```
cd /home
mkdir data
```
2.把MySQL服务进程停掉

```
mysqladmin -u root -p shutdown
```

3.把/var/lib/mysql整个目录移到/home/data

```
mv /var/lib/mysql　/home/data/
```
这样就把MySQL的数据文件移动到了/home/data/mysql下

4.找到my.cnf配置文件

如果/etc/目录下没有my.cnf配置文件，请到/usr/share/mysql/下找到*.cnf文件，拷贝其中一个到/etc/并改名为my.cnf)中。命令如下：

```
[root@test1 mysql]# cp /usr/share/mysql/my-medium.cnf　/etc/my.cnf
```

5.编辑MySQL的配置文件/etc/my.cnf

为保证MySQL能够正常工作，需要指明mysql.sock文件的产生位置。 修改socket=/var/lib/mysql/mysql.sock一行中等号右边的值为：/home/mysql/mysql.sock 。操作如下：

```
vi　 my.cnf　　　 (用vi工具编辑my.cnf文件，找到下列数据修改之)
# The MySQL server
[mysqld]
port　　　= 3306
#socket　 = /var/lib/mysql/mysql.sock（原内容，为了更稳妥用“#”注释此行）
socket　 = /home/data/mysql/mysql.sock　　　（加上此行）
```

6.修改MySQL启动脚本/etc/rc.d/init.d/mysql

最后，需要修改MySQL启动脚本/etc/rc.d/init.d/mysql，把其中datadir=/var/lib/mysql一行中，等号右边的路径改成你现在的实际存放路径：home/data/mysql。

```
[root@test1 etc]# vi　/etc/rc.d/init.d/mysql
#datadir=/var/lib/mysql　　　　（注释此行）
datadir=/home/data/mysql　　 （加上此行）
```

7.重新启动MySQL服务

```
/etc/rc.d/init.d/mysql　start
```

或用reboot命令重启Linux，如果工作正常移动就成功了，否则对照前面的7步再检查一下。

## MySQL的常用操作

注意：MySQL中每个命令后都要以分号；结尾。

### 显示数据库

```
mysql> show databases;
+----------+
| Database |
+----------+
| mysql　　|
| test　　 |
+----------+
 rows in set (0.04 sec)
```

Mysql刚安装完有两个数据库：mysql和test。mysql库非常重要，它里面有MySQL的系统信息，我们改密码和新增用户，实际上就是用这个库中的相关表进行操作。

### 显示数据库中的表

```
mysql> use mysql; （打开库，对每个库进行操作就要打开此库，类似于foxpro ）
Database changed
	 
mysql> show tables;
+-----------------+
| Tables_in_mysql |
+-----------------+
| columns_priv　　|
| db　　　　　　　|
| func　　　　　　|
| host　　　　　　|
| tables_priv　　 |
| user　　　　　　|
+-----------------+
6 rows in set (0.01 sec)
```

### 显示数据表的结构

```
describe 表名;
```

### 显示表中的记录

```
select * from 表名;
例如：显示mysql库中user表中的纪录。所有能对MySQL用户操作的用户都在此表中。
Select * from user;
```

### 建库

```
create database 库名;
例如：创建一个名字位aaa的库
mysql> create databases aaa;
```
### 建表

```
use 库名；
create table 表名 (字段设定列表)；
例如：在刚创建的aaa库中建立表name,表中有id(序号，自动增长)，xm（姓名）,xb（性别）,csny（出身年月）四个字段
use aaa;
mysql> create table name (id int(3) auto_increment not null primary key, xm char(8),xb char(2),csny date);
可以用describe命令察看刚建立的表结构。
mysql> describe name;
	 
+-------+---------+------+-----+---------+----------------+
| Field | Type | Null | Key | Default | Extra　　　　　|
+-------+---------+------+-----+---------+----------------+
| id　　| int(3)　|　　　| PRI | NULL　　| auto_increment |
| xm　　| char(8) | YES　|　　 | NULL　　|　　　　　　　　|
| xb　　| char(2) | YES　|　　 | NULL　　|　　　　　　　　|
| csny　| date　　| YES　|　　 | NULL　　|　　　　　　　　|
+-------+---------+------+-----+---------+----------------+
```

### 增加记录

```
例如：增加几条相关纪录。
mysql> insert into name values(”,’张三’,’男’,’1971-10-01′);
mysql> insert into name values(”,’白云’,’女’,’1972-05-20′);
可用select命令来验证结果。
mysql> select * from name;
+—-+——+——+————+
| id | xm　 | xb　 | csny　　　 |
+—-+——+——+————+
|　1 | 张三 | 男　 | 1971-10-01 |
|　2 | 白云 | 女　 | 1972-05-20 |
+—-+——+——+————+
```
### 修改纪录

```
例如：将张三的出生年月改为1971-01-10
mysql> update name set csny=’1971-01-10′ where xm=’张三’;
```

### 删除纪录

```
例如：删除张三的纪录。
mysql> delete from name where xm=’张三’;
```

### 删库和删表

```
drop database 库名;
drop table 表名；
```

## 增加MySQL用户

格式：grant select on 数据库.* to 用户名@登录主机 identified by “密码”

1、增加一个用户user_1密码为123，让他可以在任何主机上登录，并对所有数据库有查询、插入、修改、删除的权限。首先用以root用户连入MySQL，然后键入以下命令：

```
mysql> grant select,insert,update,delete on *.* to user_1@”%” Identified by “123″;
```

例1增加的用户是十分危险的，如果知道了user_1的密码，那么他就可以在网上的任何一台电脑上登录你的MySQL数据库并对你的数据为所欲为了，解决办法见例2。


2、增加一个用户user_2密码为123,让此用户只可以在localhost上登录，并可以对数据库aaa进行查询、插入、修改、删除的操作（localhost指本地主机，即MySQL数据库所在的那台主机），这样用户即使用知道user_2的密码，他也无法从网上直接访问数据库，只能通过MYSQL主机来操作aaa库。

```
mysql>grant select,insert,update,delete on aaa.* to user_2@localhost identified by “123″;
用新增的用户如果登录不了MySQL，在登录时用如下命令：
mysql -u user_1 -p　-h 192.168.113.50　（-h后跟的是要登录主机的ip地址）
```

## mysql的数据导入导出

mysql数据库执行导入导出命令在可执行的mysql目录下执行

1. 从数据库导出数据库或表文件：

```
mysqldump -u用戶名 -p密码 -d 数据库名 表名 > 脚本名;
导出整个数据库结构和数据
mysqldump -h localhost -uroot -p123456 database > e:\dump.sql
导出单个数据表结构和数据
mysqldump -h localhost -uroot -p123456 database table > e:\dump.sql
导出整个数据库结构（不包含数据）
mysqldump -h localhost -uroot -p123456 -d database > e:\dump.sql
导出单个数据表结构（不包含数据）
mysqldump -h localhost -uroot -p123456 -d database table > e:\dump.sql
```

2. 导入数据库或表到数据库（数据库要先建好）

```
 方法1：mysql -h localhost -uroot -p123456 -d database table < e:\dump.sql
 方法2：
    1.进入MySQL：mysql -u root -p
    2.输入：use 目标数据库名
    3.导入文件：source  e:\dump.sql
```

## mysql中用户的权限分配

```
用户管理
mysql>use mysql;
查看
mysql> select host,user,password from user ;
创建
mysql> create user  zx_root   IDENTIFIED by 'xxxxx';   //identified by 会将纯文本密码加密作为散列值存储
修改
mysql>rename   user  feng  to   newuser；//mysql 5之后可以使用，之前需要使用update 更新user表
删除
mysql>drop user newuser;   //mysql5之前删除用户时必须先使用revoke 删除用户权限，然后删除用户，mysql5之后drop 命令可以删除用户的同时删除用户的相关权限
更改密码
mysql> set password for zx_root =password('xxxxxx');
mysql> update  mysql.user  set  password=password('xxxx')  where user='otheruser'
查看用户权限
mysql> show grants for zx_root;
赋予权限
mysql> grant select on dmc_db.*  to zx_root;
回收权限
mysql> revoke  select on dmc_db.*  from  zx_root;  //如果权限不存在会报错
 
上面的命令也可使用多个权限同时赋予和回收，权限之间使用逗号分隔
mysql> grant select，update，delete  ，insert  on dmc_db.*  to  zx_root;
如果想立即看到结果使用
flush  privileges ;
命令更新 
```

设置权限时必须给出一下信息

1. 要授予的权限
2. 被授予访问权限的数据库或表
3. 用户名

grant和revoke可以在几个层次上控制访问权限
1. 整个服务器，使用 grant ALL  和revoke  ALL
2. 整个数据库，使用on  database.*
3. 特点表，使用on  database.table
4. 特定的列
5. 特定的存储过程
 
user表中host列的值的意义

* %              匹配所有主机
* localhost    localhost不会被解析成IP地址，直接通过UNIXsocket连接
* 127.0.0.1      会通过TCP/IP协议连接，并且只能在本机访问；
* ::1                 ::1就是兼容支持ipv6的，表示同ipv4的127.0.0.1
 

``` 
grant 普通数据用户，查询、插入、更新、删除 数据库中所有表数据的权利。
grant select on testdb.* to common_user@’%’
grant insert on testdb.* to common_user@’%’
grant update on testdb.* to common_user@’%’
grant delete on testdb.* to common_user@’%’
或者，用一条 MySQL 命令来替代：
grant select, insert, update, delete on testdb.* to common_user@’%’
9>.grant 数据库开发人员，创建表、索引、视图、存储过程、函数。。。等权限。
grant 创建、修改、删除 MySQL 数据表结构权限。
grant create on testdb.* to developer@’192.168.0.%’;
grant alter on testdb.* to developer@’192.168.0.%’;
grant drop on testdb.* to developer@’192.168.0.%’;
grant 操作 MySQL 外键权限。
grant references on testdb.* to developer@’192.168.0.%’;
grant 操作 MySQL 临时表权限。
grant create temporary tables on testdb.* to developer@’192.168.0.%’;
grant 操作 MySQL 索引权限。
grant index on testdb.* to developer@’192.168.0.%’;
grant 操作 MySQL 视图、查看视图源代码 权限。
grant create view on testdb.* to developer@’192.168.0.%’;
grant show view on testdb.* to developer@’192.168.0.%’;
grant 操作 MySQL 存储过程、函数 权限。
grant create routine on testdb.* to developer@’192.168.0.%’; -- now, can show procedure status
grant alter routine on testdb.* to developer@’192.168.0.%’; -- now, you can drop a procedure
grant execute on testdb.* to developer@’192.168.0.%’;
10>.grant 普通 DBA 管理某个 MySQL 数据库的权限。
grant all privileges on testdb to dba@’localhost’
其中，关键字 “privileges” 可以省略。
11>.grant 高级 DBA 管理 MySQL 中所有数据库的权限。
grant all on *.* to dba@’localhost’
12>.MySQL grant 权限，分别可以作用在多个层次上。
1. grant 作用在整个 MySQL 服务器上：
grant select on *.* to dba@localhost; -- dba 可以查询 MySQL 中所有数据库中的表。
grant all on *.* to dba@localhost; -- dba 可以管理 MySQL 中的所有数据库
2. grant 作用在单个数据库上：
grant select on testdb.* to dba@localhost; -- dba 可以查询 testdb 中的表。
3. grant 作用在单个数据表上：
grant select, insert, update, delete on testdb.orders to dba@localhost;
4. grant 作用在表中的列上：
grant select(id, se, rank) on testdb.apache_log to dba@localhost;
5. grant 作用在存储过程、函数上：
grant execute on procedure testdb.pr_add to ’dba’@’localhost’
grant execute on function testdb.fn_add to ’dba’@’localhost’
注意：修改完权限以后 一定要刷新服务，或者重启服务，刷新服务用：FLUSH PRIVILEGES。
```

## mysql主从复制和读写分离的搭建

Mysql的主从复制的搭建，大家可以采用在linux，windows，docker，dockr-compose来搭建mysql

本次采用方式为docker-composer来搭建多个mysql服务端

目录结构如下
 
```
docker-compose.yml文件内容
version: '2'
services:
  mysql-master:
    build:
      context: ./
      dockerfile: master/Dockerfile
    environment:
      - "MYSQL_ROOT_PASSWORD=root"
      - "MYSQL_DATABASE=replicas_db"
    links:
      - mysql-slave
    ports:
      - "33065:3306"
    restart: always
    hostname: mysql-master
  mysql-slave:
    build:
      context: ./
      dockerfile: slave/Dockerfile
    environment:
      - "MYSQL_ROOT_PASSWORD=root"
      - "MYSQL_DATABASE=replicas_db"
    ports:
      - "33066:3306"
    restart: always
hostname: mysql-slave
```

Master的dockerfile和my.cnf

```
Dockerfile内容
FROM mysql:5.7
MAINTAINER harrison
ADD ./master/my.cnf /etc/mysql/my.cnf
```

```
my.cnf内容
[mysqld]
## 设置server_id，一般设置为IP，注意要唯一
server_id=100
## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
binlog-ignore-db=mysql
## 开启二进制日志功能，可以随便取，最好有含义（关键就是这里了）
log-bin=replicas-mysql-bin
## 为每个session分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=1M
## 主从复制的格式（mixed,statement,row，默认格式是statement）
binlog_format=mixed
## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
```

Slave的dockerfile和my.cnf

```
Dockerfile内容
FROM mysql:5.7
MAINTAINER harrison
ADD ./slave/my.cnf /etc/mysql/my.cnf
```

```
my.cnf内容
[mysqld]
## 设置server_id，一般设置为IP，注意要唯一
server_id=101
## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
binlog-ignore-db=mysql
## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
log-bin=replicas-mysql-slave1-bin
## 为每个session 分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=1M
## 主从复制的格式（mixed,statement,row，默认格式是statement）
binlog_format=mixed
## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
## relay_log配置中继日志
relay_log=replicas-mysql-relay-bin
## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1
## 防止改变数据(除了特殊的线程)
read_only=1
```

进入 docker 目录，运行 docker-compose 启动命令。

```
$ docker-compose up -d
检查从库的起始状态
$ show master status;
```

从数据库处于 未同步复制状态。

检查主库的状态

```
$ show master status;
```

记录 主数据库 binary-log的文件名称和数据同步起始位置。

```
File: replicas-mysql-bin.000003
Position: 154
```

从库配置主库信息

在 从数据库上运行主数据库的相关配置sql进行主从关联

```
CHANGE MASTER TO
    MASTER_HOST='mysql-master',
    MASTER_USER='root',
    MASTER_PASSWORD='root',
    MASTER_LOG_FILE='replicas-mysql-bin.000003',
    MASTER_LOG_POS=154;
```

重新启动 slave 服务

```
$ stop slave
$ start slave
```

进一步检查从数据库的状态信息，两者已经进行数据同步关联。

## master修改密码如何同步slave

```
所以，更新密码后，只需要：
change master to master_user='replication user', master_password='new passwd';
```

## mysql慢查询的原因

* 没有索引或者没有用到索引(这是查询慢最常见问题，是程序设计的缺陷) 
* I/O吞吐量小，形成了瓶颈效应。 
* 没有创建计算列导致查询不优化。 
* 内存不足 
* 网络速度慢 
* 查询出的数据量过大(可以采用多次查询，其他的方法降低数据量) 
* 锁或者死锁(这也是查询慢最常见的问题，是程序设计的缺陷) 
* sp_lock,sp_who,活动的用户查看,原因是读写竞争资源。 
* 返回了不必要的行和列 
* 查询语句不好，没有优化

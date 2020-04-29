---
layout: post
date:   2020-04-28
categories: [mysql, docker]
title: "基于docker部署MySQL读写分离库"
---

本文记录：如何通过docker搭建本地的MySQL读写分离库。

### 第一步，创建mysql的工作目录

分别为主库和从库创建配置目录及工作目录等。

	主库：
	mysql
	 - master
	   - conf
	   - data
	   - logs
	从库：
	mysql
	 - slave
	   - conf
	   - data
	   - logs
		
### 第二步，添加配置文件

1 在主库目录master/conf目录下，创建文件```my.cnf```，并加入以下内容

```
[mysqld]
log-bin=mysql-bin
server-id=1
```

2 在从库目录slave/conf目录下，创建文件```my.cnf```，并加入以下内容

```
[mysqld]
log-bin=mysql-bin
server-id=2
```

注意：主库和从库的```server-id```要不一样。

### 第三步，创建主库

运行docker命令：

```
docker run --restart=always -p 3307:3306 -v /mysql/master/conf:/etc/mysql -v /mysql/master/logs:/var/log/mysql -v /mysql/master/data:/var/lib/mysql --name mysql-master -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7 
```

这样，主库就创建好了，端口是：3307，ROOT密码是：123456。

注意：-v 后面的参数是本地目录映射到docker目录，冒号前面是本地目录，要与你在第一步中创建的主库的目录相一致。

### 第四步，创建从库

运行docker命令：

```
docker run --restart=always -p 3308:3306 -v /mysql/slave/conf:/etc/mysql -v /mysql/slave/logs:/var/log/mysql -v /mysql/slave/data:/var/lib/mysql --name mysql-slave -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

这样，从库就创建好了，端口是：3308，ROOT密码是：123456。

### 第五步，主库设置

1 创建并授权从库同步用户

通过客户端或命令行连接主库，然后运行命令：

```
GRANT REPLICATION SLAVE ON *.* to 'user'@'%' identified by 'mysql';
```

这样，就在主库中创建了一个密码为“mysql”的“user”用户。

2 查询主库的binlog信息，并记录。

运行命令：

```
show master status;
```

会得到如下类似的信息：

```
'mysql-bin.000003', '429', '', '', ''
```

### 第六步，从库设置

通过客户端或命令行连接从库，然后运行命令：

```
change master to master_host='127.0.0.1', master_user='user', master_log_file='mysql-bin.000003', master_log_pos=429, master_port=3307, master_password='mysql';
``` 

运行成功后，主从库就配置好了。

### 后记

为了安全起见，从库中最好创建一个只读的用户，避免写入，进而避免主从同步的不必要麻烦。
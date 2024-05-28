---
layout:     post     # 使用的布局（不需要改）
title:      基于docker快速搭建mysql主从集群
subtitle:   mysql主从集群搭建，docker使用
date:       2024-05-28 
author:     Tong # 作者
header-img: img/post-bg-2015.jpg 
catalog: true 
tags: #标签
    - MySQL
    - Docker
typora-copy-images-to: ../assets/images
typora-root-url: ../../blog
---

### 1、拉取MySQL的docker镜像

````shell
docker pull mysql:8.0.32
````



### 2、创建对应的mysql配置文件，不然挂载目录后mysql服务无法启动

在对应目录创建my.cnf文件

master配置文件：/root/mysql/master/config/conf.d/my.cnf

````text
[mysqld]
## 同一局域网内注意要唯一
server-id=100  
## 开启二进制日志功能，可以随便取（关键）
log-bin=mysql-bin
````



slave配置文件：/root/mysql/slave-1/config/conf.d/my.cnf

````text
[mysqld]  
## 设置server_id,注意要唯一  
server-id=101 
## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用  
log-bin=mysql-slave-bin 
## relay_log配置中继日志  
relay_log=edu-mysql-relay-bin
````



### 3、创建master、slave容器

创建master容器

````shell
docker run -p 13306:3306 -v /root/mysql/master/config:/etc/mysql --name mysql-master -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0.32
````

创建slave容器

````shell
docker run -p 23306:3306 -v /root/mysql/slave-1/config:/etc/mysql --name mysql-slave -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0.32
````



创建后

![image-20240528223044377](/assets/images/image-20240528223044377.png)



扩展：

可以通过下面的命令进入容器内部，容器内部也是一个linux服务

````shell
docker exec -it mysql-master /bin/bash
````



### 4、在master节点创建用于同步binlog文件的用户，并授予权限

````mysql
# 创建slave用户
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';

# 授予复制账号 REPLICATION SLAVE 权限，复制才能真正地工作。
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
````



````mysql
# 查看所有用户
SELECT User, Host FROM mysql.user;
````

<img src="/assets/images/image-20240528225043313.png" alt="image-20240528225043313" style="zoom:100%;" align="left" />



### 5、开启主从集群

进入master容器

````shell
docker exec -it mysql-master /bin/bash
````

在 Master 进入 mysql，执行 `show master status;` 查询主节点binlog文件和同步文件偏移位置

````mysql
show master status;
````

![image-20240528225925282](/assets/images/image-20240528225925282.png)



进入slave容器

````shell
docker exec -it mysql-slave /bin/bash
````

在 Slave 中进入 mysql，执行

````mysql
CHANGE MASTER TO master_host = '172.17.0.2',
master_user = 'slave',
master_password = '123456',
master_port = 3306,
master_log_file = 'mysql-bin.000004',
master_log_pos = 440,
master_connect_retry = 30;
````

Master_host可以通过下面的命令获取

````shell
docker inspect --format='{{.NetworkSettings.IPAddress}}' mysql-master
````

![image-20240528230356930](/assets/images/image-20240528230356930.png)

```text
master_host ：Master 的地址，指的是容器的独立 ip
master_port：Master 的端口号，指的是容器的端口号
master_user：用于数据同步的用户
master_password：用于同步的用户的密码
master_log_file：指定 Slave 从哪个日志文件开始复制数据，即上文中提到的 File 字段的值
master_log_pos：从哪个 Position 开始读，即上文中提到的 Position 字段的值
master_connect_retry：如果连接失败，重试的时间间隔，单位是秒，默认是 60 秒
```



制定完主节点后执行

````mysql
show slave status\G;
````

![image-20240528231406551](/assets/images/image-20240528231406551.png)



开启从节点

````mysql
start slave;
````

再次查询发现报错

![image-20240528233816602](/assets/images/image-20240528233816602.png)

报错主要有下面几种情况：

- 网络不通
  检查 ip, 端口
- 密码不对
  检查是否创建用于同步的用户和用户密码是否正确
- pos 不对
  检查 Master 的 Position

使用 MySQL 8.0 版本及以上，默认的身份验证插件为 `caching_sha2_password` 时。这种插件要求客户端连接时使用加密连接（例如 TLS/SSL）

更改用户的身份验证插件：

```mysql
ALTER USER 'slave'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
```

修改后Position位置会变更



最后根据提示一步步总是成功了

![image-20240528234312648](/assets/images/image-20240528234312648.png)



### 6、测试

master上执行

````mysql
# 创建db01
create database db01;
# 使用db01
use db01;
# 创建teacher表
create table teacher
(
    id   int auto_increment
        primary key,
    name varchar(20) null comment '名称',
    age  int         null
);
# 插入数据
insert into teacher(id, name, age) VALUE(null, '张三', 18);
# 查询数据
select * from teacher;
````

![image-20240528235317082](/assets/images/image-20240528235317082.png)



slave上执行

````mysql
# 查询teacher表数据
select * from db01.teacher;
````

![image-20240528235424087](/assets/images/image-20240528235424087.png)



完美


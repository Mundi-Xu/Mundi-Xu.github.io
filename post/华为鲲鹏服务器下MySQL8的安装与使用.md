---
title: 华为鲲鹏服务器下MySQL8的安装与远程连接配置
date: 2021-03-10 23:05:21
tags: [arm64, Ubuntu, MySQL]
toc: true
excerpt: 在华为鲲鹏学生机Ubuntu 20.04.2 LTS (GNU/Linux 4.15.0-136-generic aarch64)上配置安装MySQL8
---

# Docker安装

> Docker Version: 20.10.5
> Site: https://hub.docker.com/r/mysql/mysql-server
> Ubuntu 20.04.2 LTS (GNU/Linux 4.15.0-136-generic aarch64)

Docker的安装就跳过了，直接快进到mysql-server的安装，好消息是mysql官方Docker已经适配了arm64架构，所以直接参照官方文档安装就可以了。

```shell
docker pull mysql/mysql-server # 默认选择latest
docker run --name=mysql1 -d mysql/mysql-server
docker ps # 等待初始化完成，starting变为healthy
docker logs mysql1 # 监控容器输出
docker logs mysql1 2>&1 | grep GENERATED # 初始化完成后查看密码
GENERATED ROOT PASSWORD: Axegh3kAJyDLaRuBemecis&EShOs # 生成的随机密码
docker exec -it mysql1 mysql -uroot -p # 输入上面生成的随机密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password'; # 修改root密码
```

# 包管理器安装

> mysql  Ver 8.0.23-0ubuntu0.20.04.1 for Linux on aarch64 ((Ubuntu))

更新源，安装依赖

```shell
sudo apt-get install mysql-server
sudo apt install mysql-client
sudo apt install libmysqlclient-dev
sudo mysql # 查看是否成功
```

# MySQL配置

mysql安装完成后默认是没有密码的,需要先修改密码

```mysql
alter user 'root'@'localhost' identified by "password";
```

远程连接需要单独配置

```shell
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
# 注释掉bind-address= 127.0.0.1 或改为0.0.0.0
sudo service mysql restart # 重启MySQL服务
```

添加远程连接的账号

```mysql
create user root@'%' identified by 'password';
grant all privileges on *.* to root@'%';
flush privileges;
```

最后在华为ECS的安全组配置中开启3306端口就可以远程连接了。
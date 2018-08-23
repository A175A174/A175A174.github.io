---
title: Linux下安装MySql8
key: 201808192
tags: Linux MySql
---

# 一、安装

1.安装yum源，<https://dev.mysql.com/downloads/repo/yum/>，下载安装即可，也可以用以下命令下载

```shell
# 下载rpm安装包
curl https://repo.mysql.com/mysql80-community-release-el7-1.noarch.rpm -o /tmp/mysql80-community-release-el7-1.noarch.rpm

# 安装yum源
rpm -ivh mysql80-community-release-el7-1.noarch.rpm
```

2.安装mysql

```shell
# 安装mysql
yum install mysql-community-server -y

# 开启服务
systemctl start mysqld.service

# 开机自启动
systemctl enable mysqld.service

# 查看端口
netstat -ln | grep 3306
```

<!--more-->

# 二、配置

## 1.初始化

```shell
# 从日志文件中获取root账户密码
grep "password" /var/log/mysqld.log
```

![img](/myres/20180819/20180821000000005.png)

直接用该密码登陆会要求改密码，最好先进行向导操作

```shell
# 安全向导
mysql_secure_installation

# 输入root密码
Enter password for user root:
# 设置新密码，密码太简单会提示错误：Your password does not satisfy the current policy requirements
New password
# 是否确认修改root密码
Change the password for root
# 是否删除匿名用户
Remove anonymous users
# 是否禁止root远程登录
Disallow root login remotely
# 是否删除test数据库
Remove test database and access to it
# 是否现在刷新权限
Reload privilege tables now
```

## 2.设置简单密码

[官方](https://dev.mysql.com/doc/refman/8.0/en/validate-password.html)在新版本中对密码设置要求比较严格

```shell
# 查看密码策略
SHOW VARIABLES LIKE 'validate_password%';

# 修改密码策略，修改密码检查强度和密码长度就可以
set global validate_password.policy=0;
set global validate_password.length=4;

# 修改当前用户密码
ALTER USER USER() IDENTIFIED BY '123456';
# 修改指定用户密码
ALTER user 'root'@'localhost' IDENTIFIED BY '123456';
```

忘记root密码情况下设置密码

```shell
# 编辑配置文件
vi /etc/my.cnf
# 文件最后添加免密登陆
skip-grant-tables
# 重启mysql服务
systemctl restart mysqld.service
# 登陆mysql，不用密码
mysql -uroot

# 清空指定账户密码
use mysql;
update user set authentication_string='' where user='root';

# 退出mysql，删除免密登陆配置，重启mysql服务，用空密码登陆，然后进行修改密码操作
```

## 3.添加远程访问权限

首先**开放本机3306端口**

```shell
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
# 保存配置
service iptables save
```

修改已有用户

```shell
# 选择mysql数据库
use mysql;

# 修改指定用户登录位置，%为任意地址，可配置指定IP
update user set host='%' where user='root';

# 修改登陆密码和加密规则，8版本默认的认证插件为Caching_sha2_password，原来是mysql_native_password，不修改Navicat等工具连接会认证失败
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';

# 刷新权限
flush privileges;
```

创建新用户

```shell
# 创建用户(user1为用户名，%为登陆地址任意ip,也可指定，123456为登录密码)
CREATE USER 'user1'@'%' IDENTIFIED WITH mysql_native_password BY '123456';

# 默认创建的用户是无权限，只能登录而已，（all：所有权限，有select,update等等权限，后面的*.*：指定数据库.指定表，这里指所有，to后面是用户）
grant all on *.* to 'user1'@'%';

# 刷新权限
flush privileges;
```

查看登陆权限

```shell
use mysql;
select host,user from user;
```

## 4.配置文件（/etc/my.cnf）

```conf
[mysqld]
# 关闭密码验证，可设置简单密码
validate_password=off

# 存储引擎
default-storage-engine=INNODB

# 编码
character-set-server=utf8

# 排序规则
collation-server=utf8_general_ci

# 免密登陆
skip-grant-tables
```

# 三、命令

```shell
# 查看版本
select version();

# 设置编码
SET NAMES utf8;

# 查看编码
show variables like '%character%';

# 查看排序规则
show variables like 'collation%';

# 查看储存引擎，Support列，YES表示支持，DEFAULT表示默认，NO表示不支持
show engines;
show variables like '%storage_engine%';

# 设置储存引擎
SET default_storage_engine=InnoDB;

# 查看日志文件位置
select @@log_error;
```

# [官方文档](https://dev.mysql.com/doc/refman/8.0/en/)

---
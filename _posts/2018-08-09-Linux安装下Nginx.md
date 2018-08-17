---
title: Linux安装下Nginx
key: 201808091
tags: Nginx Linux
---

# 一、下载

[点击下载](http://nginx.org/en/download.html)

# 二、安装

安装编译所需要的环境

```shell
yum install gcc zlib zlib-devel pcre-devel openssl openssl-devel -y
```

上传下载的安装包，然后解压

```shell
tar -zxvf nginx-1.14.0.tar.gz
```

进入到解压的目录，设置nginx的安装目录，然后编译安装

<!--more-->

```shell
./configure --prefix=/usr/local/nginx
make
make install
```

![tu](/myres/20180809/20180808234050.png)

进入到安装目录测试是否安装成功

```shell
./nginx -t
```

![tu](/myres/20180809/20180808233725.png)

# 三、命令

![tu](/myres/20180809/20180815164605.png)

```conf
./nginx  #打开 nginx
nginx -s reload|reopen|stop|quit  #重新加载配置|重启|停止|退出 nginx
nginx -t   #测试配置是否有语法错误

nginx [-?hvVtq] [-s signal] [-c filename] [-p prefix] [-g directives]

-?,-h           : 打开帮助信息
-v              : 显示版本信息并退出
-V              : 显示版本和配置选项信息，然后退出
-t              : 检测配置文件是否有语法错误，然后退出
-q              : 在检测配置文件期间屏蔽非错误信息
-s signal       : 给一个 nginx 主进程发送信号：stop（停止）, quit（退出）, reopen（重启）, reload（重新加载配置文件）
-p prefix       : 设置前缀路径（默认是：/usr/local/nginx/）
-c filename     : 设置配置文件（默认是：/usr/local/nginx/conf/nginx.conf）
-g directives   : 设置配置文件外的全局指令
```

---
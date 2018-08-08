---
title: Linux安装下Nginx
key: 20180809
tags: TeXt
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

---
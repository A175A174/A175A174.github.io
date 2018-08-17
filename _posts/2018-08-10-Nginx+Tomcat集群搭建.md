---
title: Nginx+Tomcat集群搭建
key: 20180810
tags: Nginx Tomcat
---

# 作用

提高服务性能，并发能力，高可用性，提供项目架构横向扩展能力

# 一、配置

这里以单机环境做测试，首先把Tomcat解压两份。

首先修改Tomcat配置文件

依次进入两个Tomcat的conf目录，编辑server.xml文件添加UTF-8编码

```xml
<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" URIEncoding="UTF-8"/>
```

添加环境变量
<!--more-->

```shell
vim /etc/profile

# 在最后添加如下变量

# 第一个Tomcat
TOMCAT_HOME=/usr/local/apache-tomcat-9.0.10-1
CATALINA_BASE=/usr/local/apache-tomcat-9.0.10-1
CATALINA_HOME=/usr/local/apache-tomcat-9.0.10-1
export CATALINA_BASE CATALINA_HOME TOMCAT_HOME
# 第二个Tomcat
TOMCAT_2_HOME=/usr/local/apache-tomcat-9.0.10-2
CATALINA_2_BASE=/usr/local/apache-tomcat-9.0.10-2
CATALINA_2_HOME=/usr/local/apache-tomcat-9.0.10-2
export CATALINA_2_BASE CATALINA_2_HOME TOMCAT_2_HOME

# 环境生效
source /etc/profile
```

修改其中一个Tomcat的环境变量，另一个可不修改，进入bin目录修改catalina.sh文件

```conf
# OS specific support.  $var _must_ be set to either true or false.
export CATALINA_BASE=$CATALINA_2_BASE
export CATALINA_HOME=$CATALINA_2_HOME
```

接着修改Tomcat服务端口，端口号与其他Tomcat不一样且没有被占用即可，进入conf目录修改service.xml

```xml
# http访问端口
<Connector port="8090" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" URIEncoding="UTF-8"/>
# AJP端口
<Connector port="8019" protocol="AJP/1.3" redirectPort="8443" />
# Shutdown远程停止服务端口
<Server port="8015" shutdown="SHUTDOWN">
```

再修改Nginx配置文件

## 几种负载方式

```conf
# 轮询，不考虑服务器处理能力
upstream 127.0.0.1:80{
    server 127.0.0.1:8080;
    server 127.0.0.1:8090;
    # 其他非backup机器dwon或者忙的时候请求backup机器
    server 127.0.0.1:8090 backup;
}
# 权重，考虑每台服务器处理能力，weight默认为1
upstream 127.0.0.1:80{
    server 127.0.0.1:8080 weight=10;
    server 127.0.0.1:8090 weight=20;
}
# ip hash，实现同一用户访问同一服务器，不一定平均
upstream 127.0.0.1:80{
    ip_hash;
    server 127.0.0.1:8080;
    server 127.0.0.1:8090;
}
# url hash(第三方)，需安装插件，实现同一服务访问同一个服务器，不一定平均，请求频繁的url会请求到同一服务器
upstream 127.0.0.1:80{
    server 127.0.0.1:8080;
    server 127.0.0.1:8090;
    hash $request_uri;
}
# fair(第三方)，需安装插件，按服务器相应时间分配，响应时间短的有限分配
upstream 127.0.0.1:80{
    server 127.0.0.1:8080;
    server 127.0.0.1:8090;
    fair;
}
```

进入到Nginx的配置目录，打开conf下的nginx.conf文件

在http节点最后添加下面一行配置，然后就可以把配置写在conf文件夹下的upconf文件里，这样易于管理

```config
include host/*.conf;
```

然后在conf/host目录下新建xxx.conf配置文件，添加如下内容

```conf
upstream mall.com{
    server 127.0.0.1:8080;
    server 127.0.0.1:8090;
}
server {
    #监听端口
    listen 80;
    #开启目录浏览
    autoindex on;
    #访问地址，要能访问得到主机，可以做host映射
    server_name 192.168.8.10;
    location / {
        proxy_pass http://mall.com;
    }
}
```

# 二、启动测试

进入到Tomcat的bin目录，执行startup.sh文件，为了区分替换其中一个的logo，路径为webapps/ROOT/tomcat.png

进入到Nginx的sbin目录，执行nginx文件

![tu](/myres/20180810/20180810000010.png)

![tu](/myres/20180810/20180809235251.png)
![tu](/myres/20180810/20180809235303.png)
![tu](/myres/20180810/20180809235351.png)

可以看到访问同一地址可以随机访问Tomcat服务器

---
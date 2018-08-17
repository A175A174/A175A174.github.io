---
title: Spring Session的使用
key: 20180817
tags: Spring Session
---

# 一、简介

方便的解决集群环境下Session共享问题，对业务代码没有入侵

# 二、使用

在web工程中pom.xml引入jar包，这里使用Redis环境

```xml
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
    <version>2.0.5.RELEASE</version>
</dependency>

<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
    <type>jar</type>
    <scope>compile</scope>
</dependency>
```

配置web.xml文件

<!--more-->

```xml
<!--DelegatingFilterProxy类将通过springSessionRepositoryFilter这个名称去查找Spring容器中的Bean并将其转换为过滤器，对于调用DelegatingFilterProxy的每个请求，将调用springSessionRepositoryFilter这个过滤器-->
<filter>
    <filter-name>springSessionRepositoryFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSessionRepositoryFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

## 1.单个Redis下（Redis Standalone）

```xml
<!--springSessionRepositoryFilter在这里被加入容器-->
<bean class="org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration">
    <!--存入Redis数据的有效时间，单位秒-->
    <property name="maxInactiveIntervalInSeconds" value="60" />
</bean>

<!--定制Cookie-->
<bean id="defaultCookieSerializer" class="org.springframework.session.web.http.DefaultCookieSerializer">
    <property name="cookieName" value="koioik"/>
    <!--域不能随意设置，要符合自己的访问域名-->
    <property name="domainName" value="localhost"/>
    <property name="useHttpOnlyCookie" value="true"/>
    <property name="cookiePath" value="/"/>
    <property name="cookieMaxAge" value="60"/>
</bean>

<!-- Redis Standalone 单节点配置 -->
<bean id="redisStandaloneConfiguration" class="org.springframework.data.redis.connection.RedisStandaloneConfiguration">
    <property name="hostName" value="192.168.8.10"/>
    <property name="port" value="6379"/>
    <property name="database" value="0"/>
    <property name="password">
        <bean class="org.springframework.data.redis.connection.RedisPassword">
            <constructor-arg index="0" value=""/>
        </bean>
    </property>
</bean>

<!-- Redis 连接配置 -->
<bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
    <constructor-arg name="standaloneConfig" ref="redisStandaloneConfiguration"/>
</bean>
```

测试请求

```java
@RequestMapping(value = "/login.do", method = RequestMethod.GET)
public String login(HttpSession session){
    session.setAttribute("kekeke","hahaha");
    String kekeke = (String) session.getAttribute("kekeke");
    return kekeke;
}
```

![img](/myres/20180817/20180816003944.png)

可以看到原本存入Session的值被存入了Redis

## 2.多个Redis下（Redis Cluster）

```xml
<!-- Redis Cluster 多节点配置 -->
<bean id="clusterConfiguration" class="org.springframework.data.redis.connection.RedisClusterConfiguration">
    <constructor-arg>
        <list>
            <value>192.168.8.10:6379</value>
            <value>192.168.8.10:6380</value>
            <!--<value>192.168.8.10:6381</value>-->
            <!--<value>192.168.8.10:6382</value>-->
        </list>
    </constructor-arg>
</bean>

<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <!--连接耗尽时是否阻塞, false报异常,ture阻塞直到超时, 默认true-->
    <property name="blockWhenExhausted" value="true"/>
    <!--是否启用pool的jmx管理功能, 默认true-->
    <property name="jmxEnabled" value="true"/>
    <property name="jmxNamePrefix" value="pool"/>
    <!--jedis调用returnObject方法时，是否进行有效检查-->
    <property name="testOnReturn" value="true"/>
    <!--是否启用后进先出, 默认true-->
    <property name="lifo" value="true"/>
    <!--最大空闲连接数, 默认8个-->
    <property name="maxIdle" value="20" />
    <!--最小空闲连接数, 默认0-->
    <property name="minIdle" value="0" />
    <!--最大连接数, 默认8个-->
    <property name="maxTotal" value="20" />
    <!--获取连接时的最大等待毫秒数(如果设置为阻塞时BlockWhenExhausted),如果超时就抛异常, 小于零:阻塞不确定的时间,  默认-1-->
    <property name="maxWaitMillis" value="-1"/>
    <!--逐出连接的最小空闲时间 默认1800000毫秒(30分钟)-->
    <property name="minEvictableIdleTimeMillis" value="1800000"/>
    <!--每次逐出检查时 逐出的最大数目 如果为负数就是 : 1/abs(n), 默认3-->
    <property name="numTestsPerEvictionRun" value="3"/>
    <!--对象空闲多久后逐出, 当空闲时间>该值 且 空闲连接>最大空闲数 时直接逐出,不再根据MinEvictableIdleTimeMillis判断  (默认逐出策略)-->
    <property name="softMinEvictableIdleTimeMillis" value="1800000"/>
    <!--在获取连接的时候检查有效性, 默认false-->
    <property name="testOnBorrow" value="true" />
    <!--在空闲时检查有效性, 默认false-->
    <property name="testWhileIdle" value="false"/>
    <!--逐出扫描的时间间隔(毫秒) 如果为负数,则不运行逐出线程, 默认-1-->
    <property name="timeBetweenEvictionRunsMillis" value="-1"/>
</bean>

<!-- Redis 连接配置 -->
<bean class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
    <constructor-arg name="poolConfig" ref="jedisPoolConfig"/>
    <constructor-arg name="clusterConfig" ref="clusterConfiguration"/>
</bean>
```

修改测试代码

```java
@RequestMapping(value = "/login.do", method = RequestMethod.GET)
public String login(HttpSession session){
    for (int i = 0; i < 100 ; i++) {
        session.setAttribute("kekeke"+i,"hahaha"+i);
    }
    String kekeke = (String) session.getAttribute("kekeke1");
    return kekeke;
}
```

![img](/myres/20180817/20180816035732.png)

## PS：在搭建集群测试环境的时候要用本机IP地址，不要用127.0.0.1。CLUSTER NODES查出来的IP要与XML配置代码上的IP对应，不然启动项目会报 Could not get a resource from the pool 错误。

# [附上Demo](https://github.com/A175A174/Demo/tree/master/springSessionDemo)

---
---
title: SpringMVC拦截器
key: 20180815
tags: Spring
---

# 一、工程构建

1.引入SpringMVC需要的jar包

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.0.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.0.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>5.0.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.0.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.0.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.2</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>5.0.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-expression</artifactId>
    <version>5.0.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```

<!--more-->

2.配置web.xml文件

```xml
<!--静态资源的处理,设置必须在SpringMVC的Dispatcher前面-->
<servlet-mapping>
    <servlet-name>default</servlet-name>
    <url-pattern>*.jpg</url-pattern>
    <url-pattern>*.js</url-pattern>
    <url-pattern>*.css</url-pattern>
    <url-pattern>*.png</url-pattern>
    <url-pattern>*.gif</url-pattern>
    <url-pattern>*.json</url-pattern>
    <url-pattern>*.html</url-pattern>
    <url-pattern>*.htm</url-pattern>
    <url-pattern>*.swf</url-pattern>
</servlet-mapping>

<!--加载SpringMVC-->
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>

        <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <!-- >=0容器会初始化此servlet，否则使用时才初始化 -->
    <load-on-startup>1</load-on-startup>
    <async-supported>true</async-supported>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <!--拦截路径，/为拦截所有，*为通配符，不能设置成*，启动会报错-->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

3.配置SpringMVC配置文件

```xml
<!--注解驱动-->
<mvc:annotation-driven/>

<!--包扫描-->
<context:component-scan base-package="controller" annotation-config="true"/>

<!--过滤静态资源-->
<mvc:default-servlet-handler/>

<!--拦截器-->
<mvc:interceptors>
    <!-- 定义在这里的，所有的都会拦截-->
    <mvc:interceptor>
        <!--manage/a.do -> /manage/*-->
        <!--manage/product/save.do -> /manage/**-->
        <mvc:mapping path="/**"/>
        <!--<mvc:exclude-mapping path="/manage/user/login.do"/>-->
        <bean class="interceptor.AuthorityInterceptor" />
    </mvc:interceptor>
</mvc:interceptors>
```

# 二、创建拦截器类

只有一个要求，实现HandlerInterceptor接口即可，然后在SpringMVC配置文件中添加下一，实现后要重写三个方法
方法名称|说明
:---:|:--:
preHandle|在请求处理之前进行调用，若该请求存在多个Interceptor ，依据声明顺序依次执行，返回为false表示请求结束，后续的Interceptor和Controller都不会执行；返回true会继续调用下一个 Interceptor的preHandle方法，如果已经是最后一个Interceptor，就会请求Controller中的方法
postHandle|preHandle方法返回true时才会执行。postHandle方法在Controller中的方法调用之后执行，在视图返回渲染之前被调用，所以可以在这个方法中对Controller处理之后的ModelAndView对象进行操作。postHandle方法被调用的方向跟preHandle是相反的，先声明的 Interceptor的postHandle方法反而会后执行
afterCompletion|preHandle方法返回true时才会执行。该方法在整个请求结束之后，也就是在渲染了视图之后执行

```java
@RestController
public class UserController {
    @RequestMapping(value = "/login.do", method = RequestMethod.GET)
    public String login(String name){
        return name;
    }
}
```

## 1.通过拦截器获取访问的接口和参数

```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    HandlerMethod handlerMethod = (HandlerMethod) handler;
    //解析HandlerMethod
    String methodName = handlerMethod.getMethod().getName();
    String className = handlerMethod.getBean().getClass().getName();
    StringBuffer requestBuffer = new StringBuffer();
    Map paramMap = request.getParameterMap();
    Iterator it = paramMap.entrySet().iterator();
    while (it.hasNext()) {
        Map.Entry entry = (Map.Entry) it.next();
        String mapKey = (String) entry.getKey();
        String mapValue = Arrays.toString((String[]) entry.getValue());
        requestBuffer.append(mapKey).append("=").append(mapValue);
    }
    System.out.println("路径：" + className);
    System.out.println("方法：" + methodName);
    System.out.println("参数：" + requestBuffer);
    return true;
}
```

![img](/myres/20180815/20180815004947.png)

## 2.修改返回类容

在做一些验证时不通过如果直接返回一个false对前端并不友好，前后端分离项目通常都是返回Json，而方法的返回值不能修改，这样就只能通过其它方式实现了，比如修改Response

```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    response.reset();//要先reset，否则报异常
    response.setCharacterEncoding("UTF-8");//设置编码，否则会乱码
    response.setContentType("application/json;charset=UTF-8");//这里要设置返回值的类型，因为是json接口。
    PrintWriter out = response.getWriter();
    Map resultMap = new HashMap();
    resultMap.put("success", false);
    resultMap.put("msg", "请登录管理员");
    out.print(JsonUtil.obj2String(resultMap));
    out.flush();
    out.close();//写完要关闭

    return false;
}
```

![img](/myres/20180815/20180815012559.png)

# [附上Demo](https://github.com/A175A174/Demo/tree/master/springmvcinterceptor)

---
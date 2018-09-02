---
title: SpringMVC学习笔记
key: 20180701
tags: Spring
---

# 一、构建工程

以maven形式创建webapp项目

## 1.1、添加jar包依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.0.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```

## 1.2、修改web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd" version="4.0">

<display-name>Archetype Created Web Application</display-name>

<!--springmvc-->
<servlet>
    <servlet-name>spring-dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <!-- 指定配置文件位置 -->
        <param-value>classpath:/spring-mvc.xml</param-value>
    </init-param>
</servlet>
<servlet-mapping>
    <servlet-name>spring-dispatcher</servlet-name>
    <!-- 指定拦截的请求 -->
    <url-pattern>/</url-pattern>
</servlet-mapping>

</web-app>
```

<!--more-->

## 1.3、webapp完整工程目录结构

```note
mywebapp/
├── pom.xml
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com.webapp
│   │   │       └── App.java
│   │   ├── resources
│   │   └── webapp
|   |          ├── index.jsp
|   |          ├── META-INF
|   |          │   └── MANIFEST.MF
|   |          └── WEB-INF
|   |              ├── classes
|   |              ├── lib
|   |              └── web.xml
│   └── test
│       ├── java
│       │   └── com.webapp
│       │               └── AppTest.java
│       └── resources
└── target
```

# 二、常用注解使用

## 2.1、HelloWorld

在resources目录下创建springmvc配置文件

```xml
<!-- spring-mvc.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--配置包扫描-->
    <context:component-scan base-package="com.demo"/>

    <!--会自动注册DefaultAnnotationHandlerMapping与AnnotationMethodHandlerAdapter两个bean,是spring MVC为@Controllers分发请求所必须的-->
    <mvc:annotation-driven/>

    <!-- 访问静态资源 -->
    <!--一般Web应用服务器默认的Servlet名称是"default"，因此DefaultServletHttpRequestHandler可以找到。如果Web应用服务器的默认Servlet名称不是"default"，则需要通过default-servlet-name属性显示指定-->
    <mvc:default-servlet-handler/>

</beans>
```

新建Controller类

```java
package com.demo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
@RequestMapping("/hello")
public class HelloWorld {

    @ResponseBody
    @RequestMapping("/world")
    public String hello(){
        return "hello world";
    }
}
```

访问 <http://localhost:8080/hello/world> 即可看到结果

## 2.2、@RequestMapping

```java
@Controller
@RequestMapping("/hello")
public class HelloWorld {


    // @RequestMapping 支持Ant风格路径匹配
    // ?  匹配一个字符----/user/a??=====/user/adf
    // *  匹配任意个任意字符/user/a*====/useraowf
    // ** 匹配多层路径/user/**/cd====/user/cd

    @RequestMapping("/world")
    public String hello(){
        return null;
    }

    @RequestMapping(value = "/world1",method = RequestMethod.GET)
    public String hello1(){
        return null;
    }

    //请求体中必须包含username和age参数，且age不等于10，否则无法访问
    @RequestMapping(value = "/world2",params = {"username","age!=10"})
    public String hello2(){
        return null;
    }

    //请求头中必须包含Accept-Language，且值为zh-CN,zh;q=0.9，否则无法访问
    @RequestMapping(value = "/world3",headers = {"Accept-Language=zh-CN,zh;q=0.9"})
    public String hello3(){
        return null;
    }
}
```

## 2.3、@PathVariable

```java
//获取请求地址中的值，REST风格，http://localhost:8080/world/123
@RequestMapping(value = "/world4/{id}")
public String hello4(@PathVariable("id") String id){
    return null;
}
```

## 2.4、@RequestParam

```java
//获取请求地址中的值，http://localhost:8080/world?id=123&name=haha
@RequestMapping(value = "/world5")
public String hello5(@RequestParam(value = "id", required = false) String id,
                     @RequestParam(value = "name", defaultValue = "zhuzhu") String name){
    return "hello world";
}
```

## 2.5、@RequestHeader

```java
//获取请求头中的值
@RequestMapping(value = "/gethader6")
public String hello5(@RequestHeader("Accept-Language") String language){
    return "hello world";
}
```

## 2.6、@CookieValue

```java
//获取Cookie中的值
@RequestMapping(value = "/gethader7")
public void hello6(@CookieValue("JSESSIONID") String id){
    System.out.println(id);
}
```

# 三、自动装配

新建pojo类

```java
package com.demo.pojo;

public class User {
    private String name;
    private Integer age;

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    Override
    public String toString() {
        return "User{" + "name='" + name + '\'' + ", age=" + age + '}';
    }
}
```

添加Controller

```java
// 自动装配，支持级联属性，如user.address.tel
// http://localhost:8080/world?name=zhangsan&age=10
@RequestMapping("/world21")
public String hello(User user){
    return "hello world";
}
```

# 四、使用Servlet原生API

```java
//传入原生API
@RequestMapping("/world22")
public void hello1(HttpServletRequest request,
                    HttpServletResponse response,
                    HttpSession session) throws IOException {
    response.getWriter().write("hello world");
}
```

# 五、处理模型数据

配置视图文件前后缀

```xml
<!-- spring-mvc -->
<!--配置视图解析器-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

## 5.1、ModelAndView

```java
@RequestMapping("/world23")
public ModelAndView hello1(){
    ModelAndView modelAndView = new ModelAndView("susu");
    modelAndView.addObject("name","zhangsan");
    return  modelAndView;
}
```

会访问/susu.jsp文件，页面中用以下方式获取值

```jsp
name ： ${requestScope.name}
```

## 5.2、Map、Model、ModelMap

```java
@RequestMapping("/world23")
public String hello1(Map<String,Object> map){
    map.put("name","zhangsan");
    return  "susu";
}
```

同上

## 5.3、@SessionAttributes

```java
// 把数据放到Session中，可以按名称或类型
@SessionAttributes(value = {"user"},types = {String.class})
@Controller
public class HelloWorld2 {
    @RequestMapping("/world24")
    public String hello3(Map<String,Object> map){
        User user = new User("zhangsan",18);
        map.put("user",user);
        map.put("shsh","wosho");
        return  "susu";
    }
}
```

会访问/susu.jsp文件，页面中用以下方式获取值，session域和request域都可以拿到shsh

```jsp
<!-- susu.jsp -->
user ： ${sessionScope.user}
shsh ： ${sessionScope.user}
shsh ： ${requestScope.user}
```

## 5.4、@ModelAttribute

### 5.4.1、修饰方法

被 @ModelAttribute 注解的方法会在Controller每个方法执行之前都执行，因此对于一个Controller中包含多个URL的时候，要谨慎使用。

```java
@RequestMapping("/world21")
public void hello(User user){
    System.out.println(user);
}

@ModelAttribute
public void getUser(Map<String,Object> map){
    User user = new User(null,13);
    map.put("user", user);
}
```

访问 <http://localhost:8080/world21?name=zhangsan> 会打印 User{name='zhangsan', age=13}

springmvc 确定目标方法 POJO 入参过程

1.拿到POJI的参数作为key，POJO 没有使用 @ModelAttribute 注解修饰 key 为 POJO 类名第一个字母小写，使用了该注解 key 为 该注解设置的值

2.在map（implicitModel）中查找key对应的对象，存在则作为 POJO 传入，即在当前类 @ModelAttribute 标注的方法中保存过，且 key 和 1 获取的 key 一样

3.不存在则检查当前类是否使用 @SessionAttributes 注解，若使用，且注解的 value 属性值包含了 key，则会从httpsession中获取对应的值，存在则作为 POJO 传入，不存在且 map 中没有该 key 则抛出异常

4.若当前对象没有使用 @SessionAttributes 注解，或注解的 value 属性不包含key，则会通过反射来创建 POJO 对象传入

5.SpringMVC 会把 key 和 value 保存到 map 中，进而保存到 request 中，jsp 页面就可以获取到

### 5.4.1、修饰入参

```java
@RequestMapping("/world21")
public void hello(@ModelAttribute("abc") User user){
    System.out.println(user);
}

@ModelAttribute
public void getUser(Map<String,Object> map){
    User user = new User(null,13);
    map.put("abc", user);
}
```

jsp 页面获取值的时候也要用 abc

# 6、视图解析器

请求方法返回值类型处理

![img](/myres/201806/28/20180831102131.png)

## 6.1、View

* 视图的作用是渲染模型数据，将模型里的数据以某种形式呈现给客户
* 为了实现视图模型和具体实现技术的解耦，Spring 在org.springframework.web.servlet 包中定义了一个高度抽象的 View接口
* 视图对象由视图解析器负责实例化。由于视图是无状态的，所以他们不会有线程安全的问题

常用视图实现类：

![img](/myres/201806/28/20180831120215.png)

## 6.2、ViewResolver

* SpringMVC 为逻辑视图名的解析提供了不同的策略，可以在 Spring WEB 上下文中配置一种或多种解析策略，并指定他们之间的先后顺序。每一种映射略对应一个具体的视图解析器实现类
* 视图解析器的作用比较单一：将逻辑视图解析为一个具体的视图对象。
* 所有的视图解析器都必须实现 ViewResolver 接口

常用视图解析器实现类：

![img](/myres/201806/28/20180831120756.png)

我们可以选择一种视图解析器或混用多种视图解析器，每个视图解析器都实现了 Ordered 接口并开放出一个 order 属性，可以通过 order 属性指定解析器的优先顺序，order 越小优先级越高，SpringMVC 会按视图解析器顺序的优先顺序对逻辑视图名进行解析，直到解析成功并返回视图对象，否则将抛出 ServletException 异常

## 6.3、InternalResourceView

```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

```java
@RequestMapping("/susu")
public String hello3(){
    return  "susu";
}
```

会去 /WEB-INF/ 目录下寻找 susu.jsp 文件，想寻找多个路径可以如下配置，会按照优先级来

```xml
<bean  id="html" class="org.springframework.web.servlet.view.InternalResourceViewResolver"  >
    <property name="order" value="3344" /><!-- 优先级 -->
    <property name="prefix" value="/WEB-INF/"></property>
    <property name="suffix" value=".html"/>
    <property name="contentType" value="text/html"></property>
</bean>

<bean id="jsp"  class="org.springframework.web.servlet.view.InternalResourceViewResolver" >
    <property name="order" value="44" /><!-- 优先级 -->
    <property name="prefix" value="/WEB-INF/"/>
    <property name="suffix" value=".jsp"/>
    <property name="contentType" value="text/html"/>
</bean>
```

## 6.4、JstlView

若项目中使用了 JSTL，则 SpringMVC 会自动把视图由 InternalResourceView 转为 JstlView

```xml
<!-- pom.xml -->
<!-- 导入JSTL包，1.2 版本包含了 standard -->
<dependency>
    <groupId>jstl</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
```

用 SpringMVC 做国际化

### 6.4.1、创建语言文件

在资源目录下新建 i18n_en_US.properties

```properties
i18n.username=Username
i18n.password=Password
```

再新建 i18n_zh_CN.properties

```properties
i18n.username=\u7528\u6237\u540D
i18n.password=\u5BC6\u7801
```

### 6.4.2、配置springmvc

```xml
<bean class="org.springframework.context.support.ResourceBundleMessageSource">
    <property name="basename" value="i18n"/>
</bean>
```

### 6.4.3、页面中获取

直接使用 <fmt:message> 标签，出现不能识别该标签的错误，因为在web.xml中配置的DispatcherServlet的url-pattern为“/”，不会匹配访问*.jsp的url，所以直接访问首页并不会经过DispatcherServlet，导致无法读取到资源文件，添加 <fmt:bundle> 标签即可

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" pageEncoding="utf-8" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <fmt:bundle basename="i18n">
        <fmt:message key="i18n.username"></fmt:message>
        <br/>
        <fmt:message key="i18n.password"></fmt:message>
    </fmt:bundle>
</body>
</html>
```

### 6.4.3、添加请求

```java
@RequestMapping("/world26")
public String hello4(){
    return  "i18n";
}
```

也可以在springmvc中配置 <<mvc:view-controller>> 标签，这样就不会经过 controller 处理，配置了后其它请求失效添加 <<mvc:annotation-driven/>> 标签就好了

```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/"/>
    <property name="suffix" value=".jsp"/>
</bean>

<mvc:view-controller path="/world26" view-name="/i18n"/>

<mvc:annotation-driven/>
```

## 6.5、自定义视图

### 6.5.1、新建视图解析器类

```java
@Component
public class HelloView implements View {
    //返回内容类型
    @Override
    public String getContentType() {
        return "text/html";
    }

    //渲染视图
    @Override
    public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
        response.getWriter().write("hello world");
    }
}
```

### 6.5.2、配置springmvc

```xml
<!--配置视图解析器，它是用 ioc 容器中名来字解析-->
<bean class="org.springframework.web.servlet.view.BeanNameViewResolver">
    <!--设置视图解析器优先级,先用自己的，不行再用 springmvc 的-->
    <property name="order" value="100"/>
</bean>
```

### 6.5.3、添加 controller 请求

```java
@RequestMapping("/world27")
public String hello5(){
    return  "helloView";
}
```

## 6.6、转发与重定向

### 6.6.1、转发：在服务器端转发，客户端是不知道的，一次请求

```java
@RequestMapping(value="/world28")
public String hello7(){
    return "forward:/helloView";
}
```

### 6.6.2、重定向：在客户端重定向，第一次请求服务端返回302给客户端，客户端会再请求一次服务器，二次请求，第一次传输的数据不会带到第二次

```java
@RequestMapping(value="/world29")
public String hello8(){
    return "redirect:/helloView";
}
```

### 6.6.3、内部实现 UrlBasedViewResolver

```java
@Override
protected View createView(String viewName, Locale locale) throws Exception {
    // If this resolver is not supposed to handle the given view,
    // return null to pass on to the next resolver in the chain.
    if (!canHandle(viewName, locale)) {
        return null;
    }

    // Check for special "redirect:" prefix.
    if (viewName.startsWith(REDIRECT_URL_PREFIX)) {
        String redirectUrl = viewName.substring(REDIRECT_URL_PREFIX.length());
        RedirectView view = new RedirectView(redirectUrl,
                isRedirectContextRelative(), isRedirectHttp10Compatible());
        String[] hosts = getRedirectHosts();
        if (hosts != null) {
            view.setHosts(hosts);
        }
        return applyLifecycleMethods(REDIRECT_URL_PREFIX, view);
    }

    // Check for special "forward:" prefix.
    if (viewName.startsWith(FORWARD_URL_PREFIX)) {
        String forwardUrl = viewName.substring(FORWARD_URL_PREFIX.length());
        return new InternalResourceView(forwardUrl);
    }

    // Else fall back to superclass implementation: calling loadView.
    return super.createView(viewName, locale);
}
```

# 7、RESTful API

form 表单不支持 Delete 和 Put 方式提交，我们可以配置 SpringMVC 来达到目的

```xml
<!-- web.xml -->
<!--form表单请求转换，如post转delete-->
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/</url-pattern>
</filter-mapping>
```

```html
<!-- 前端在 form 表单中添加一个隐藏域就可以了，value 值为要提交的类型 -->
<input type="hidden" name="_method" value="DELETE"/>
```

# 8、处理静态资源

修改 springmvc 配置文件

```xml
<!--一般Web应用服务器默认的Servlet名称是"default"，因此DefaultServletHttpRequestHandler可以找到它。如果你所有的Web应用服务器的默认Servlet名称不是"default"，则需要通过default-servlet-name属性显示指定：-->
<mvc:default-servlet-handler default-servlet-name="default"/>

<!-- mapping：映射，location：本地资源路径，一定要是webapp根目录下。 **，它表示映射/img/下的所有文件 -->
<mvc:resources location="/img/" mapping="/img/**"/>
```

# 9、SpringMVC 表单标签

通过 SpringMVC 的表单标签可以实现将模型数据中的属性和 HTML 表单元素相绑定，以实现表单数据更便捷编辑和表单值的回显

* 一般情况下，通过 GET 请求获取表单页面，而通过POST 请求提交表单页面，因此获取表单页面和提交表单页面的 URL 是相同的。只要满足该最佳条件的契约，<<form:form>> 标签就无需通过 action 属性指定表单提交的 URL
* 可以通过 modelAttribute 属性指定绑定的模型属性，若没有指定该属性，则默认从 request 域对象中读取 command 的表单 bean，如果该属性值也不存在，则会发生错误
* SpringMVC 提供了多个表单组件标签，如 <<form:input/>>、<<form:select/>> 等，用以绑定表单字段的属性值，它们的共有属性如下：
* – path：表单字段，对应 html 元素的 name 属性，支持级联属性
* – htmlEscape：是否对表单值的 HTML 特殊字符进行转换，默认值为 true
* – cssClass：表单组件对应的 CSS 样式类名
* – cssErrorClass：表单组件的数据存在错误时，采取的 CSS 样式
* form:input、form:password、form:hidden、form:textarea：对应 HTML 表单的 text、password、hidden、textarea标签
* form:radiobutton：单选框组件标签，当表单 bean 对应的属性值和 value 值相等时，单选框被选中
* form:radiobuttons：单选框组标签，用于构造多个单选框
* – items：可以是一个 List、String[] 或 Map
* – itemValue：指定 radio 的 value 值。可以是集合中 bean 的一个属性值
* – itemLabel：指定 radio 的 label 值
* – delimiter：多个单选框可以通过 delimiter 指定分隔符
* form:checkbox：复选框组件。用于构造单个复选框
* form:checkboxs：用于构造多个复选框。使用方式同form:radiobuttons 标签
* form:select：用于构造下拉框组件。使用方式同form:radiobuttons 标签
* form:option：下拉框选项组件标签。使用方式同form:radiobuttons 标签
* form:errors：显示表单组件或数据校验所对应的错误
* – <form:errors path= “ *” /> ：显示表单所有的错误
* – <form:errors path= “ user*” /> ：显示所有以 user 为前缀的属性对应的错误
* – <form:errors path= “ username” /> ：显示特定表单对象属性的错误

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" pageEncoding="utf-8" %>
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <form:form action="formTag" method="delete" modelAttribute="user">
        <table>
            <tr>
                <td>Name:</td><td><form:input path="name"/></td>
            </tr>
            <tr>
                <td>Age:</td><td><form:input path="age"/></td>
            </tr>
            <tr>
                <td colspan="2"><input type="submit" value="提交"/></td>
            </tr>
        </table>
    </form:form>
</body>
</html>
```

如果Model中存在一个属性名称为 user 的 javaBean，而且该javaBean拥有属性name和age的时候，在渲染上面的代码时就会取对应属性值赋给对应标签的值

# 10、数据绑定流程

1. Spring MVC 框架将 ServletRequest 对象及目标方法的入参实例传递给 WebDataBinderFactory 实例，以创建 DataBinder 实例对象
2. DataBinder 调用装配在 Spring MVC 上下文中的ConversionService 组件进行数据类型转换、数据格式化工作。将 Servlet 中的请求信息填充到入参对象中
3. 调用 Validator 组件对已经绑定了请求消息的入参对象进行数据合法性校验，并最终生成数据绑定结果BindingData 对象
4. Spring MVC 抽取 BindingResult 中的入参对象和校验错误对象，将它们赋给处理方法的响应入参

Spring MVC 通过反射机制对目标处理方法进行解析，将请求消息绑定到处理方法的入参中。数据绑定的核心部件是DataBinder

![img](/myres/201806/28/20180831180116.png)

## 10.1、自定义数据类型转换器

Spring MVC 上下文中内建了很多转换器，可完成大多数 Java 类型的转换工作。

ConversionService 是 Spring 类型转换体系的核心接口。

* 可以利用 ConversionServiceFactoryBean 在 Spring 的 IOC容器中定义一个 ConversionService. Spring 将自动识别出IOC 容器中的 ConversionService，并在 Bean 属性配置及Spring MVC 处理方法入参绑定等场合使用它进行数据的转换
* 可通过 ConversionServiceFactoryBean 的 converters 属性注册自定义的类型转换器

Spring 支持的转换器：

* Spring 定义了 3 种类型的转换器接口，实现任意一个转换器接口都可以作为自定义转换器注册到ConversionServiceFactroyBean 中：
* – Converter<S,T>：将 S 类型对象转为 T 类型对象
* – ConverterFactory：将相同系列多个 “同质” Converter 封装在一起。如果希望将一种类型的对象转换为另一种类型及其子类的对象（例如将 String 转换为 Number 及 Number 子类（Integer、Long、Double 等）对象）可使用该转换器工厂类
* – GenericConverter：会根据源类对象及目标类对象所在的宿主类中的上下文信息进行类型转换

```java
// 字符串转User对象
package com.demo.converters;
package com.demo.converters;

import com.demo.pojo.User;
import org.springframework.core.convert.converter.Converter;
import org.springframework.stereotype.Component;

@Component
public class UserConverter implements Converter<String, User> {

    @Override
    public User convert(String source) {
        System.out.println(source);
        if (source != null && !"".equals(source)){
            String[] ss = source.split("-");
            User user = new User(ss[0],Integer.parseInt(ss[1]));
            return user;
        }
        return null;
    }
}

// controller层
package com.demo.controller;

import com.demo.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
public class HelloWorld3 {

    @RequestMapping(value = "/world30")
    public void hello9(@RequestParam("user") User user) {
        System.out.println(user);
    }
}
```

```xml
<!-- 配置转换器 -->
<mvc:annotation-driven conversion-service="conversionService"/>

<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <ref bean="userConverter"/>
        </set>
    </property>
</bean>
```

访问 <http://localhost:8080/world30?user=zhangsan-189> 即可看到效果

## 10.2、关于 \<mvc:annotation-driven/>

\<mvc:annotation-driven /> 会自动注册 RequestMappingHandlerMapping、RequestMappingHandlerAdapter 与 ExceptionHandlerExceptionResolver 三个 bean。

* 还将提供以下支持：
* – 支持使用 ConversionService 实例对表单参数进行类型转换
* – 支持使用 @NumberFormat annotation、@DateTimeFormat注解完成数据类型的格式化
* – 支持使用 @Valid 注解对 JavaBean 实例进行 JSR 303 验证
* – 支持使用 @RequestBody 和 @ResponseBody

## 10.3、@InitBinder

由 @InitBinder 标识的方法，可以对 WebDataBinder 对象进行初始化。WebDataBinder 是 DataBinder 的子类，用于完成由表单字段到 JavaBean 属性的绑定

* @InitBinder方法不能有返回值，它必须声明为void。
* @InitBinder方法的参数通常是是 WebDataBinder

```java
package com.demo.converters;

import com.demo.pojo.User;
import org.springframework.core.convert.converter.Converter;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.InitBinder;

@Component
public class UserConverter implements Converter<String, User> {

    @InitBinder
    public void initBinder(WebDataBinder dataBinder){
        // 不自动绑定 name 属性
        dataBinder.setDisallowedFields("name");
    }

    @Override
    public User convert(String source) {
        System.out.println(source);
        if (source != null && !"".equals(source)){
            String[] ss = source.split("-");
            User user = new User(ss[0],Integer.parseInt(ss[1]));
            return user;
        }
        return null;
    }
}
```

## 10.4、自定义数据格式转换器

对属性对象的输入/输出进行格式化，从其本质上讲依然属于 “类型转换” 的范畴。

* Spring 在格式化模块中定义了一个实现ConversionService 接口的FormattingConversionService 实现类，该实现类扩展了 GenericConversionService，因此它既具有类型转换的功能，又具有格式化的功能
* FormattingConversionService 拥有一个FormattingConversionServiceFactroyBean 工厂类，后者用于在 Spring 上下文中构造前者
* FormattingConversionServiceFactroyBean 内部已经注册了 :
* – NumberFormatAnnotationFormatterFactroy：支持对数字类型的属性使用 @NumberFormat 注解
* – JodaDateTimeFormatAnnotationFormatterFactroy：支持对日期类型的属性使用 @DateTimeFormat 注解
* 装配了 FormattingConversionServiceFactroyBean 后，就可以在 Spring MVC 入参绑定及模型数据输出时使用注解驱动了。\<mvc:annotation-driven/> 默认创建的ConversionService 实例即为FormattingConversionServiceFactroyBean

### 10.4.1、日期格式化

@DateTimeFormat 注解可对java.util.Date、java.util.Calendar、java.long.Long 时间类型进行标注：

* – pattern 属性：类型为字符串。指定解析/格式化字段数据的模式，如：”yyyy-MM-dd hh:mm:ss”
* – iso 属性：类型为 DateTimeFormat.ISO。指定解析/格式化字段数据的ISO模式，包括四种：ISO.NONE（不使用） -- 默认、ISO.DATE(yyyy-MM-dd) 、ISO.TIME(hh:mm:ss.SSSZ)、ISO.DATE_TIME(yyyy-MM-dd hh:mm:ss.SSSZ)
* – style 属性：字符串类型。通过样式指定日期时间的格式，由两位字符组成，第一位表示日期的格式，第二位表示时间的格式：S：短日期/时间格式、M：中日期/时间格式、L：长日期/时间格式、F：完整日期/时间格式、-：忽略日期或时间格式

```java
// 在对象自动绑定的时候就按照这个格式来传，必须配置 <mvc:annotation-driven/>

@DateTimeFormat(pattern = "yyyy-MM-dd")
private Date shengri;
```

### 10.4.2、数值格式化

@NumberFormat 可对类似数字类型的属性进行标注，它拥有两个互斥的属性：

* – style：类型为 NumberFormat.Style。用于指定样式类型，包括三种：Style.NUMBER（正常数字类型）、Style.CURRENCY（货币类型）、 Style.PERCENT（百分数类型）
* – pattern：类型为 String，自定义样式，如 patter="#,###"

```java
// 把制定格式字符串转换为数值，12,12,3--->1212.3

@NumberFormat(pattern = "##,##.#")
private Float money;
```

配置数据格式化后又想配置数据类型转换

```xml
<mvc:annotation-driven conversion-service="conversionService"/>

<!-- 不使用 org.springframework.context.support.ConversionServiceFactoryBean -->
<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <ref bean="userConverter"/>
        </set>
    </property>
</bean>
```

如果转换出错，获取错误信息

```java
@RequestMapping(value = "/world30")
public void hello9(@RequestParam("user") User user, BindingResult result) {
    System.out.println(user);
    if (result.getErrorCount() > 0){
        for (FieldError fieldError : result.getFieldErrors()){
            System.out.println(fieldError.getField() + ":" + fieldError.getDefaultMessage());
        }
    }
}
```

## 10.5、JSR 303 数据校验

* JSR 303 是 Java 为 Bean 数据合法性校验提供的标准框架，它已经包含在 JavaEE 6.0 中 .
* JSR 303 通过在 Bean 属性上标注类似于 @NotNull、@Max等标准的注解指定校验规则，并通过标准的验证接口对 Bean进行验证

![img](/myres/201806/28/20180831204214.png)

* Hibernate Validator 是 JSR 303 的一个参考实现，除支持所有标准的校验注解外，它还支持以下的扩展注解

![img](/myres/201806/28/20180831204353.png)

### 10.5.1、Spring MVC 数据校验

* Spring 拥有自己独立的数据校验框架，同时支持 JSR 303 标准的校验框架。
* Spring 在进行数据绑定时，可同时调用校验框架完成数据校验工作。在 Spring MVC 中，可直接通过注解驱动的方式进行数据校验
* Spring 的 LocalValidatorFactroyBean 既实现了 Spring 的Validator 接口，也实现了 JSR 303 的 Validator 接口。只要在 Spring 容器中定义了一个LocalValidatorFactoryBean，即可将其注入到需要数据校验的 Bean 中。
* Spring 本身并没有提供 JSR303 的实现，所以必须将JSR303 的实现者的 jar 包放到类路径下。
* \<mvc:annotation-driven/> 会默认装配好一个LocalValidatorFactoryBean，通过在处理方法的入参上标注 @valid 注解即可让 Spring MVC 在完成数据绑定后执行数据校验的工作
* 在已经标注了 JSR303 注解的表单/命令对象前标注一个@Valid，Spring MVC 框架在将请求参数绑定到该入参对象后，就会调用校验框架根据注解声明的校验规则实施校验
* Spring MVC 是通过对处理方法签名的规约来保存校验结果的：前一个表单/命令对象的校验结果保存到随后的入参中，这个保存校验结果的入参必须是 BindingResult 或Errors 类型，这两个类都位于org.springframework.validation 包中
* 需校验的 Bean 对象和其绑定结果对象或错误对象时成对出现的，它们之间不允许声明其他的入参
* Errors 接口提供了获取错误信息的方法，如 getErrorCount() 或 getFieldErrors(String field)
* BindingResult 扩展了 Errors 接口

1.添加 Hibernate Validator 的 jar 包

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.12.Final</version>
</dependency>
```

2.在 POJO 类的属性上使用注解

```java
@NotNull
private String name;
```

3.在 Controller 方法的入参上添加注解

```java
@RequestMapping(value = "/world31")
public void hello10(@Valid User user, BindingResult result) {
    System.out.println(user);
}
```

访问 <http://localhost:8080/world31?age=18>

![img](/myres/201806/28/20180831211059.png)

### 10.5.2、在页面上显示数据校验错误的消息

在目标方法中获取校验结果

* 在表单/命令对象类的属性中标注校验注解，在处理方法对应的入参前添加 @Valid，Spring MVC 就会实施校验并将校验结果保存在被校验入参对象之后的 BindingResult 或Errors 入参中。常用方法：
* – FieldError getFieldError(String field)
* – List<FieldError> getFieldErrors()
* – Object getFieldValue(String field)
* – Int getErrorCount()

Spring MVC 除了会将表单/命令对象的校验结果保存到对应的 BindingResult 或 Errors 对象中外，还会将所有校验结果保存到 “隐含模型”

* 即使处理方法的签名中没有对应于表单/命令对象的结果入参，校验结果也会保存在 “隐含对象” 中。
* 隐含模型中的所有数据最终将通过 HttpServletRequest 的属性列表暴露给 JSP 视图对象，因此在 JSP 中可以获取错误信息
* 在 JSP 页面上可通过 <form:errors path=“userName”> 显示错误消息

```java
// 校验错误重定向页面，带上错误消息
@RequestMapping(value = "/world32")
public String hello11(@Valid User user, BindingResult result, Map<String,Object> map) {
    System.out.println(user);
    if (result.getErrorCount() > 0){
        return "world32";
    }
    return "ok";
}
```

页面上获取

```jsp
<!-- 获取所有错误 -->
<form:errors path="*"/>

<!-- 获取指定字段错误 -->
<form:errors path="name"/>
```

### 10.5.2、国际化数据校验错误消息

每个属性在数据绑定和数据校验发生错误时，都会生成一个对应的 FieldError 对象。

* 当一个属性校验失败后，校验框架会为该属性生成 4 个消息代码，这些代码以校验注解类名为前缀，结合 modleAttribute、属性名及属性类型名生成多个对应的消息代码：例如 User 类中的 password 属性标注了一个 @Pattern 注解，当该属性值不满足 @Pattern 所定义的规则时, 就会产生以下 4个错误代码：
* – Pattern.user.password
* – Pattern.password
* – Pattern.java.lang.String
* – Pattern
* 当使用 Spring MVC 标签显示错误消息时， Spring MVC 会查看WEB 上下文是否装配了对应的国际化消息，如果没有，则显示默认的错误消息，否则使用国际化消息。
* 若数据类型转换或数据格式转换时发生错误，或该有的参数不存在，或调用处理方法时发生错误，都会在隐含模型中创建错误消息。其错误代码前缀说明如下：
* – required：必要的参数不存在。如 @RequiredParam(“param1”) 标注了一个入参，但是该参数不存在
* – typeMismatch：在数据绑定时，发生数据类型不匹配的问题
* – methodInvocation：Spring MVC 在调用处理方法时发生了错误
* 注册国际化资源文件

1.添加国际化文件

```properties
NotEmpty.user.name=name error,is not empty
```

2.注册国际化资源文件

```xml
<bean class="org.springframework.context.support.ResourceBundleMessageSource">
    <property name="basename" value="i18n"/>
</bean>
```

# 11、返回 JSON 类型数据

## 11.1、实现

1.添加jar包

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.6</version>
</dependency>
```

2.添加 @ResponseBody 注解

```java
@ResponseBody
@RequestMapping(value = "/world33")
public List<User> hello11() {
    List<User> list = new LinkedList<>();
    list.add(new User("zahndan",18));
    list.add(new User("lisi",19));
    return  list;
}
```

## 11.2、实现原理，HttpMessageConverter\<T>

* HttpMessageConverter<T> 是 Spring3.0 新添加的一个接口，负责将请求信息转换为一个对象（类型为 T），将对象（类型为 T）输出为响应信息
* HttpMessageConverter<T>接口定义的方法：
* – Boolean canRead(Class<?> clazz,MediaType mediaType): 指定转换器可以读取的对象类型，即转换器是否可将请求信息转换为 clazz 类型的对象，同时指定支持 MIME 类型(text/html,applaiction/json等)
* – Boolean canWrite(Class<?> clazz,MediaType mediaType):指定转换器是否可将 clazz 类型的对象写到响应流中，响应流支持的媒体类型在MediaType 中定义。
* – List\<MediaType> getSupportMediaTypes()：该转换器支持的媒体类型。
* – T read(Class<? extends T> clazz,HttpInputMessage inputMessage)：将请求信息流转换为 T 类型的对象。
* – void write(T t,MediaType contnetType,HttpOutputMessgaeoutputMessage):将T类型的对象写到响应流中，同时指定相应的媒体类型为 contentType。

![img](/myres/201806/28/20180831215131.png)

HttpMessageConverter\<T> 的实现类

![img](/myres/201806/28/20180831215315.png)

## 11.3、HttpMessageConverter\<T> 的使用

* 使用 HttpMessageConverter<T> 将请求信息转化并绑定到处理方法的入参中或将响应结果转为对应类型的响应信息，Spring 提供了两种途径：
* – 使用 @RequestBody / @ResponseBody 对处理方法进行标注
* – 使用 HttpEntity<T> / ResponseEntity<T> 作为处理方法的入参或返回值
* 当控制器处理方法使用到 @RequestBody/@ResponseBody 或 HttpEntity<T>/ResponseEntity<T> 时, Spring 首先根据请求头或响应头的 Accept 属性选择匹配的 HttpMessageConverter, 进而根据参数类型或泛型类型的过滤得到匹配的 HttpMessageConverter, 若找不到可用的 HttpMessageConverter 将报错
* @RequestBody 和 @ResponseBody 不需要成对出现

# 11、国际化

* 默认情况下，SpringMVC 根据 Accept-Language 参数判断客户端的本地化类型。
* 当接受到请求时，SpringMVC 会在上下文中查找一个本地化解析器（LocalResolver），找到后使用它获取请求所对应的本地化类型信息。
* SpringMVC 还允许装配一个动态更改本地化类型的拦截器，这样通过指定一个请求参数就可以控制单个请求的本地化类型。

## 11.1、SessionLocaleResolver、LocaleChangeInterceptor 工作原理

![img](/myres/201806/28/20180831224308.png)

## 11.2、本地化解析器和本地化拦截器

* AcceptHeaderLocaleResolver：根据 HTTP 请求头的Accept-Language 参数确定本地化类型，如果没有显式定义本地化解析器， SpringMVC 使用该解析器。
* CookieLocaleResolver：根据指定的 Cookie 值确定本地化类型
* SessionLocaleResolver：根据 Session 中特定的属性确定本地化类型
* LocaleChangeInterceptor：从请求参数中获取本次请求对应的本地化类型

```xml
 <!-- 存储区域设置信息SessionLocaleResolver类通过一个预定义会话名将区域化信息存储在会话中。 -->
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver" />

<mvc:interceptors>
    <!-- 该拦截器拦截HTTP请求，使其重新设置页面的区域化信息 -->
    <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"></bean>
</mvc:interceptors>
```

# 12、文件上传

* Spring MVC 为文件上传提供了直接的支持，这种支持是通过即插即用的 MultipartResolver 实现的。Spring 用 Jakarta Commons FileUpload 技术实现了一个MultipartResolver 实现类：CommonsMultipartResovler
* Spring MVC 上下文中默认没有装配 MultipartResovler，因此默认情况下不能处理文件的上传工作，如果想使用 Spring的文件上传功能，需现在上下文中配置 MultipartResolver

配置 MultipartResolver

* defaultEncoding: 必须和用户 JSP 的 pageEncoding 属性一致，以便正确解析表单的内容
* 为了让 CommonsMultipartResovler 正确工作，必须先将 Jakarta Commons FileUpload 及 Jakarta Commons io的类包添加到类路径下。

1.添加 jar 包

```xml
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.3</version>
</dependency>
```

2.配置SpringMVC

```xml
<bean class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="defaultEncoding" value="UTF-8"/>
    <property name="maxUploadSize" value="10240000"/>
</bean>
```

3.配置 web.xml

```xml
<servlet>
    <servlet-name>spring-dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:/spring-mvc.xml</param-value>
    </init-param>
    <multipart-config>
        <location>/</location>
        <max-file-size>5242880</max-file-size><!--5MB-->
        <max-request-size>20971520</max-request-size><!--20MB-->
        <file-size-threshold>0</file-size-threshold>
    </multipart-config>
</servlet>
<servlet-mapping>
    <servlet-name>spring-dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

4.添加 Controller 层请求

```java
@RequestMapping(value = "/up")
public void up(@RequestParam("file") MultipartFile file) {
    System.out.println(file.getOriginalFilename());
}
```

5.前端页面

```jsp
<form action="/up" method="post"  enctype="multipart/form-data">
    <input type="file" name="file">
    <input type="submit" value="upload"/>
</form>
```

# 13、异常处理

Spring MVC 通过 HandlerExceptionResolver 处理程序的异常，包括 Handler 映射、数据绑定以及目标方法执行时发生的异常。

## 13.1、ExceptionHandlerExceptionResolver

* 主要处理 Handler 中用 @ExceptionHandler 注解定义的方法。
* @ExceptionHandler 注解定义的方法优先级问题：例如发生的是NullPointerException，但是声明的异常有RuntimeException 和 Exception，此候会根据异常的最近继承关系找到继承深度最浅的那个 @ExceptionHandler注解方法，即标记了 RuntimeException 的方法
* ExceptionHandlerMethodResolver 内部若找不到@ExceptionHandler 注解的话，会找@ControllerAdvice 中的@ExceptionHandler 注解方法

## 13.2、ResponseStatusExceptionResolver

* 在异常及异常父类中找到 @ResponseStatus 注解，然后使用这个注解的属性进行处理。
* 定义一个 @ResponseStatus 注解修饰的异常类
* 若在处理器方法中抛出了上述异常：若ExceptionHandlerExceptionResolver 不解析述异常。由于触发的异常 UnauthorizedException 带有@ResponseStatus注解。因此会被ResponseStatusExceptionResolver 解析到。最后响应HttpStatus.UNAUTHORIZED 代码给客户端。HttpStatus.UNAUTHORIZED 代表响应码401，无权限。关于其他的响应码请参考 HttpStatus 枚举类型源码。

## 13.3、DefaultHandlerExceptionResolver

* 对一些特殊的异常进行处理，比如NoSuchRequestHandlingMethodException、HttpRequestMethodNotSupportedException、HttpMediaTypeNotSupportedException、HttpMediaTypeNotAcceptableException等。

## 13.4、SimpleMappingExceptionResolver

* 如果希望对所有异常进行统一处理，可以使用SimpleMappingExceptionResolver，它将异常类名映射为视图名，即发生异常时使用对应的视图报告异常

---
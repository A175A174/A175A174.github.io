---
title: Spring Boot入门下
key: 20180716
tags: SpringBoot
---

# 一、日志

## [官方文档](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/reference/htmlsingle/#boot-features-logging)

## 1、日志框架

**市面上的日志框架**：JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j....

| 日志门面  （日志的抽象层）                 | 日志实现                        |
| ---------------------------------------- | ------------------------------- |
| ~~JCL（Jakarta Commons Logging）~~ SLF4j（Simple Logging Facade for Java） **~~jboss-logging~~** | Log4j  JUL（java.util.logging）  Log4j2  **Logback** |

左边选一个门面（抽象层）、右边来选一个实现：

SpringBoot：底层是Spring框架，Spring框架默认是用JCL，SpringBoot 选用 SLF4j 和 logback

<!--more-->

## 2、SLF4j使用

### 2.1、如何在系统中使用SLF4j

[官方文档](https://www.slf4j.org/manual.html)

开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类，而是调用日志抽象层里面的方法

给系统里面导入slf4j的jar和  logback的实现jar

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

![img](/myres/201807/16/concrete-bindings.png)

每一个日志的实现框架都有自己的配置文件。使用slf4j以后，**配置文件还是做成日志实现框架自己本身的配置文件**

### 2.2、遗留问题

我使用的是slf4j+logback，但使用的其他jar包有一些自己日志实现: Spring（commons-logging）、Hibernate（jboss-logging）、MyBatis、xxxx

统一日志记录，即使是别的框架和我一起统一使用slf4j进行输出

![img](/myres/201807/16/legacy.png)

如何让系统中所有的日志都统一到slf4j：

1、将系统中其他日志框架先排除出去

2、用中间包来替换原有的日志框架

3、我们导入slf4j其他的实现

## 3、SpringBoot日志框架关系

![img](/myres/201807/16/20180828133614.png)

可以看到SpringBoot底层也是使用slf4j+logback的方式进行日志记录，SpringBoot用中间替换包也把其他的日志都替换成了slf4j

SpringBoot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志，引入其他框架的时候，只需要把这个框架依赖的日志框架**排除掉**即可

```xml
<dependency>
    <groupId>xxxx.xxxxxx</groupId>
    <artifactId>xxxxx</artifactId>
    <exclusions>
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## 4、日志使用

### 4.1、默认配置

SpringBoot默认配置好了日志，可以直接使用

```java
//记录器
Logger logger = LoggerFactory.getLogger(getClass());
@Test
public void contextLoads() {
    //System.out.println();
    //日志的级别；
    //由低到高   trace<debug<info<warn<error
    //可以调整输出的日志级别；日志就只会在这个级别以以后的高级别生效
    logger.trace("这是trace日志...");
    logger.debug("这是debug日志...");
    //SpringBoot默认给我们使用的是info级别的，没有指定级别的就用SpringBoot默认规定的级别；root级别
    logger.info("这是info日志...");
    logger.warn("这是warn日志...");
    logger.error("这是error日志...");
}
```

SpringBoot一些日志配置

```properties
# application.properties

# 调整com包下的日志输出级别
logging.level.com=trace

# 不指定路径在当前项目下生成springboot.log日志，也可以指定完整的路径
# logging.file=G:/springboot.log

# 在当前磁盘的根路径下创建spring文件夹和里面的log文件夹，使用 spring.log 作为默认文件
logging.path=/spring/log

# 在控制台输出的日志的格式
logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n

# 指定文件中日志输出的格式
logging.pattern.file=%d{yyyy-MM-dd} === [%thread] === %-5level === %logger{50} ==== %msg%n
```

```properties
日志输出格式：
    %d 表示日期时间
    %thread 表示线程名
    %-5level 级别从左显示5个字符宽度
    %logger{50} 表示logger名字最长50个字符，否则按照句点分割
    %msg 日志消息，
    %n 是换行符
----------------------------------------------------------------------->
%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
```

logging.file 和 logging.path 二选一，区别如下

| logging.file | logging.path | Example  | Description             |
| ------------ | ------------ | -------- | ----------------------- |
| (none)       | (none)       |          | 只在控制台输出           |
| 指定文件名    | (none)       | my.log   | 输出日志到my.log文件     |
| (none)       | 指定目录     | /var/log | 输出到指定目录的 spring.log 文件中 |

### 4.2、指定配置

给类路径下放上每个日志框架自己的配置文件即可；SpringBoot就不使用他默认配置的了

| Logging System          | Customization                            |
| ----------------------- | ---------------------------------------- |
| Logback                 | `logback-spring.xml` or `logback-spring.groovy` or `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`      |
| JDK (Java Util Logging) | `logging.properties`                     |

关于日志配置文件名字

logback.xml：直接就被日志框架识别了；

logback-spring.xml：日志框架就不直接加载日志的配置项，由SpringBoot解析日志配置，可以使用SpringBoot的高级Profile功能

```xml
<!-- 可以指定某段配置只在某些环境下生效 -->
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>
<springProfile name="dev, staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>
<springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

```xml
<!-- logback-spring.xml -->
<appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
    <layout class="ch.qos.logback.classic.PatternLayout">
        <springProfile name="dev"><!-- dev环境日志格式 -->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n</pattern>
        </springProfile>
        <springProfile name="!dev"><!-- 不是dev环境日志格式 -->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ==== [%thread] ==== %-5level %logger{50} - %msg%n</pattern>
        </springProfile>
    </layout>
</appender>
```

如果使用logback.xml作为日志配置文件，还要使用profile功能，会有以下错误

 `no applicable action for [springProfile]`

## 5、切换日志框架

可以按照slf4j的日志适配图，进行相关的切换，实际开发当中不推荐这样做

```xml
<!-- slf4j+log4j的方式 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>logback-classic</artifactId>
            <groupId>ch.qos.logback</groupId>
        </exclusion>
        <exclusion>
            <artifactId>log4j-over-slf4j</artifactId>
            <groupId>org.slf4j</groupId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
</dependency>
```

```xml
<!-- 切换为log4j2 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-logging</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

# 二、Web开发

## 1、简介

使用SpringBoot；

1）、创建SpringBoot应用，选中我们需要的模块

2）、SpringBoot已经默认将这些场景配置好了，只需要在配置文件中指定少量配置就可以运行起来

3）、自己编写业务代码

## 2、SpringBoot对静态资源的映射规则

```java
// 资源配置类，可以设置和静态资源有关的参数，缓存时间等
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {

    private static final String[] CLASSPATH_RESOURCE_LOCATIONS = {
            "classpath:/META-INF/resources/", "classpath:/resources/",
            "classpath:/static/", "classpath:/public/" };
```

查看SpringMVC自动配置类WebMvcAuotConfiguration

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
    CacheControl cacheControl = this.resourceProperties.getCache()
            .getCachecontrol().toHttpCacheControl();
    if (!registry.hasMappingForPattern("/webjars/**")) {
        customizeResourceHandlerRegistration(registry
                .addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/")
                .setCachePeriod(getSeconds(cachePeriod))
                .setCacheControl(cacheControl));
    }
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    // 静态资源文件夹配置
    if (!registry.hasMappingForPattern(staticPathPattern)) {
        customizeResourceHandlerRegistration(
                registry.addResourceHandler(staticPathPattern)
                        .addResourceLocations(getResourceLocations(
                                this.resourceProperties.getStaticLocations()))
                        .setCachePeriod(getSeconds(cachePeriod))
                        .setCacheControl(cacheControl));
    }
}

// 欢迎页配置
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(
        ApplicationContext applicationContext) {
    return new WelcomePageHandlerMapping(
            new TemplateAvailabilityProviders(applicationContext),
            applicationContext, getWelcomePage(),
            this.mvcProperties.getStaticPathPattern());
}

// 网页图标配置
@Configuration
@ConditionalOnProperty(value = "spring.mvc.favicon.enabled", matchIfMissing = true)
public static class FaviconConfiguration implements ResourceLoaderAware {

    private final ResourceProperties resourceProperties;

    private ResourceLoader resourceLoader;

    public FaviconConfiguration(ResourceProperties resourceProperties) {
        this.resourceProperties = resourceProperties;
    }

    @Override
    public void setResourceLoader(ResourceLoader resourceLoader) {
        this.resourceLoader = resourceLoader;
    }

    @Bean
    public SimpleUrlHandlerMapping faviconHandlerMapping() {
        SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
        mapping.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);
        mapping.setUrlMap(Collections.singletonMap("**/favicon.ico",
                faviconRequestHandler()));
        return mapping;
    }

    @Bean
    public ResourceHttpRequestHandler faviconRequestHandler() {
        ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
        requestHandler.setLocations(resolveFaviconLocations());
        return requestHandler;
    }

    private List<Resource> resolveFaviconLocations() {
        String[] staticLocations = getResourceLocations(
                this.resourceProperties.getStaticLocations());
        List<Resource> locations = new ArrayList<>(staticLocations.length + 1);
        Arrays.stream(staticLocations).map(this.resourceLoader::getResource)
                .forEach(locations::add);
        locations.add(new ClassPathResource("/"));
        return Collections.unmodifiableList(locations);
    }

}
```

### 2.1）、所有 /webjars/** 都去 classpath:/META-INF/resources/webjars/ 找资源

[​webjars](http://www.webjars.org/)：以jar包的方式引入静态资源

```xml
<!--引入jquery-webjar，访问路径：localhost:8080/webjars/jquery/3.3.1/jquery.js-->
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.3.1</version>
</dependency>
```

### 2.2）、"/**" 访问当前项目的任何资源，都去静态资源文件夹找

```properties
"classpath:/META-INF/resources/"
"classpath:/resources/"
"classpath:/static/"
"classpath:/public/"
"/"：当前项目的根路径
```

localhost:8080/abc ===>  去静态资源文件夹里面找abc

### 2.3）、欢迎页：静态资源文件夹下的所有index.html页面；被"/**"映射

localhost:8080/   找index页面

### 2.4）、所有的 **/favicon.ico  都是在静态资源文件下找

## 3、模板引擎

JSP、Velocity、Freemarker、Thymeleaf

![img](/myres/201807/16/template-engine.png)

SpringBoot推荐的Thymeleaf，语法更简单，功能更强大；

### 3.1、引入thymeleaf

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<!-- springboot 1.x 需要切换thymeleaf版本 -->
<properties>
    <!-- thymeleaf3 -> layout2-->
    <!-- thymeleaf2 -> layout1-->
    <thymeleaf.version>3.0.9.RELEASE</thymeleaf.version>
    <thymeleaf-layout-dialect.version>2.2.2</thymeleaf-layout-dialect.version>
</properties>
```

### 3.2、Thymeleaf使用

查看自动配置类的默认规则

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

    private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;

    public static final String DEFAULT_PREFIX = "classpath:/templates/";

    public static final String DEFAULT_SUFFIX = ".html";
```

可以看到前后缀，我们只要把HTML页面放在classpath:/templates/下，thymeleaf就能自动渲染了

使用：

```html
<!DOCTYPE html>
<!-- 导入thymeleaf的名称空间，有语法提示 -->
<html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
        <h1>成功！</h1>
        <!--th:text 将div里面的文本内容设置为 -->
        <div th:text="${hello}">这是显示欢迎信息</div>
    </body>
</html>
```

### 3.3、Thymeleaf语法规则

![img](/myres/201807/16/2018-07-16_1218.png)

3.3.1）、th:text：改变当前元素里面的文本内容

​th：任意html属性；来替换原生属性的值

3.3.2）、表达式

```properties
Simple expressions:（表达式语法）
    ● Variable Expressions: ${...}：获取变量值；OGNL；
        1）、获取对象的属性、调用方法
        2）、使用内置的基本对象：
        #ctx : the context object.
        #vars: the context variables.
        #locale : the context locale.
        #request : (only in Web Contexts) the HttpServletRequest object.
        #response : (only in Web Contexts) the HttpServletResponse object.
        #session : (only in Web Contexts) the HttpSession object.
        #servletContext : (only in Web Contexts) the ServletContext object.
        ${session.foo} # 从session中取值
        3）、内置的一些工具对象：
        #execInfo : information about the template being processed.
        #messages : methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
        #uris : methods for escaping parts of URLs/URIs
        #conversions : methods for executing the configured conversion service (if any).
        #dates : methods for java.util.Date objects: formatting, component extraction, etc.
        #calendars : analogous to #dates , but for java.util.Calendar objects.
        #numbers : methods for formatting numeric objects.
        #strings : methods for String objects: contains, startsWith, prepending/appending, etc.
        #objects : methods for objects in general.
        #bools : methods for boolean evaluation.
        #arrays : methods for arrays.
        #lists : methods for lists.
        #sets : methods for sets.
        #maps : methods for maps.
        #aggregates : methods for creating aggregates on arrays or collections.
        #ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).
    ● Selection Variable Expressions: *{...}：选择表达式：和${}在功能上是一样，可以配合 th:object="${session.user}使用：
        <div th:object="${session.user}">
            <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
            <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
            <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
        </div>
    ● Message Expressions: #{...}：获取国际化内容
    ● Link URL Expressions: @{...}：定义URL
        @{/order/process(execId=${execId},execType='FAST')}
    ● Fragment Expressions: ~{...}：片段引用表达式
        <div th:insert="~{commons :: main}">...</div>
-------------------------------------------------------------------------
Literals（字面量）
    ● Text literals: 'one text' , 'Another one!' ,…
    ● Number literals: 0 , 34 , 3.0 , 12.3 ,…
    ● Boolean literals: true , false
    ● Null literal: null
    ● Literal tokens: one , sometext , main ,…
-------------------------------------------------------------------------
Text operations:（文本操作）
    ● String concatenation: +
    ● Literal substitutions: |The name is ${name}|
-------------------------------------------------------------------------
Arithmetic operations:（数学运算）
    ● Binary operators: + , - , * , / , %
    ● Minus sign (unary operator): -
-------------------------------------------------------------------------
Boolean operations:（布尔运算）
    ● Binary operators: and , or
    ● Boolean negation (unary operator): ! , not
-------------------------------------------------------------------------
Comparisons and equality:（比较运算）
    ● Comparators: > , < , >= , <= ( gt , lt , ge , le )
    ● Equality operators: == , != ( eq , ne )
-------------------------------------------------------------------------
Conditional operators:条件运算（三元运算符）
    ● If-then: (if) ? (then)
    ● If-then-else: (if) ? (then) : (else)
    ● Default: (value) ?: (defaultvalue)
-------------------------------------------------------------------------
Special tokens:
    ● No-Operation: _
```

## 4、SpringMVC自动配置

## [官方文档](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/reference/htmlsingle/#boot-features-developing-web-applications)

Spring Boot 对 SpringMVC 做了自动配置：

* Inclusion of ContentNegotiatingViewResolver and BeanNameViewResolver beans.
  - 自动配置了ViewResolver（视图解析器：根据方法的返回值得到视图对象（View），视图对象决定如何渲染（转发或重定向））
  - ContentNegotiatingViewResolver：组合所有的视图解析器的
  - 如何定制：我们可以自己给容器中添加一个视图解析器，SpringBoot自动的将其组合进来
* Support for serving static resources, including support for WebJars (covered later in this document)).
  - 静态资源文件夹路径，webjars
* Automatic registration of Converter, GenericConverter, and Formatter beans.
  - 自动注册了 `Converter`, `GenericConverter`, `Formatter` beans.
  - Converter：转换器；  public String hello(User user)：类型转换使用Converter
  - `Formatter`  格式化器；  2017.12.17===Date
* Support for HttpMessageConverters (covered later in this document).
  - HttpMessageConverter：SpringMVC用来转换Http请求和响应的；User---Json
  - `HttpMessageConverters` 是从容器中确定；获取所有的HttpMessageConverter
  - 自己给容器中添加HttpMessageConverter，只需要将自己的组件注册容器中（@Bean,@Component）
* Automatic registration of MessageCodesResolver (covered later in this document).
  - 定义错误代码生成规则
* Static index.html support.
  - 静态首页访问
* Custom Favicon support (covered later in this document).
  - favicon.ico 静态资源，网页图标
* Automatic use of a ConfigurableWebBindingInitializer bean (covered later in this document).
  - 我们可以配置一个ConfigurableWebBindingInitializer来替换默认的；（添加到容器）
  - 初始化WebDataBinder，请求数据=====JavaBean

---
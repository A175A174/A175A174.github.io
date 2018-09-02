---
title: Spring Boot 入门上
key: 20180701
tags: SpringBoot
---

# 一、HelloWorld

1.新建Maven工程，添加依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.4.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

2.新建启动类

```java
@SpringBootApplication
public class Main {
    public static void main(String[] args) {
        SpringApplication.run(Main.class,args);
    }
}
```

3.新建Controllet类

```java
@RestController
public class HelloController {
    @RequestMapping(value = "/hello")
    public String hello(){
        return "hello world!";
    }
}
```

运行启动类，访问<http://127.0.0.1:8080/hello>就可以看到效果

![img](/myres/201807/15/20180827174038.png)

<!--more-->

4.部署

4.1添加maven插件

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

4.2打包运行

![img](/myres/201807/15/20180827174742.png)

用java命令运行，访问<http://127.0.0.1:8080/hello>就可以看到效果

```cmd
java -jar jar文件路径
```

# 二、HelloWorldj简单分析

1.pom.xml文件

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.4.RELEASE</version>
</parent>

里面引用了，它是真正管理SpringBoot里面所有依赖jar包的版本
以后导入依赖默认是不需要写版本，里面没有的依赖才需要声明版本号
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.0.4.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>


spring-boot-starter：spring-boot场景启动器
spring-boot-starter-web：帮我们导入了web模块正常运行所依赖的组件
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
Spring Boot将所有的功能场景都抽取出来，做成一个个的starters（启动器）
要用什么功能就导入什么场景的启动器，相关依赖会自动导入
```

2.启动类

```java
@SpringBootApplication
public class Main {
    public static void main(String[] args) {
        SpringApplication.run(Main.class,args);
    }
}
```

@**SpringBootApplication**：标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用；

查看SpringBootApplication注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration //Spring Boot配置类，里面有@Configuration注解，类似以前的xml配置
@EnableAutoConfiguration //开启自动配置功能
@ComponentScan(excludeFilters = {
        @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

查看 @EnableAutoConfiguration 注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage //自动配置包
@Import(AutoConfigurationImportSelector.class) //导入配置选择器
public @interface EnableAutoConfiguration {
```

查看 AutoConfigurationImportSelector.class 类，Debug运行

![img](/myres/201807/15/20180827183533.png)

可以看到我们之所以不用写配置文件是应为SpringBoot帮我们导入了许多自动配置类，那么这些配置类从哪来，点进去可以看到有个自动配置包

![img](/myres/201807/15/20180827183534.png)

查看 @AutoConfigurationPackage 注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {
```

查看 AutoConfigurationPackages.Registrar.class 类，Debug运行

![img](/myres/201807/15/20180827182506.png)

可以看到会拿到启动类所在包名，它会将此包及下面所有子包里面的所有组件扫描到Spring容器中

# 三、使用Spring Initializer快速创建Spring Boot项目

1.IDEA使用Spring的项目创建向导快速创建一个Spring Boot项目，选择我们需要的模块；向导会联网创建Spring Boot项目

![img](/myres/201807/15/20180827190224.png)

默认生成的Spring Boot项目

- 主程序已经生成好了，我们只需要我们自己的逻辑
- resources文件夹中目录结构
- static：保存所有的静态资源； js css  images；
- templates：保存所有的模板页面；（Spring Boot默认jar包使用嵌入式的Tomcat，默认不支持JSP页面）；可以使用模板引擎（freemarker、thymeleaf）；
- application.properties：Spring Boot应用的配置文件；可以修改一些默认设置；

2.STS使用 Spring Starter Project快速创建项目

# 四、配置文件

SpringBoot使用一个全局的配置文件，配置文件名是固定的；

•application.properties

•application.yml

配置文件的作用：修改SpringBoot自动配置的默认值

## 1、ymal语法

k:(空格)v 表示一对键值对

以**空格**的缩进来控制层级关系；只要是左对齐的，都是同一个层级，属性和值都大小写敏感

```yaml
server:
    port: 8081
    path: /hello
```

### 1.1、字面量：普通的值（数字，字符串，布尔）

字符串默认不用加上单引号或者双引号

```yaml
# ""：双引号不会转义字符串里面的特殊字符，特殊字符会作为本身想表示的意思
# 输出；zhangsan 换行 lisi
name: "zhangsan \n lisi"

# ''：单引号会转义特殊字符，特殊字符最终只是一个普通的字符串数据
# 输出；zhangsan \n lisi
name: ‘zhangsan \n lisi’
```

### 1.2、对象、Map（属性和值）（键值对）

```yaml
friends:
    lastName: zhangsan
    age: 20

# 行内写法
friends: {lastName: zhangsan,age: 18}
```

### 1.3、数组（List、Set）：

```yaml
pets:
 - cat
 - dog
 - pig

# 行内写法
pets: [cat,dog,pig]
```

### 1.4、配置文件值注入

```yaml
# 配置文件
person:
    lastName: hello
    age: 18
    boss: false
    birth: 2017/12/12
    maps: {k1: v1,k2: 12}
    lists:
      - lisi
      - zhaoliu
    dog:
      name: 小狗
      age: 12
```

```java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个 javaBean 组件中
 * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定
 * prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 * 只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
```

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

## 2、properties语法

### 2.1、@Value获取值和@ConfigurationProperties获取值比较

|            |@ConfigurationProperties|@Value|
| --- | --- | --- |
|功能|批量注入配置文件中的属性|一个个指定|
|松散绑定（松散语法）|支持|不支持|
|SpEL|不支持|支持|
|JSR303数据校验|支持|不支持|
|复杂类型封装|支持|不支持|

只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value，专门编写了一个javaBean来和配置文件进行映射，就使用@ConfigurationProperties

```java
# 配置文件注入值数据校验
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {
    //lastName必须是邮箱格式
    @Email
    //@Value("${person.last-name}")
    private String lastName;
    //@Value("#{11*2}")
    private Integer age;
    //@Value("true")
    private Boolean boss;
```

### 2.2、@PropertySource、@ImportResource、@Bean

#### 2.2.1、@**PropertySource**：加载指定的配置文件

```java
/**
 * @ConfigurationProperties：将本类中的所有属性和 全局配置文件 中相关的配置进行绑定
 * prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 * 只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能；
 */
@Component
@ConfigurationProperties(prefix = "person")
@PropertySource(value = {"classpath:person.properties"})
public class Person {
    //@Value("${person.last-name}")
    private String lastName;
    //@Value("#{11*2}")
    private Integer age;
    //@Value("true")
    private Boolean boss;
```

#### 2.2.2、@**ImportResource**：导入Spring的xml配置文件

想让Spring的xml配置文件生效，在一个配置类上标注@**ImportResource**

```java
# 导入Spring的xml配置文件让其生效
@ImportResource(locations = {"classpath:beans.xml"})
```

```xml
<!--beans.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="helloService" class="com.atguigu.springboot.service.HelloService"></bean>
</beans>
```

#### 2.2.2、@**Bean**：导入JavaBean配置

```java
// @Configuration：指明当前类是一个配置类
@Configuration
public class MyAppConfig {
    //将方法的返回值添加到容器中，容器中这个组件默认的id就是方法名
    @Bean
    public HelloService helloService02(){
        System.out.println("配置类@Bean给容器中添加组件了...");
        return new HelloService();
    }
}
```

### 2.3、配置文件占位符

#### 2.3.1、随机数

```java
${random.value}
${random.int}
${random.long}
${random.int(10)}
${random.int[1024,65536]}
```

#### 2.3.2、占位符获取之前配置的值，如果没有可以是用:指定默认值

```properties
person.last-name=张三${random.uuid}
person.age=${random.int}
person.birth=2017/12/15
person.boss=false
person.dog.name=${person.hello:hello}_dog
person.dog.age=15
```

## 3、多配置环境切换

### 3.1、多Profile文件

主配置文件名可以是 application-{profile}.properties/yml，默认使用application.properties的配置

在application.properties配置文件中来指定环境

spring.profiles.active=dev

### 3.2、yml支持多文档块方式

```yml

server:
  port: 8081
spring:
  profiles:
    active: prod #指定使用哪个环境

---
server:
  port: 8083
spring:
  profiles: dev #指定属于哪个环境

---
server:
  port: 8084
spring:
  profiles: prod #指定属于哪个环境
```

### 3.3、其他方式激活指定profile

1、命令行，可以直接在测试的时候，配置传入命令行参数

java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev

2、虚拟机参数 （VM options）

-Dspring.profiles.active=dev

## 4、配置文件加载位置

### 4.1、内部配置加载顺序

springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件

–file:./config/

–file:./

–classpath:/config/

–classpath:/

优先级由高到底，高优先级的配置会覆盖低优先级的配置；

SpringBoot会从这四个位置全部加载主配置文件；**互补配置**

项目打包好以后，我们可以使用**命令行参数**的形式，启动项目的时候来指定配置文件的新位置，指定配置文件和默认加载的这些配置文件共同起作用形成**互补配置**

```cmd
java -jar xxxx.jar --spring.config.location=G:/application.properties
```

### 4.2、外部配置加载顺序

SpringBoot也可以从以下位置加载配置； 优先级从高到低，高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置

1.命令行参数，所有的配置都可以在命令行上进行指定，多个配置用空格分开； --配置项=值

java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc

2.来自java:comp/env的JNDI属性

3.Java系统属性（System.getProperties()）

4.操作系统环境变量

5.RandomValuePropertySource配置的random.*属性值

（6、7、8、9）总结为：由jar包外向jar包内进行寻找，优先加载带profile，再来加载不带profile

6.jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件

7.jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件

8.jar包外部的application.properties或application.yml(不带spring.profile)配置文件

9.jar包内部的application.properties或application.yml(不带spring.profile)配置文件

10.@Configuration注解类上的@PropertySource

11.通过SpringApplication.setDefaultProperties指定的默认属性

所有支持的配置加载来源：[参考官方文档](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/reference/htmlsingle/#boot-features-external-config)

配置文件可以配置的选项：[配置文件能配置的属性参照](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/reference/htmlsingle/#common-application-properties)

# 五、自动配置原理浅析

## 5.1、上面分析时说了在启动时会导入许多自动配置类，这里以HttpEncodingAutoConfiguration（Http编码自动配置）来分析，看干了什么

```java
@Configuration //表示是一个配置类，和以前编写的配置文件一样，也可以给容器中添加组件
@EnableConfigurationProperties(HttpEncodingProperties.class) //启动指定类的ConfigurationProperties功能，将配置文件中对应的值和HttpEncodingProperties绑定起来；并把HttpEncodingProperties加入到ioc容器中
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET) //Spring底层@Conditional注解，根据不同的条件来确定是否添加到容器中，判断当前应用是否是web应用，如果是，当前配置类生效
@ConditionalOnClass(CharacterEncodingFilter.class) //判断当前项目有没有这个类CharacterEncodingFilter，它是SpringMVC中进行乱码解决的过滤器
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true) //判断配置文件中是否存在某个配置  spring.http.encoding.enabled；如果不存在，判断也是成立的，即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的；
public class HttpEncodingAutoConfiguration {

    //SpringBoot的配置文件的映射类
    private final HttpEncodingProperties properties;

    //只有一个有参构造器的情况下，参数的值就会从容器中拿
    public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
        this.properties = properties;
    }

    @Bean  //给容器中添加一个组件，这个组件的某些值需要从properties中获取
    @ConditionalOnMissingBean //判断容器是否没有这个组件
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());
        filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
        filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
        return filter;
    }

    @Bean
    public LocaleCharsetMappingsCustomizer localeCharsetMappingsCustomizer() {
        return new LocaleCharsetMappingsCustomizer(this.properties);
    }

    private static class LocaleCharsetMappingsCustomizer implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory>, Ordered {

        private final HttpEncodingProperties properties;

        LocaleCharsetMappingsCustomizer(HttpEncodingProperties properties) {
            this.properties = properties;
        }

        @Override
        public void customize(ConfigurableServletWebServerFactory factory) {
            if (this.properties.getMapping() != null) {
                factory.setLocaleCharsetMappings(this.properties.getMapping());
            }
        }

        @Override
        public int getOrder() {
            return 0;
        }

    }

}
```

```java
@ConfigurationProperties(prefix = "spring.http.encoding") //从配置文件中获取指定的值和bean的属性进行绑定
public class HttpEncodingProperties {

    public static final Charset DEFAULT_CHARSET = StandardCharsets.UTF_8;
```

## 5.2、总结：

1）、SpringBoot启动会加载大量的自动配置类

2）、我们看我们需要的功能有没有SpringBoot默认写好的自动配置类

3）、我们再来看这个自动配置类中到底配置了哪些组件，（只要我们要用的组件有，我们就不需要再来配置了）

4）、给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这些属性的值，xxxxAutoConfigurartion：自动配置类，给容器中添加组件，xxxxProperties:封装配置文件中相关属性

## 5.3、附：@Conditional派生注解，作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效

| @Conditional扩展注解             | 作用（判断是否满足当前指定条件）|
| ------------------------------- | -----------------------------|
| @ConditionalOnJava              | 系统的java版本是否符合要求     |
| @ConditionalOnBean              | 容器中存在指定Bean；          |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean；        |
| @ConditionalOnExpression        | 满足SpEL表达式指定            |
| @ConditionalOnClass             | 系统中有指定的类              |
| @ConditionalOnMissingClass      | 系统中没有指定的类            |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值  |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件    |
| @ConditionalOnWebApplication    | 当前是web环境                 |
| @ConditionalOnNotWebApplication | 当前不是web环境               |
| @ConditionalOnJndi              | JNDI存在指定项                |

## 5.4、查看生效的自动配置类

```yaml
# application.yml
debug=true
```

```log
# 控制台输出日志
=========================
AUTO-CONFIGURATION REPORT（自动配置报告）
=========================

Positive matches:（自动配置类启用的）
-----------------
   DispatcherServletAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)
      - @ConditionalOnWebApplication (required) found StandardServletEnvironment (OnWebApplicationCondition)

Negative matches:（没有启动，没有匹配成功的自动配置类）
-----------------
   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 'org.apache.activemq.ActiveMQConnectionFactory' (OnClassCondition)

   AopAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'org.aspectj.lang.annotation.Aspect', 'org.aspectj.lang.reflect.Advice' (OnClassCondition)
```

---
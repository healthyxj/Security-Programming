# 一、介绍

## Spring

首先需要了解Spring框架。Spring是Java最广泛的框架，成功来源于理念，而不是技术。理念包括**IoC(Inversion of Control,控制反转)和AOP(Aspect Oriented Programming, 面向切面编程)**。因此，Spring是一个轻量级的DI/IoC和AOP容器的开源框架。

Spring的优势

* 低侵入、**低耦合**
* 声明式事务管理
* 便于继承其他框架(Mybatis)
* 降低了Java开发难度

### IoC控制反转

是设计思想，将原本在程序中手动创建对象的控制权，交给Spring框架来管理。若需要使用某个对象，只需要从Spring容器中获取需要使用的对象，不关系对象的创建过程，**把创建对象的控制权反转给了Spring框架**。

### AOP

面向切面编程，把功能分为核心业务(登录、增加数据、删除数据)功能和周边(性能统计、日志、事务管理)功能。

## Mybatis

Mybatis是基于Java的持久层框架。

持久层是将业务数据存储到磁盘，具备长期存储能力。缺点就是速度慢

使用Mybatis后，关注点只需要放在SQL语句上。

## SpringMVC

将传统的模型层划分为业务层(Service)和数据访问层(Dao, Data Access Object)



SSM框架是Spring MVC ，Spring和Mybatis框架的整合，是标准的MVC模式，将整个系统划分为View、Controller、Service、Dao(Data Access Object，数据访问对象)层，使用Spring MVC负责请求的转发和视图管理，Spring实现业务对象管理，Mybatis作为数据对象的持久化引擎。

## 各层的介绍

### 1.持久层(Mybatis)：

Dao层(mapper)

Dao层是负责数据持久层，与**数据库进行联络**的一些任务都封装于此。

设计过程：

* Dao层的设计首先需要设计Dao接口
* 在Spring(Resources中)的配置文件定义接口的实现类
* 在模块中间调用此接口来进行数据业务的处理，不关心此接口的具体实现类是哪个类
* Dao层的数据源配置，以及有关数据库连接的参数都在Spring的配置文件中进行配置。

例如，要定义一个根据用户Id查询相关信息的接口

1.在dao包中定义UserDao接口(需要导入entity中的用户类)，定义findUserById方法

~~~java
package cn.lm.dao;

import cn.lm.entity.User;

public interface UserDao {

    //根据id寻找对应的User
    User findUserById(int id);
}
~~~

2.在Spring**(Resources的mapper下的UserDao.xml)的配置文件**中定义接口的实现类

~~~xml
<!--设置为IUserDao接口方法提供sql语句配置-->
<mapper namespace="cn.lm.dao.UserDao">
    <select id="findUserById" resultType="cn.lm.entity.User" parameterType="int">
        SELECT * FROM user WHERE id = #{id}
    </select>
</mapper>
~~~

不过需要先在**Spring-Mybatis.xml中配置**，扫描Dao包，动态地实现Dao中的接口，注入到Spring容器中。

~~~xml
<!--配置扫描Dao接口包，动态实现Dao接口，注入到spring容器-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--注入sqlSessionFactory-->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <!--给出需要扫描Dao接口包-->
        <property name="basePackage" value="cn.lm.dao"/>
    </bean>
~~~

### 2.业务层(Spring):

Service层

Service层主要负责**业务模块的逻辑应用设计**。

设计过程：

* 首先设计接口，然后是对应的实现类
* 在Spring的配置文件中配置其实现的关联
* **Service层的业务实现，具体需要调用到已定义的Dao层的接口**
* 封装Service层的业务逻辑有利于通用的业务逻辑的独立性和重复利用性

例如，要实现之前Dao层的实现类，就需要用到Service层

1.首先在Service层定义接口和实现类。(同样**需要导入User类**)

接口名称是否一致存在疑问

~~~java
package cn.lm.service;

import cn.lm.entity.User;

public interface UserService {

    //根据id寻找对应的User
    public User findUserById(int id);
}
~~~

定义实现类(一般在接口后面加上Impl)

需要导入的包有用户类(要实现的类)、**Dao中的接口(作为Resource)**、org.springframework.stereotype.Service、javax.annotation.Resource。

~~~java
package cn.lm.service;

import cn.lm.entity.User;
import cn.lm.dao.UserDao;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;

/*
* UserService对应的实现类
*
* @author:@h**
* @create:2021-06-02
* */

@Service("userService")
public class UserServiceImpl implements UserService{

    @Resource
    private UserDao userDao;

    public User findUserById(int id){
        return userDao.findUserById(id);
    }

}
~~~

**返回的是用户类**

### 3.表现层(SpringMVC):

Controller层

Controller层负责具体**业务模块流程的控制**。

设计流程：

* 此层需要调用Service层的接口来控制业务流程
* 针对具体的业务流程，会有不同的控制器，我们具体的设计过程中可以将流程进行抽象归纳，设计出可以重复利用的子单元流程模块，这样不仅使程序结构变得清晰，也大大减少了代码量。



### 4.视图层(View)：

View层

View层与控制层结合紧密，主要负责前台jsp页面的开发



## 各层的关系

* **DAO层、Service层这两个层次都可以单独开发**，互相的耦合度很低，完全可以独立进行
* Controller，View层因为耦合度比较高，因而要结合在一起开发，但是也可以看作一个整体独立于前两个层进行开发。**在层与层之前我们只需要知道接口的定义，调用接口即可完成所需要的逻辑单元应用**
* Service层是建立在DAO层之上的，建立了DAO层后才可以建立Service层。而Service层又是在Controller层之下的，因而Service层应该既调用DAO层的接口，又要提供接口给Controller层的类来进行调用，它刚好处于一个中间层的位置。每个模型都有一个Service接口，每个接口分别封装各自的业务处理方法。**即开发时，按照Dao->Service->Controller的顺序**
  

## SSM的原理

1. 客户端发送请求到DispacherServlet（分发器）
2. 由DispacherServlet控制器查询HanderMapping，找到处理请求的Controller
3. Controller调用Service业务逻辑层处理后返回结果



![](img\SSM流程.png)



# 二、Maven

Maven是跨平台的项目管理工具。

在IDEA下创建默认的Maven项目，可以看到相同的结构

* 在pom.xml用于维护当前项目所依赖的jar包
* 所有的Java代码放在src/main/java下
* 所有的测试代码放在src/test/java

Maven能同一管理jar包

可以在创建时修改User settings file以及Local repository

2021-06-03 10:09

## 基本配置

### pom.xml

在pom.xml中声明依赖的jar包

~~~xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>cn.ssm</groupId>
  <artifactId>ssm</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>ssm Maven Webapp</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
    <!--spring version-->
    <spring.version>4.3.13.RELEASE</spring.version>
    <!--mybatis version-->
    <mybatis.version>3.4.5</mybatis.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
    </dependency>
    <!--java ee-->
    <dependency>
      <groupId>javax</groupId>
      <artifactId>javaee-api</artifactId>
      <version>7.0</version>
    </dependency>
    <!--实现slf4j接口并整合-->
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>1.1.1</version>
    </dependency>
    <!--json-->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.3.1</version>
    </dependency>
    <!--mysql-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.23</version>
    </dependency>

    <!--数据库连接池-->
    <dependency>
      <groupId>com.mchange</groupId>
      <artifactId>c3p0</artifactId>
      <version>0.9.5.2</version>
    </dependency>
    <!--Mybatis-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>${mybatis.version}</version>
    </dependency>
    <!--mybatis/spring整合包-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.1</version>
    </dependency>
    <!--Spring-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>4.3.13.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>4.3.13.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>4.3.13.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>4.3.13.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>4.3.13.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>4.3.13.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>4.3.13.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>4.3.13.RELEASE</version>
    </dependency>


  </dependencies>

  <build>
    <finalName>ssm</finalName>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <plugin>
          <groupId>org.mortbay.jetty</groupId>
          <artifactId>jetty-util</artifactId>
          <version>6.1.25</version>
          <configuration>
            <connectors>
              <connector implemantation="org.mortbay.jetty.nio.SelectChannelConnector">
                <port>8888</port>
                <maxIdleTime>30000</maxIdleTime>
              </connector>
            </connectors>
          </configuration>
        </plugin>
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
        <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_war_packaging -->
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.8.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.22.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-war-plugin</artifactId>
          <version>3.2.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>

~~~

### web.xml

在web.xml中声明编码过滤器并配置DispatcherServlet

~~~xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

  <!--编码过滤器-->
  <filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

  <!--配置DispatcherServlet-->
  <servlet>
    <servlet-name>SpringMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--配置spring MVC需要加载的问价-->
    <init-param>
      <param-name>contextConfiguration</param-name>
      <param-value>classpath:spring-*.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
    <async-supported>true</async-supported>
  </servlet>
  <servlet-mapping>
    <servlet-name>SpringMVC</servlet-name>
    <!--匹配所有请求-->
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
~~~

### spring-mybatis.xml

在spring-mybatis.xml中完成spring和mybatis的配置

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd " >

    <!--扫描service包下所使用注解的类型-->
    <context:component-scan base-package="cn.lm.service"/>

    <!--配置数据库相关参数properties的属性：${url}-->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!--数据库连接池-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="maxPoolSize" value="${c3p0.maxPoolSize}"/>
        <property name="minPoolSize" value="${c3p0.minPoolSize}"/>
        <property name="autoCommitOnClose" value="${c3p0.autoCommitOnClose}"/>
        <property name="checkoutTimeout" value="${c3p0.checkoutTimeout}"/>
        <property name="acquireRetryAttempts" value="${c3p0.acquireRetryAttempts}"/>
    </bean>

    <!--配置SqlSessionFaactory对象-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--注入数据库连接池-->
        <property name="dataSource" ref="dataSource"/>
        <!--扫描entity包，使用别名-->
        <property name="typeAliasesPackage" value="cn.lm.entity"/>
        <!--扫描sql配置文件：mapper需要的xml文件-->
        <property name="mapperLocations" value="classpath:mapper/*.xml"/>
    </bean>

    <!--配置扫描Dao接口包，动态实现Dao接口，注入到spring容器-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--注入sqlSessionFactory-->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <!--给出需要扫描Dao接口包-->
        <property name="basePackage" value="cn.lm.dao"/>
    </bean>

    <!--配置事务处理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注入数据库连接池-->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--配置基于注解的声明式事务-->
    <tx:annotation-driven transaction-manager="transactionManager"/>
</beans>

~~~

### spring-mvc.xml

完成spring-mvc的配置

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd " >

    <!--扫描web相关的bean-->
    <context:component-scan base-package="cn.lm.controller"/>

    <!--开启SpringMVC注解模式-->
    <mvc:annotation-driven/>

    <!--静态资源默认servlet配置-->
    <mvc:default-servlet-handler/>

    <!--配置jsp显示ViewResolver-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>

    </bean>

</beans>
~~~

### jdbc.properties

在jdbc.properties中配置c3p0数据库连接池

~~~properties
jdbc.driver = com.mysql.cj.jdbc.Driver

#数据库地址
jdbc.url=jdbc:mysql://localhost:3306/ssm?useUnicode=true&characterEncoding=utf8

#用户名
jdbc.username=root

#密码
jdbc.password=123456HXJ.com

#最大连接数
c3p0.maxPoolSize=30

#最小连接数
c3p0.minPoolSize=10

#关闭后连接不自动commit
c3p0.autoCommitOnClose=false

#获取连接超时时间
c3p0.checkoutTimeout=10000

#当获取连接失败重试次数
c3p0.acquireRetryAttempts=2
~~~

### logback.xml

在logback.xml中完成日志输出的相关配置

~~~xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration debug="true">
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="STDOUT"/>
    </root>
</configuration>

~~~

## 测试SSM框架

在MySQL中创建好表

### entity包

创建User类

~~~java
package cn.lm.entity;

/*
* User 实体类
*
* @author: @h**
* @create:2021-06-02
* */

public class User {
    private int id;
    private String username;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}
~~~

### dao包

在dao包下创建UserDao接口

~~~java
package cn.lm.dao;

import cn.lm.entity.User;

public interface UserDao {

    //根据id寻找对应的User
    User findUserById(int id);
}
~~~

### 创建映射

在resources/mapper下创建UserDao.xml映射文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--设置为IUserDao接口方法提供sql语句配置-->
<mapper namespace="cn.lm.dao.UserDao">
    <select id="findUserById" resultType="cn.lm.entity.User" parameterType="int">
        SELECT * FROM user WHERE id = #{id}
    </select>
</mapper>
~~~

### 测试Dao

每编写好一个Dao，都需要测试。在Test/java/UserDaoTest创建测试类

~~~java
import cn.lm.dao.UserDao;
import cn.lm.entity.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

/*
* UserDao的测试类
*
* @author:@h**
* @create:2021-06-02
* */

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath:spring-mybatis.xml"})
public class UserDaoTest {

    @Autowired
    private UserDao userDao;

    @Test
    public void testFindUserById(){
        int id = 1;
        User user = userDao.findUserById(id);
        System.out.println(user.getId() + ":" + user.getUsername());
    }
}
~~~

### service包

测试成功后，就可以编写Service接口

~~~java
package cn.lm.service;

import cn.lm.entity.User;

public interface UserService {

    //根据id寻找对应的User
    public User findUserById(int id);
}
~~~

和对应的实现类

~~~java
package cn.lm.service;

import cn.lm.entity.User;
import cn.lm.dao.UserDao;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;

/*
* UserService对应的实现类
*
* @author:@h**
* @create:2021-06-02
* */

@Service("userService")
public class UserServiceImpl implements UserService{

    @Resource
    private UserDao userDao;

    public User findUserById(int id){
        return userDao.findUserById(id);
    }

}
~~~

### controller包

创建UserController类

~~~java
package cn.lm.controller;

import cn.lm.entity.User;
import cn.lm.service.UserService;
import org.springframework.ui.Model;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import javax.annotation.Resource;

/*
* User的控制类
*
* @author:@h**
* @create:2021-06-02
* */

@Controller
@RequestMapping("")
public class UserController {

    @Resource
    private UserService userService;

    @RequestMapping("/findUser")
    public String findUser(Model model){
        int id = 1;
        User user = this.userService.findUserById(id);
        model.addAttribute("user",user);
        return "index";
    }
}
~~~

最后设置以下index.jsp文件



# 三、前端



按钮实现页面的跳转

~~~jsp
<a href="http://www.baidu.com/">
    <button>百度</button>
</a>

<!--or使用JavaScript函数-->

<script>
function jump(){
 window.location.href="http://www.baidu.com/";
}
</script>
<input type="button" value="百度" onclick=javascrtpt:jump() />
// 或者<input type="button" value="百度" onclick="jump()" />
// 或者<button onclick="jump()">百度</button>
~~~



居中

~~~jsp
 <div style="text-align:center">
   <button>按钮居中</button>                     
 </div>

<!--方式二-->

<div>   
   <button  style="display:block;margin:0 auto">按钮居中</button>
</div>

~~~



使用外部css标签

~~~html
    <link rel="stylesheet" type="text/css" href="${pageContext.request.contextPath}/static/plugins/bootstrap-3.4.1-dist/css/bootstrap.css">
    <link rel="stylesheet" type="text/css" href="${pageContext.request.contextPath}/static/plugins/bootstrap-3.4.1-dist/css/bootstrap.min.css">
    <link rel="stylesheet" type="text/css" href="${pageContext.request.contextPath}/static/plugins/bootstrap-3.4.1-dist/css/Mycss.css">
~~~



css返回的text/html



css返回到中间

~~~
https://jingyan.baidu.com/article/e2284b2b67b5f1e2e6118d22.html
~~~



## 增删改查

查询

~~~jsp
<!-- 			查询记录  -->
			 <form id="search_form" action="${pageContext.request.contextPath }/admin/items/allList.do" method="post">
	           <div class="row">
	                 <div class="col-md-2">
	                       <input type="text" id="search_item_name" class="form-control input-sm" placeholder="输入用户名" name="username" value="">
	                 </div>                
	                 <div class="col-md-9">
	                    
	                    <button id="search_btn" type="submit" class="btn btn-primary"><span class="glyphicon glyphicon-search"></span>查询</button>&nbsp;&nbsp;
	                 </div>
	            </div> 
             </form>

~~~



## 修改

~~~jsp
 <!-- 			记录修改弹出层 -->
			<div class="modal fade" id="editLayer" tabindex="-1" role="dialog" aria-labelledby="myModalLabel">
			<div class="modal-dialog">
			        <div class="modal-content">
			          <div class="modal-header">
			          
			            <button data-dismiss="modal" class="close" type="button"><span aria-hidden="true">×</span><span class="sr-only">Close</span></button>
			            <h4 class="modal-title">修改记录</h4>
			            
			          </div>
			            <div class="modal-body">
				            <form id="edit_items_form" class="form-horizontal">
				                <input type="hidden" id="edit_item_id" class="form-control" name="id"> <br>
								<input type="text" id="edit_item_username" class="form-control" placeholder="用户名" name="username"> <br>
	                            <input type="password" id="edit_item_password" class="form-control" placeholder="密码" name="password">
	                        </form>
				        </div>
			          
			          
			          <div class="modal-footer">
			            <button data-dismiss="modal" class="btn btn-default" type="button">关闭</button>
			            <button class="btn btn-primary" type="button" onclick="updateItem()">确认修改</button>
			          </div>
			          
			        </div>
			      </div>
			</div>

~~~



## 增加

[SpringBoot - 获取POST请求参数详解（附样例：表单数据、json、数组、对象） (hangge.com)](https://www.hangge.com/blog/cache/detail_2485.html)

~~~jsp

<!-- 			记录添加弹出层 -->
			<button type="button" class="btn btn-primary" data-toggle="modal" data-target="#addLayer"> <span class="glyphicon glyphicon-plus"></span>添加记录 </button>
			<div class="modal fade" id="addLayer" tabindex="-1" role="dialog" aria-labelledby="myModalLabel">
			<div class="modal-dialog">
			        <div class="modal-content">
			          <div class="modal-header">
			          
			            <button data-dismiss="modal" class="close" type="button"><span aria-hidden="true">×</span><span class="sr-only">Close</span></button>
			            <h4 class="modal-title">添加记录</h4>
			            
			          </div>
			            <div class="modal-body">
				            <form id="add_items_form" class="form-horizontal">
				                <input type="hidden" class="form-control" name="id"> <br>
								<input type="text" class="form-control" placeholder="用户名" name="username"> <br>
	                            <input type="password" class="form-control" placeholder="密码" name="password">
	                        </form>
				        </div>
			          
			          
			          <div class="modal-footer">
			            <button data-dismiss="modal" class="btn btn-default" type="button">关闭</button>
			            <button class="btn btn-primary" type="button" onclick="addItem()">确认添加</button>
			          </div>
			          
			        </div>
			      </div>
			</div>

~~~



~~~html
<button type="button" style="margin-left:15px" class="btn btn-primary btn-sm" id="btn_cxjxb"> 查询教学班 </button>
~~~



# 四、数据库



~~~sql
--显示数据库ssm的结构
show ssm;

--显示表user的结构
desc user;

--查看书的内容
select * from book;

--删除


~~~

auto_increment

~~~sql
--不需要用户自己输入id

    <div class="form-group">
        <label for="id">
            id：
        </label>
        <input type="text" class="form-control" id="id" name="id" placeholder="请输入">
    </div>
~~~



~~~sql
--管理员添加书籍也不需要id
    <div class="form-group">
        <label for="id">
            id：
        </label>
        <input type="text" class="form-control" id="id" name="id" placeholder="请输入名称">
    </div>
~~~

~~~jsp
    <a href="${pageContext.request.contextPath}/book/search">
        <button type="button" class="btn btn-primary" onclick="">搜索</button>
    </a>
~~~



# 五、可以完善的点

~~~
检验用户名是否重复

拦截器验证权限
~~~


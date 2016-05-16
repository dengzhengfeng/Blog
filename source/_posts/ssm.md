title: 搭建一个spring、springMVC、mybatis工程
date: 2016-05-03 11:37:07
tags: ["spring","springMVC","mybatis"]
categories: ["java"]
---
好久没有从头开始搭建开发代码了。今天做个小工具，居然发现mybatis的配置和以前不一样了，先记录一下吧。[示例代码](https://github.com/dengzhengfeng/ssm/)
<h2 id="context">准备环境</h2>
>gradle+maven+idea+postgresql
完成在数据库中查询一条数据并展示

<h2 id="git">初始化git</h2>
`git init`
<h2 id="gradle">初始化gradle</h2>
`gradle init --type java-library`
<h2 id="folder">创建目录</h2>
>最终的目录结构如下

![目录结构](/img/ssm/folder.png)
src
——main
————java.org.ssm
——————controller
——————dao
——————pojo
——————service
——webapp
————WEB-INF
——————config
——————spring
————————appServlet
——————views
<h2 id="folder">配置依赖</h2>
>增加war，gretty插件

``` +json
apply plugin: 'war'
apply from: 'https://raw.github.com/akhikhl/gretty/master/pluginScripts/gretty.plugin'
```
>配置spring、springMVC、mybatis依赖

``` +json
dependencies {
    compile 'org.springframework:spring-webmvc:3.1.1.RELEASE'
    compile 'org.springframework:spring-jdbc:3.1.1.RELEASE'
    compile 'org.mybatis:mybatis-spring:1.2.5'
    compile 'org.mybatis:mybatis:3.3.1'
    compile 'commons-dbcp:commons-dbcp:1.2.2'
    compile 'postgresql:postgresql:9.1-901-1.jdbc4'
    compile 'org.slf4j:slf4j-api:1.7.14'

    testCompile 'junit:junit:4.12'
}
```
>配置gretty

``` +json
gretty {
    port = 8080
    contextPath = '/'
    servletContainer = 'jetty9'
}
```
由于gradle不会打包mybatis的xml文件，我们要手动增加一个任务
``` +json
task copyTask(type: Copy) {
    copy{
        from ('src/main/java/org/ssm/dao'){
            include '**/*.xml'
        }
        into 'build/classes/main/org/ssm/dao'
    }
}
```
<h2 id="xml">配置web.xml</h2>
>在web.xml中，我们要将springMVC的listener和Servlets配置进来，同时部分spring的xml也要配置进来

``` +xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
    <!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            /WEB-INF/spring/root-context.xml,
            /WEB-INF/spring/mybatis-config.xml
        </param-value>
    </context-param>
    <!-- Creates the Spring Container shared by all Servlets and Filters -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <!-- Processes application requests -->
    <servlet>
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>appServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```
springMVC的配置spring/appServlet/servlet-context.xml
``` +xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/mvc"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
	<!-- DispatcherServlet Context: defines this servlet's request-processing infrastructure -->
	<!-- Enables the Spring MVC @Controller programming model -->
	<annotation-driven />
	<!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources directory -->
	<resources mapping="/resources/**" location="/resources/" />
	<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
	<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<beans:property name="prefix" value="/WEB-INF/views/" />
		<beans:property name="suffix" value=".jsp" />
	</beans:bean>
	<context:component-scan base-package="org.ssm.controller" />
</beans:beans>
```
<h2 id="mybatis-config">配置mybatis</h2>
>配置config/jdbc.properties（使用postgresql）

``` +properties
jdbc.driver=org.postgresql.Driver
jdbc.url=jdbc:postgresql://192.168.1.128:5432/testdb
jdbc.username=dzf
jdbc.password=123456
```
配置
spring/mybatis-config.xml
``` +xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="location">
            <value>/WEB-INF/config/jdbc.properties</value>
        </property>
    </bean>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
          destroy-method="close">
        <property name="driverClassName" value="${jdbc.driver}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
    </bean>
    
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="org.ssm.dao" />
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>
</beans>
```
<h2 id="mybatis">mybatis代码编写</h2>
>编写实体类org.ssm.pojo.UserDoc.java

``` +java
public class UserDoc {
	private String userName;

	public String getUserName() {
		return userName;
	}

	public void setUserName(String userName) {
		this.userName = userName;
	}
}
```
>编写Mapper org.ssm.dao.UserMapper.java

``` +java
public interface UserMapper {
	public UserDoc getUser();
}
```
>编写sqlmap UserMapper.xml

``` +xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
	  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
	  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.ssm.dao.UserMapper">
	<select id="getUser" resultType="org.ssm.pojo.UserDoc">
		SELECT "userName" FROM "UserDoc" limit 1
	</select>
</mapper>
```
>编写服务类 org.ssm.service.UserService

``` +java
public class UserService {
    @Autowired
    private UserMapper userMapper;

    public UserDoc getUserDoc(){
        return userMapper.getUser();
    }
}
```
将服务类注册进spring
``` +xml
<bean id="UserService" class="org.ssm.service.UserService"/>
```

<h2 id="springMVC">springMVC 代码编写</h2>
>编写Controller

``` +java
@Controller
public class HomeController {

	private static final Logger logger = LoggerFactory.getLogger(HomeController.class);

	@Autowired
	private UserService userService;

	@RequestMapping(value = "/", method = RequestMethod.GET)
	public String home(Locale locale, Model model) {
		UserDoc doc = userService.getUserDoc();

		model.addAttribute("username", doc.getUserName());

		return "home";
	}
}
```
>编写view home.jsp

``` +jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ page session="false" %>
<%@page pageEncoding="UTF-8" contentType="text/html; charset=UTF-8" %>
<html>
<head>
	<title>欢迎${username}</title>
</head>
<body>
<h1>
	你好 ${username}!
</h1>

<P>  欢迎${username}. </P>
</body>
</html>
```

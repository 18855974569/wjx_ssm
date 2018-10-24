SSM配置介绍
目录
一. 项目结构图	2
二. 配置文件介绍	3
2.1 配置文件	3
a. applicationContext.xml	3
b. mybatis-config.xml	4
c. spring-mvc-servlet.xml	5
d. spring-redis.xml	6
2.2 properties文件	7
a. jdbc.properties	7
b. log4j.properties	8
c. redis.properties	9
2.3 web.xml配置	9
2.4 pom.xml配置文件	11
2.5 mapper.xml的配置	16
2.6 redis缓存中间类(Spring注入redis连接池)	17
2.7 RedisCache缓存类	17
2.8 userList.jsp代码	21
三. 测试ssm配置是否成功	27
1. 启动tomcat程序	27
2. 在浏览器输入地址	27
3. 验证redis二级缓存是否生效	28
四. 打包部署到服务器上面发布出去	32
1. Maven项目打包	32
2. 部署到服务器	33
五. 注意事项	34

一.项目结构图


二.配置文件介绍
2.1 配置文件
a.applicationContext.xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-4.3.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-4.3.xsd">

	<!-- 配置扫描@Service注解 -->
	<context:component-scan base-package="com.wjx" />

	<!-- 读取properties文件 -->
	<bean id="propertyConfigurer"
		class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
		<property name="locations">
			<list>
				<value>classpath:properties/jdbc.properties</value>
				<value>classpath:properties/redis.properties</value>
			</list>
		</property>
	</bean>

	<!-- 数据库连接池 -->
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
		destroy-method="close">
		<property name="driverClassName" value="${driver}" />
		<property name="url" value="${url}" />
		<property name="username" value="${username}" />
		<property name="password" value="${password}" />
		<!-- 初始化连接大小 -->
		<property name="initialSize" value="${initialSize}"></property>
		<!-- 连接池最大数量 -->
		<property name="maxActive" value="${maxActive}"></property>
		<!-- 连接池最大空闲 -->
		<property name="maxIdle" value="${maxIdle}"></property>
		<!-- 连接池最小空闲 -->
		<property name="minIdle" value="${minIdle}"></property>
		<!-- 获取连接最大等待时间 -->
		<property name="maxWait" value="${maxWait}"></property>
	</bean>

	<!-- Mybatis的工厂bean -->
	<bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<!-- 核心配置文件的位置 -->
		<property name="configLocation" value="classpath:config/mybatis-config.xml" />
	</bean>

	<!-- Mapper动态代理 -->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="com.wjx.mapper" />
	</bean>

	<!-- 注解事务 -->
	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource" />
	</bean>

	<!-- 导入redis配置 -->
	<import resource="spring-redis.xml" />
	
</beans>

b.mybatis-config.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<settings>
		<!-- 日志开启 -->
		<setting name="logImpl" value="LOG4J" />
		<!-- 二级缓存开启 -->
		<setting name="cacheEnabled" value="true" />
		<setting name="lazyLoadingEnabled" value="false" />
		<setting name="aggressiveLazyLoading" value="true" />
	</settings>
	<typeAliases>
		<typeAlias alias="User" type="com.wjx.pojo.User" />
	</typeAliases>
</configuration>




c.spring-mvc-servlet.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:beans="http://www.springframework.org/schema/beans"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans-4.3.xsd 
      http://www.springframework.org/schema/context
      http://www.springframework.org/schema/context/spring-context-4.3.xsd 
      http://www.springframework.org/schema/mvc
      http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd">


	<!-- 配置扫描，controller包 -->
	<context:component-scan base-package="com.wjx.**.controller"
		use-default-filters="false">
		<context:include-filter type="annotation"
			expression="org.springframework.stereotype.Controller" />
	</context:component-scan>
	
	<mvc:annotation-driven />
	<!-- 配置视图解析器 -->
	<bean
		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix">
			<value>/WEB-INF/jsp/</value>
		</property>
		<property name="suffix">
			<value>.jsp</value>
		</property>
	</bean>
	<!-- 配置静态资源 -->
	<mvc:annotation-driven />
	<mvc:resources mapping="/statics/**" location="/statics/"></mvc:resources>
	<mvc:resources mapping="/layui/**" location="/layui/"></mvc:resources>
</beans>

d.spring-redis.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:util="http://www.springframework.org/schema/util"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="
             http://www.springframework.org/schema/beans
             http://www.springframework.org/schema/beans/spring-beans.xsd
             http://www.springframework.org/schema/util
             http://www.springframework.org/schema/util/spring-util.xsd">

	<!-- redis数据源 -->
	<bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
		<property name="maxIdle" value="${redis.maxIdle}" />
		<property name="maxTotal" value="${redis.maxTotal}" />
		<property name="maxWaitMillis" value="${redis.maxWaitMillis}" />
		<property name="testOnBorrow" value="${redis.testOnBorrow}" />
	</bean>

	<!-- Spring-redis连接池管理工厂 -->
	<bean id="jedisConnectionFactory"
		class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
		p:host-name="${redis.host}" p:port="${redis.port}" p:pool-config-ref="poolConfig" />

	<!-- redis模板 -->
	<bean id="jedisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
		<property name="connectionFactory" ref="jedisConnectionFactory"></property>
		<property name="keySerializer">
			<bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />
		</property>
		<property name="valueSerializer">
			<bean class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
		</property>
	</bean>

	<!-- 使用中间类解决RedisCache.jedisConnectionFactory的静态注入，从而使MyBatis实现第三方缓存 -->
	<bean id="redisCacheTransfer" class="com.redis.RedisCacheTransfer">
		<property name="jedisConnectionFactory" ref="jedisConnectionFactory" />
	</bean>

</beans>










2.2 properties文件

a.jdbc.properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://114.116.24.32:3306/wjx?useUnicode=true&characterEncoding=utf-8
username=wjx
password=123
initialSize=0
maxActive=20
maxIdle=20
minIdle=1
maxWait=60000

b.log4j.properties
#定义LOG输出级别
log4j.rootLogger=INFO,Console,File
#定义日志输出目的地为控制台 
log4j.appender.Console=org.apache.log4j.ConsoleAppender 
log4j.appender.Console.Target=System.out 
#可以灵活的指定日志输出格式，下面一行是指定具体的格式 
log4j.appender.Console.layout=org.apache.log4j.PatternLayout 
log4j.appender.Console.layout.ConversionPattern=[%c]-%m%n
#mybatis显示SQL语句日志配置  
#log4j.logger.org.mybatis=DEBUG
log4j.logger.net.cxp.blog.dao=DEBUG
#文件大小到达指定尺寸的时候产生一个新的文件
log4j.appender.File=org.apache.log4j.RollingFileAppender
#指定输出目录
#log4j.appender.File.File=logs/ssm.log
#定义文件最大大小
log4j.appender.File.MaxFileSize=10MB
#输出所有日志，如果换成DEBUG表示输出DEBUG以上级别日志
log4j.appender.File.Threshold=ALL
log4j.appender.File.layout=org.apache.log4j.PatternLayout
log4j.appender.File.layout.ConversionPattern=[%p][%d{yyyy-MM-dd HH\:mm|\:ss}][%c]%m%n
#mybatis显示SQL语句日志配置  
#log4j.logger.com.wjx=DEBUG       com.wjx是mapper文件包
log4j.logger.java.sql.Connection=debug
log4j.logger.java.sql.Statement=debug
log4j.logger.java.sql.PreparedStatement=debug 
log4j.logger.com.wjx=DEBUG

c.redis.properties

#redis.host=127.0.0.1
redis.host=114.116.24.32
redis.port=6379
redis.maxIdle=50
redis.maxTotal=100
redis.maxWaitMillis=3000
redis.testOnBorrow=true
redis.timeout=5000







2.3 web.xml配置
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
    http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

	<!-- 加载Spring容器配置 -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	<!-- 设置Spring容器加载所有的配置文件的路径 ,默认加载的是web-inf下的applicationcontext.xml文件，此处是自定义 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:config/applicationContext.xml</param-value>
	</context-param>

	<!-- 配置SpringMVC核心控制器 -->
	<servlet>
		<servlet-name>springMVC</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<!-- 默认加载的是wen-inf下的XXX-servlet.xml（springMVC-servlet.xml）文件，此处自定义 -->
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:config/spring-mvc-servlet.xml</param-value>
		</init-param>
		<!-- 启动加载一次 -->
		<load-on-startup>1</load-on-startup>
	</servlet>

	<!--为DispatcherServlet建立映射 -->
	<servlet-mapping>
		<servlet-name>springMVC</servlet-name>
		<!-- 此处可以可以配置成*.do -->
		<url-pattern>/</url-pattern>
	</servlet-mapping>

	<!-- 防止Spring内存溢出监听器 -->
	<listener>
		<listener-class>org.springframework.web.util.IntrospectorCleanupListener</listener-class>
	</listener>

	<!-- 解决工程编码过滤器 -->
	<filter>
		<filter-name>encodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
		<init-param>
			<param-name>forceEncoding</param-name>
			<param-value>true</param-value>
		</init-param>
	</filter>

	<filter-mapping>
		<filter-name>encodingFilter</filter-name>
		<url-pattern>/*</u<build>
	<finalName>MavenProject</finalName>
</build>

rl-pattern>
	</filter-mapping>

	<!-- log4j日志记录 -->
	<context-param>
		<!-- 日志配置路径 -->
		<param-name>log4jConfigLocation</param-name>
		<param-value>classpath:properties/log4j.properties
    </param-value>
	</context-param>
	<context-param>
		<!-- 日志页面刷新间隔 -->
		<param-name>log4jRefreshInterval</param-name>
		<param-value>6000</param-value>
	</context-param>
	<listener>
		<listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
	</listener>


	<welcome-file-list>
		<welcome-file>index.jsp</welcome-file>
	</welcome-file-list>
</web-app>




2.4 pom.xml配置文件
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.wjx</groupId>
	<artifactId>MavenProject</artifactId>
	<packaging>war</packaging>
	<version>0.0.1-SNAPSHOT</version>
	<name>MavenProject Maven Webapp</name>
	<url>http://maven.apache.org</url>

     <!-- 版本号 -->
	<properties>
		<!-- spring版本号 -->
		<spring.version>4.0.2.RELEASE</spring.version>
		<!-- mybatis版本号 -->
		<mybatis.version>3.2.6</mybatis.version>
		<!-- log4j日志文件管理包版本 -->
		<slf4j.version>1.7.7</slf4j.version>
		<log4j.version>1.2.17</log4j.version>
		<!-- Servlet-API 版本号 -->
		<servlet.version>6.0.53</servlet.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>3.8.1</version>
			<scope>test</scope>
		</dependency>
		<!-- spring核心包 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-oxm</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-aop</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<!-- mybatis核心包 -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>${mybatis.version}</version>
		</dependency>
		<!-- mybatis/spring包 -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis-spring</artifactId>
			<version>1.2.2</version>
		</dependency>

		<!-- 导入Mysql数据库链接jar包 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.30</version>
		</dependency>

		<!-- 导入dbcp的jar包，用来在applicationContext.xml中配置数据库 -->
		<dependency>
			<groupId>commons-dbcp</groupId>
			<artifactId>commons-dbcp</artifactId>
			<version>1.2.2</version>
		</dependency>
		<!-- JSTL标签类 -->
		<dependency>
			<groupId>jstl</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>
		<!-- java ee包（方便开发的时候用，部署的时候tomcat里有） -->
		<dependency>
			<groupId>org.apache.tomcat</groupId>
			<artifactId>servlet-api</artifactId>
			<version>${servlet.version}</version>
		</dependency>
		<!-- 日志文件管理包 -->
		<!-- log start -->
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>${log4j.version}</version>
		</dependency>


		<!-- json格式化对象，方便输出日志 -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.1.41</version>
		</dependency>

		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>${slf4j.version}</version>
		</dependency>

		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>${slf4j.version}</version>
		</dependency>

		<!-- 引入JSON -->
		<dependency>
			<groupId>org.codehaus.jackson</groupId>
			<artifactId>jackson-mapper-asl</artifactId>
			<version>1.9.13</version>
		</dependency>
		<!-- 上传组件包 -->
		<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
			<version>1.3.1</version>
		</dependency>
		<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
			<version>2.4</version>
		</dependency>
		<dependency>
			<groupId>commons-codec</groupId>
			<artifactId>commons-codec</artifactId>
			<version>1.9</version>
		</dependency>


		<!-- spring-redis实现 -->
		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-redis</artifactId>
			<version>1.6.2.RELEASE</version>
		</dependency> <!-- redis客户端jar -->
		<dependency>
			<groupId>redis.clients</groupId>
			<artifactId>jedis</artifactId>
			<version>2.8.0</version>
		</dependency> <!-- Ehcache实现,用于参考 -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis-ehcache</artifactId>
			<version>1.0.0</version>
		</dependency>
	</dependencies>


	<build>
		<finalName>MavenProject</finalName>
	</build>
</project>


2.5 mapper.xml的配置
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.wjx.mapper.UserMapper">

	<!-- 开启redis缓存 -->
	<cache eviction="LRU" type="com.redis.RedisCache" />

	<select id="queryUserList" parameterType="User" resultType="User">
		select * from users
	</select>

	<select id="queryUserById" parameterType="User" resultType="User">
		select * from users where id = #{id}
	</select>

	<insert id="addUser" parameterType="User">
		insert into users (name,age)
		values (#{name},#{age})
	</insert>

	<update id="updateUser" parameterType="User">

		<if test="id != null">
			update users
			<set>
				<if test="name != null">
					name = #{name},
				</if>
				<if test="age != null">
					age=#{age},
				</if>
			</set>
			where id = #{id}
		</if>
	</update>

	<delete id="deleteUser" parameterType="User">
		delete from users where id = #{id}
	</delete>
</mapper>

2.6 redis缓存中间类(Spring注入redis连接池)
package com.redis;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;

/**
 * 
 * @描述: 静态注入中间类
 * @author wjx
 * @date 2018-10-12
 */
public class RedisCacheTransfer {

	@Autowired
	public void setJedisConnectionFactory(JedisConnectionFactory jedisConnectionFactory) {
		System.out.println("数据源:"+jedisConnectionFactory);
		RedisCache.setJedisConnectionFactory(jedisConnectionFactory);
	}
}
2.7 RedisCache缓存类
package com.redis;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

import org.apache.ibatis.cache.Cache;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.data.redis.connection.jedis.JedisConnection;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.serializer.JdkSerializationRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializer;

import redis.clients.jedis.exceptions.JedisConnectionException;

/**
 * 
 * @描述: 使用第三方内存数据库Redis作为二级缓存
 * @版权: Copyright (c) 2016
 * @作者: wjx
 * @版本: 1.0
 * @创建日期:
 * @创建时间:
 */
public class RedisCache implements Cache {

    private static final Logger logger = LoggerFactory.getLogger(RedisCache.class);

    private static JedisConnectionFactory jedisConnectionFactory;

    private final String id;

    /**
     * The {@code ReadWriteLock}.
     */
    private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock();


	public RedisCache(final String id) {
        if (id == null) {
            throw new IllegalArgumentException("Cache instances require an ID");
        }
        logger.debug("MybatisRedisCache:id=" + id);
        this.id = id;
    }

    public void clear() {
        JedisConnection connection = null;
        try {
            connection = jedisConnectionFactory.getConnection();
            connection.flushDb();
            connection.flushAll();
        } catch (JedisConnectionException e) {
            e.printStackTrace();
        } finally {
            if (connection != null) {
                connection.close();
            }
        }
    }

    public String getId() {
        return this.id;
    }

    public Object getObject(Object key) {
        Object result = null;
        JedisConnection connection = null;
        try {
            connection = jedisConnectionFactory.getConnection();
            RedisSerializer<Object> serializer = new JdkSerializationRedisSerializer();
            result = serializer.deserialize(connection.get(serializer.serialize(key)));
        } catch (JedisConnectionException e) {
            e.printStackTrace();
        } finally {
            if (connection != null) {
                connection.close();
            }
        }
        return result;
    }

    public ReadWriteLock getReadWriteLock() {
        return this.readWriteLock;
    }

    public int getSize() {
        int result = 0;
        JedisConnection connection = null;
        try {
            connection = jedisConnectionFactory.getConnection();
            result = Integer.valueOf(connection.dbSize().toString());
        } catch (JedisConnectionException e) {
            e.printStackTrace();
        } finally {
            if (connection != null) {
                connection.close();
            }
        }
        return result;
    }

    public void putObject(Object key, Object value) {
        JedisConnection connection = null;
        try {
            connection = jedisConnectionFactory.getConnection();
            RedisSerializer<Object> serializer = new JdkSerializationRedisSerializer();
            connection.set(serializer.serialize(key), serializer.serialize(value));
        } catch (JedisConnectionException e) {
            e.printStackTrace();
        } finally {
            if (connection != null) {
                connection.close();
            }
        }
    }

    public Object removeObject(Object key) {
        JedisConnection connection = null;
        Object result = null;
        try {
            connection = jedisConnectionFactory.getConnection();
            RedisSerializer<Object> serializer = new JdkSerializationRedisSerializer();
            result = connection.expire(serializer.serialize(key), 0);
        } catch (JedisConnectionException e) {
            e.printStackTrace();
        } finally {
            if (connection != null) {
                connection.close();
            }
        }
        return result;
    }

    public static void setJedisConnectionFactory(JedisConnectionFactory jedisConnectionFactory) {
    	System.out.println("数据源:-----------------------------------------------------------"+jedisConnectionFactory);
        RedisCache.jedisConnectionFactory = jedisConnectionFactory;
    }
}
2.8 userList.jsp代码
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<link rel="stylesheet" href="layui/css/layui.css" media="all">
<link rel="stylesheet" href="http://cdn.bootcss.com/bootstrap/3.3.5/css/bootstrap.min.css">
<title>用户列表页面</title>
</head>
<body>
	<table class="layui-hide" id="test" lay-filter="test"></table>

	<div class="modal fade" id="addUserModel" tabindex="-1" role="dialog"
		aria-labelledby="myModalLabel">
		<div class="modal-dialog" role="document">
			<div class="modal-content">
				<div class="modal-header">
					<button type="button" class="close" data-dismiss="modal"
						aria-label="Close">
						<span aria-hidden="true">×</span>
					</button>
					<h4 class="modal-title" id="myModalLabel">新增用户</h4>
				</div>
				<div class="modal-body">
					<form id="form2">
						<table>
							<tr>
								<td>姓名：</td>
								<td><input id="add_name" type="text" name="name" class="layui-input" /></td>
							</tr>
							<tr>
								<td>地址：</td>
								<td><input id="add_adress" type="text" name="address" class="layui-input" /></td>
							</tr>
							<tr>
								<td>年龄：</td>
								<td><input id="add_age" type="number" name="age" class="layui-input" /></td>
							</tr>
						</table>
					</form>
				</div>
				<div class="modal-footer">
					<button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
					<button type="button" id="addSubmit" class="btn btn-primary">Save</button>
				</div>
			</div>
		</div>
	</div>
	
	<div class="modal fade" id="editUserModel" tabindex="-1" role="dialog"
		aria-labelledby="myModalLabel">
		<div class="modal-dialog" role="document">
			<div class="modal-content">
				<div class="modal-header">
					<button type="button" class="close" data-dismiss="modal"
						aria-label="Close">
						<span aria-hidden="true">×</span>
					</button>
					<h4 class="modal-title" id="myModalLabel">修改用户</h4>
				</div>
				<div class="modal-body">
					<form id="form1">
						<table>
							<tr>
								<td>id：</td>
								<td><input id="edit_id" type="text" name="id" class="layui-input" readonly="readonly" /></td>
							</tr>
							<tr>
								<td>姓名：</td>
								<td><input id="edit_name" type="text" name="name" class="layui-input" /></td>
							</tr>
							<tr>
								<td>地址：</td>
								<td><input id="edit_adress" type="text" name="address" class="layui-input" /></td>
							</tr>
							<tr>
								<td>年龄：</td>
								<td><input id="edit_age" type="number" name="age" class="layui-input" /></td>
							</tr>
						</table>
					</form>
				</div>
				<div class="modal-footer">
					<button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
					<button type="button" id="editSubmit" class="btn btn-primary">Save</button>
				</div>
			</div>
		</div>
	</div>

</body>
</html>


<script src="http://cdn.bootcss.com/jquery/1.11.3/jquery.min.js"></script>
 <script src="http://cdn.bootcss.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>

<script type="text/javascript" src="layui/layui.js"></script>
<!-- <script type="text/javascript" src="statics/js/jquery-3.3.1.min.js"></script> -->
<script type="text/html" id="toolbarDemo">
  <div class="layui-btn-container">
    <button class="layui-btn layui-btn-sm" lay-event="add">新增</button>
    <button class="layui-btn layui-btn-sm" lay-event="edit">修改</button>
    <button class="layui-btn layui-btn-sm" lay-event="delete">删除</button>
  </div>
</script>
<script type="text/javascript">
	var table;
	var layer;
	layui.use('table', function() {
		layer = layui.layer;
		table = layui.table;
		refresh();
		//头工具栏事件
		table.on('toolbar(test)', function(obj) {
			var checkStatus = table.checkStatus(obj.config.id);
			switch (obj.event) {
			case 'add':
				$('#addUserModel').modal('show');
				break;
			case 'edit':
				var data = checkStatus.data;
				if (data.length == 0) {
					layer.msg("请选择一行编辑");
					return;
				}
				if (data.length > 1) {
					layer.msg("只可以选择一行编辑");
					return;
				}
				$('#editUserModel').modal('show');
				
				$.ajax({
					url : "queryUserById?id=" + data[0].id,
					type : "GET",
					data : {},
					success : function(data) {
						$("#editUser").css("visibility", "visible");
						$("#edit_id").val(data.id);
						$("#edit_name").val(data.name);
						$("#edit_age").val(data.age);
					}
				});
				break;
			case 'delete':
				var data = checkStatus.data;
				if (data.length == 0) {
					layer.msg("请选择一行删除");
					return;
				}
				//询问框
				layer.confirm('确定删除吗？', {
				  btn: ['确定','取消'] //按钮
				}, function(){
					$.ajax({
						url : "deleteUserList",
						type : "post",
						contentType : 'application/json; charset=UTF-8',
						data : JSON.stringify(data),
						success : function(data) {
							refresh();
							layer.msg('删除成功!');
						},
						error : function(data) {
							console.log(data);
						}
					});
				}, function(){
				 	console.log("取消删除");
				});
				break;
			}
		});
	});

	function refresh() {
		table.render({
			elem : '#test',
			url : '/MavenProject/queryUser',
			toolbar : '#toolbarDemo',
			title : '用户数据表',
			cols : [ [ {
				type : 'checkbox',
				fixed : 'left'
			}, {
				field : 'id',
				title : 'ID',
				sort : true,
				ort : true
			}, {
				field : 'name',
				title : '用户名',
				edit : 'text'
			}, {
				field : 'age',
				title : '年龄',
				edit : 'text'
			}, {
				field : 'address',
				title : '地址',
				edit : 'text'
			} ] ],
			page : true
		});
	}

	//打开新增页面
	$("#add").click(function() {
		$("#addUser").css("visibility", "visible");
	});

	/* 新增提交 */
	$("#addSubmit").click(function() {
		var data = $("#form2").serialize();
		data = changeFormData(data);

		$.ajax({
			url : "addUser", //你的路由地址
			type : "post",
			data : data,
			success : function(data) {
				$('#addUserModel').modal('hide');
				refresh();
			},
			error : function(data) {
				console.log(data);
			}
		});
	});

	//提交
	$("#editSubmit").click(function() {
		var d = $("#form1").serialize();
		$.ajax({
			url : "updateUser", //你的路由地址
			type : "post",
			data : d,
			success : function(data) {
				$('#editUserModel').modal('hide');
				refresh();
			},
			error : function(data) {
				console.log(data);
			}
		});
	});

	//把Form表单序列化的转换为json对象
	function changeFormData(data) {
		data = decodeURIComponent(data, true);//防止中文乱码  
		data = data.replace(/&/g, "','");
		data = data.replace(/=/g, "':'");
		data = "({'" + data + "'})";
		obj = eval(data);
		return obj;
	}
</script>






三.测试ssm配置是否成功
1.启动tomcat程序
2.在浏览器输入地址
http://localhost:8080/MavenProject/queryUser 
页面如下


分别进行增加、删除、修改，没有问题
3.验证redis二级缓存是否生效

验证redis是否启动了,    在终端输入命令  ps -aux | grep redis

Redis已经在6379端口号启动了（启动redis百度查看文档）
连接redis,


查看redis是存在key的，先清空，ok，现在在浏览器重新刷新页面。
在eclipse控制台，打印出sql的log为

继续查看redis的key是否被存入

可以明显的看到在刷新了浏览器之后，我在mybatis里面查询的数据已经写入到了redis缓存中，证明redis作为ssm的二级缓存配置没有问题。
把控制3.1首先连接到服务器的redis
台清空，在浏览器在刷新一次，发现，控制台并没有打印查询的log，也就是说，我们在查询相同的sql语句时候，首先在缓存中找，如果匹配到，则不去数据库查询数据，这样就减少了数据库的压力。

问题：如果，数据库数据发生了变化，那还去缓存中查数据，查出来的就都是脏数据了吗？
验证一下，首先连接Mysql数据库，

先保证数据库数据和页面数据一致，ok
现在继续在页面刷新获取的数据是redis缓存的数据。
我在数据库修改一行id=14的，把name值修改了this is test name。


这个时候再去刷新页面，注意看控制台的log有没有打印出来。发现并没有打印出来，
因为我在数据库直接修改，并没有通知redis，我已经修改了数据，但是我在浏览器里面修改，修改完成之后就会发现控制台又重新查询了一遍，剔除脏数据。
为了验证，再把redis的keys清空。
将id = 14 的name值修改:the chrome update,修改之后看控制台。


发现修改完成之后，重新查询了数据库，而且redis的数值也更新了。



四.打包部署到服务器上面发布出去
1.Maven项目打包
pom.xml里面加上这句话
<build>
	<finalName>MavenProject</finalName>
</build>
右键要打包的项目，Run As->Maven install，Ubuntu不好截图就不截图了，
控制台会打印如下信息，

打印完成查看项目结构

多了一个MavenProject.war也就是我刚刚打包的项目。


2.部署到服务器


重启服务器的tomcat，


输入服务器的地址:  http://www.wjx.com:8080/MavenProject/queryUser

至此，一个简单的ssm+redis项目已经部署到服务器上面了（加上文档编写，周期2天）；
后面继续补充！
五.注意事项
1.配置redis的时候，redis.conf的69 行 将 bind 127.0.0.1修改为bind 0.0.0.0，
不然外网telnet 114.116.24.32 6379 不通redis服务,获取不到redis实例
2.再配置mybatis二级缓存的时候，在b.mybatis-config.xml文件里面要开启二级缓存
并且在mapper.xml里面要添加如下语句（在上面的mapper.xml里面有这一句话）
<!-- 开启redis缓存 -->
<cache eviction="LRU" type="com.redis.RedisCache" />

3.解决redis的乱码
最近使用spring-data-redis RedisTemplate 操作redis时发现存储在redis中的key不是设置的string值，前面还多出了许多类似\xac\xed\x00\x05t\x00这种字符串，如下

127.0.0.1:6379> keys *
1) "\xac\xed\x00\x05t\x00\x04pass"
2) "\xac\xed\x00\x05t\x00\x04name"
3) "name"
解决方法：

private RedisTemplate redisTemplate;
	@Autowired(required = false)
	public void setRedisTemplate(RedisTemplate redisTemplate) {
         //加上这段配置
		RedisSerializer stringSerializer = new StringRedisSerializer();
		redisTemplate.setKeySerializer(stringSerializer);
		redisTemplate.setValueSerializer(stringSerializer);
		redisTemplate.setHashKeySerializer(stringSerializer);
		redisTemplate.setHashValueSerializer(stringSerializer);
	
		this.redisTemplate = redisTemplate;
	}

redisTemplate的set和get方法
//set数据
		redisTemplate.opsForValue().set("name", "tom");
		//get数据
		System.out.println("name:" + redisTemplate.opsForValue().get("a"));

4.MySQL索引背后的数据结构及算法原理
http://blog.jobbole.com/24006/
5.在Spring里面获取jdbc连接
	@Autowired
	private SqlSessionFactoryBean sessionFactoryBean;
     ......
		try {
			SqlSessionFactory sessionFactory = sessionFactoryBean.getObject();
			Connection connection = sessionFactory.openSession().getConnection();
			String sql = "select * from users";
			PreparedStatement preparedStatement = connection.prepareStatement(sql);
			ResultSet resultSet = preparedStatement.executeQuery();
			while(resultSet.next()){
				System.out.println("name:"+resultSet.getString("name"));
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
......

---
title: 常用RPC框架
date: 2017-06-22 15:16:22
tags: 总结
---
# DUBBO远程调用框架
### spring配置方式
[spring配置dubbo](http://blog.csdn.net/u013142781/article/details/50387583)
### dubbo-admin搭建
* 这个dubbo-admin部署到tomcat上只能使用jdk1.7以下版本否则会报错
* 教程
[配置dubbo-admin](http://blog.csdn.net/u013142781/article/details/50396621)
### Zookeeper搭建，这是个注册中心
* 服务器端将需要需要远程的方法注册到该注册中心
* 客户端进行订阅该注册中心
* 这里需要几个jar包
* dubbo netty zookeeper zkclient jline
* [zookeeper配置](http://blog.csdn.net/u013142781/article/details/50395650)
***
# redis的应用场景的介绍
[redis作者谈redis应用场景](http://blog.nosqlfan.com/html/2235.html)

* Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)
* string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。
* string类型是Redis最基本的数据类型，一个键最大能存储512MB。
* redis以内存作为存储介质的数据库，高速存储是他的特点，可以作为MySQL的高速缓存
* 众多语言都支持Redis，因为Redis交换数据快，所以在服务器中常用来存储一些需要频繁调取的数据，这样可以大大节省系统直接读取磁盘来获得数据的I/O开销，更重要的是可以极大提升速度。
* redis将数据写到内存，也会持久化到磁盘，断电之后重启数据会自动恢复，这点很强，数据是放到了运行内存中
* redis存储对象是将对象序列化
#### spring配置redis过程
* redis.xml
> spring也是将操作redis的方法封装到模板中了，这和jdbc和hibernate非常像 需要添加spring-data-redis包
```xml
<?xml version="1.0" encoding="UTF-8"?>   
<beans xmlns="http://www.springframework.org/schema/beans"   
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"   
    xmlns:p="http://www.springframework.org/schema/p"   
    xmlns:context="http://www.springframework.org/schema/context"   
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd   
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">  
 
    <context:property-placeholder location="classpath:redis.properties"/> 
    
    <bean id="propertyConfigurerRedis"  
        class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">  
        <property name="order" value="1" />  
        <property name="ignoreUnresolvablePlaceholders" value="true" />  
        <property name="systemPropertiesMode" value="1" />  
        <property name="searchSystemEnvironment" value="true" />  
        <property name="locations">  
        <list>  
            <value>classpath:redis.properties</value>  
        </list>  
        </property>  
    </bean>
    
    
    <bean id="jedisPoolConfig"  
        class="redis.clients.jedis.JedisPoolConfig">  
        <property name="maxIdle" value="300" />  
        <property name="testOnBorrow" value="true" />  
    </bean>
     
     
    <bean id="jedisConnectionFactory"  
        class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">  
        <property name="usePool" value="true"></property>  
        <property name="hostName" value="localhost" />  
        <property name="port" value="6379" />  
        <property name="timeout" value="5000" />  
        <property name="database" value="0"></property>  
        <constructor-arg index="0" ref="jedisPoolConfig" />  
    </bean>
    
    <!-- spring封装到模板了 -->
    <bean id="redisTemplate"  
        class="org.springframework.data.redis.core.StringRedisTemplate"    
        p:connectionFactory-ref="jedisConnectionFactory" >  
    </bean>
        
    <bean id="redisBase" abstract="true">  
        <property name="template" ref="redisTemplate"/>  
    </bean> 
    
  
</beans>
```

* java操作redis
```java
	ApplicationContext context = new FileSystemXmlApplicationContext("src\\redis.xml");
		final RedisTemplate<Serializable, Serializable> redis = (RedisTemplate<Serializable, Serializable>) context.getBean("redisTemplate");
		final JedisConnectionFactory rf = (JedisConnectionFactory) context.getBean("jedisConnectionFactory");
		 Jedis con = rf.getConnection().getNativeConnection();//获得了redis对象可以直接操作
 //其实也可以使用spring的模板但是感觉不方便
 //spring模板存储对象的时候比较好
 		
		System.out.println(con.get("spring"));
		System.out.println(con.keys("*"));
		con.set("jedis", "success");
```
### dubbo server的xml配置如下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
	xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:dubbo="http://code.alibabatech.com/schema/dubbo" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
	http://code.alibabatech.com/schema/dubbo  
    http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

	<bean id="server" class="dubboServer.DubboServer"></bean>
	
	<dubbo:application name="xixi_provider"/>
	
	 <dubbo:registry address="zookeeper://127.0.0.1:2181" />
	<dubbo:protocol name="dubbo" port="20880" />
	<dubbo:service interface="dubbo.RegisterService" ref="server" />	
	
</beans>
```
### dubbo client配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
	xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:dubbo="http://code.alibabatech.com/schema/dubbo" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
	http://code.alibabatech.com/schema/dubbo  
    http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

	
	<dubbo:application name="hehe_provider"/>
	<dubbo:registry address="zookeeper://127.0.0.1:2181" /> 	
	<dubbo:reference  id="RegisterService" interface="dubbo.RegisterService"></dubbo:reference>
</beans>
```
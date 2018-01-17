---
title: Spring随笔记录
date: 2017-09-02 13:27:56
tags: spring
---

#### spring FctoryBean and InitializingBean 
> Spring中有两种类型的Bean，一种是普通Bean，另一种是工厂Bean，即FactoryBean，这两种Bean都被容器管理，但工厂Bean跟普通Bean不同，其返回的对象不是指定类的一个实例，其返回的是该FactoryBean的getObject方法所返回的对象。

 FactoryBean接口有3个方法：

Object getObject():返回本工厂创建的对象实例。此实例也许是共享的，依赖于该工厂返回的是单例或者是原型。
boolean isSingleton():如果FactoryBean返回的是单例,该方法返回值为true,否则为false
Class getObjectType():返回对象类型。对象类型是getObject()方法返回的对象的类型，如果不知道的类型则返回null。
FactoryBean概念和接口在Spring框架中大量使用。Spring内置的有超过50个实现。

当使用ApplicationContext的getBean()方法获取FactoryBean实例本身而不是它所产生的bean，则要使用&符号+id。比如，现有FactoryBean，它有id，在容器上调用getBean("myBean")将返回FactoryBean所产生的bean，调用getBean("&myBean")将返回FactoryBean它本身的实例。

``` java
package spring4;

import org.springframework.beans.factory.FactoryBean;
import org.springframework.stereotype.Component;

/**
 * Created by javajidi_com
 */
@Component("myFactoryBean")
public class MyFactoryBean implements FactoryBean<Compent> {
    public Compent getObject() throws Exception {
        return new Compent();//这里可以进行负责的对象创建逻辑
    }

    public Class<?> getObjectType() {
        return Compent.class;
    }

    public boolean isSingleton() {
        return true;//是否是单列
    }
}

package spring4;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by javajidi_com
 */

public class Application {


    public static void main(String[] arg){
        //初始化spring容器，由于使用的是注解，没有xml文件，所有不再使用ClasspathXmlApplicationContext
        ApplicationContext context=new AnnotationConfigApplicationContext(Config.class);
        Object o1=context.getBean("myFactoryBean");
        Object o2=context.getBean("&myFactoryBean");
        System.out.println(o1.getClass());
        System.out.println(o2.getClass());
    }
}

输出
class spring4.Compent
class spring4.MyFactoryBean
```
> InitializingBean 和前面的fatorybean一样是一个接口，该接口只有一个方法，afterPropertiesSet()，该方法在bean注册到spring容器的时候就自动执行，所以多用来做一些初始化的工作，

```java
package com.sogou.nsd.factorybean;

import org.springframework.beans.factory.FactoryBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.stereotype.Component;

@Component("factorybean")
public class FactoryBeanTest implements FactoryBean, InitializingBean {
    @Override
    public Object getObject() throws Exception {
        return "this is a factorybean";
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("this method execute after bean xml file init!!!!!!!");
    }
}

```
#### springMVC HttpServletRequestWrapper
> 通常在filter中需要读取请求的参数，这个当请求是post的时候，就会存在问题，request的inputStream只能读取一次，当在controller中时就无法再次读取post的表单数据。这个时候就需要对request进行包装，每次读取的是request数据流的一个copy这样就不会存在问题，详细代码如下：
```java
package com.sogou.nsd.filter;

import sun.misc.IOUtils;
import sun.nio.ch.IOUtil;

import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.io.*;

public class MultiReadHttpServletReuest extends HttpServletRequestWrapper {
    private ByteArrayOutputStream cachedBytes;

    public MultiReadHttpServletReuest(HttpServletRequest request) {
        super(request);
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {
        if (cachedBytes == null)
            cacheInputStream();
        return new CachedServletInputStream();
    }

    @Override
    public BufferedReader getReader() throws IOException {
        return new BufferedReader(new InputStreamReader(getInputStream()));
    }

    private void cacheInputStream() throws IOException {
        /*
         * Cache the inputstream in order to read it multiple times. For
         * convenience, I use apache.commons IOUtils
         */
        cachedBytes = new ByteArrayOutputStream();
        org.apache.commons.io.IOUtils.copy(super.getInputStream(), cachedBytes);

    }

    /* An inputstream which reads the cached request body */
    public class CachedServletInputStream extends ServletInputStream {

        private ByteArrayInputStream input;

        public CachedServletInputStream() {
            /* create a new input stream from the cached request body */
            input = new ByteArrayInputStream(cachedBytes.toByteArray());
        }

        @Override
        public int read() throws IOException {
            return input.read();
        }
    }
}

```

然后在filter中将request进行包装就行，如下所示：
```java
package com.sogou.nsd.filter;

import javax.servlet.*;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import java.io.*;
import java.util.Map;

public class RequestBodyFilter extends HttpServlet implements Filter{
    private String filterName;
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        String name = filterConfig.getInitParameter("name");
        filterName = name;
        System.out.println("filterName: " + name);
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
//        HttpServletRequest request = (HttpServletRequest) servletRequest;
        MultiReadHttpServletReuest request = new MultiReadHttpServletReuest((HttpServletRequest) servletRequest);
        System.out.println(request.getMethod());
//        InputStream inputStream =  request.getInputStream();
//        InputStreamReader inputStreamReader = new InputStreamReader(inputStream);
        BufferedReader bufferedReader = request.getReader();
        String line;
        while ((line = bufferedReader.readLine()) != null)
            System.out.println(line);
            // when request method is post, request stream can only read one time, so this is
        // make the inputStream cached, and then read the cached every time;
        BufferedReader bufferedReader1 = request.getReader();  // return new object
        while ((line = bufferedReader1.readLine()) != null)
            System.out.println(line);

        String name = servletRequest.getParameter("name");
        System.out.println("name: " + name);
        System.out.println(filterName);
        filterChain.doFilter(request, servletResponse);
       // return;
    }
}
```

#### 有关sping和springmvc的关系

* context:annotation-config  
隐式在spring容器中注册 注解相关的bean  4个BeanPostProcessor
注册这4个bean处理器主要的作用是为了你的系统能够识别相应的注解
* 采用mvc:default-servlet-handler />
在springMVC-servlet.xml中配置mvc:default-servlet-handler />后，
会在Spring MVC上下文中定义一个org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler，
它会像一个检查员，对进入DispatcherServlet的URL进行筛查，如果发现是静态资源的请求，
就将该请求转由Web应用服务器默认的Servlet处理，如果不是静态资源的请求，才由DispatcherServlet继续处理。
* mvc:annotation-driven自动在spring容器中注册处理映射器和处理适配器
* context:component-scan base-package="com.sogou" 如果只在spring中加的话就是注册都到spring容器中使得该包及其子包下的bean定义注解工作起来 就是这些注解的作用要么是匹配要么是注册
为了防止多次加载多次实例化，这两个容器要分开。

一般在springmvc容器中只注册和controller相关的bean，其它都在spring容器进行注册，在spring容器中注册的bean对于两个容器都是可见的，下面是两个容器中的配置。
```xml
<!--springmvc  -->
	<context:component-scan base-package="com.sogou.bizwork.controller"  use-default-filters="false">
		<context:include-filter expression="org.springframework.stereotype.Controller" type="annotation" />
	</context:component-scan>
	<!--spring-->
		<context:component-scan base-package="com.sogou.bizwork">
		<context:exclude-filter expression="org.springframework.stereotype.Controller"
			type="annotation" />
	</context:component-scan>
```
这样配置就可以避免重复注册和加载的问题。

* 配置文件在容器中的设置，这个会有混淆，遇见过@Value注解不起作用的情况。
比如只在spring容器中配置资源文件
```xml
    <!--resource bundle file-->
    <context:property-placeholder location="classpath:redis.properties"/>
```
资源文件就无法在controller中以
>   @Value("${quorumString}")
    private String quorumString;
    
方式使用，这样输出的只是"${quorumString}"，必须要在springmvc容器中配置资源文件。

但是这样一旦资源文件改了，就需要改两处，这样就非常麻烦。
所以就有下面这种方式
先在spring容器定义bean，然后在每个容器分别引用就行了。
```xml
<!--spring-->
<bean id="properties" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
        <property name="ignoreResourceNotFound" value="true"/>
        <property name="locations">
            <list>
                <value>classpath:redis.properties</value>
            </list>
        </property>
    </bean>
    <!--resource bundle file ref-->
    <context:property-placeholder properties-ref="properties"/>
    
    <!--springMVC-->
      <context:property-placeholder properties-ref="properties"/>
```
这样就实现了配置一处，其它进行引用，实现了复用。

#### spring AOP



#### 注解
> Java中注解就是在代码中添加信息的一种形式化的方法，然后我们可以在以后某个时刻方便的使用这些数据。

java中是anatation接口，有四个元注解，自定义注解都是根据这个元注解来的，
Documented,Inherited,Target(作用范围，方法，属性，构造方法等),Retention(生命范围，源代码，class,runtime)。


>   @Target 表示该注解用于什么地方，可能的值在枚举类 ElemenetType 中，包括： 
          ElemenetType.CONSTRUCTOR 构造器声明 
          ElemenetType.FIELD 域声明（包括 enum 实例） 
          ElemenetType.LOCAL_VARIABLE 局部变量声明 
          ElemenetType.METHOD 方法声明 
          ElemenetType.PACKAGE 包声明 
          ElemenetType.PARAMETER 参数声明 
          ElemenetType.TYPE 类，接口（包括注解类型）或enum声明 

>   @Retention 表示在什么级别保存该注解信息。可选的参数值在枚举类型 RetentionPolicy 中，包括： 
          RetentionPolicy.SOURCE 注解将被编译器丢弃 
          RetentionPolicy.CLASS 注解在class文件中可用，但会被VM丢弃 
          RetentionPolicy.RUNTIME VM将在运行期也保留注释，因此可以通过反射机制读取注解的信息。 
          
> @Documented 将此注解包含在 javadoc 中 ，它代表着此注解会被javadoc工具提取成文档。在doc文档中的内容会因为此注解的信息内容不同而不同。相当与@see,@param 等。
>   @Inherited 允许子类继承父类中的注解，例子中补充。

如下自定义注解：
```java
package test;

import java.lang.annotation.*;

@Documented
@Retention(RetentionPolicy.RUNTIME)  // 定义注解的保留策略
@Target({ElementType.TYPE,ElementType.FIELD,ElementType.CONSTRUCTOR,ElementType.METHOD})         // 定义注解的作用目标
public @interface CacheAble {  // 自定义注解
    String name();
    int id() default 0;
    Class<Long> gid();
}
```

其中name参数也是name，类型和返回类型相同，其它也是这样

下面是使用注解
```java
package test;

@CacheAble(name = "nihao", id = 3, gid = Long.class)
public class ListNode {
    public ListNode next;
    public int val;
    public ListNode(int val){
        this.val = val;
    }
}

```
这样就相当于在该类中添加了一些信息，这些信息可以通过反射来获取使用。现在只是在类中加注解，也可以在成员变量上加注解，或者在方法或者参数上加注解。






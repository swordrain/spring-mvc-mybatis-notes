# 第一个Spring MVC引用

引入Spring MVC包或使用Maven等构建工具

在`web.xml`中配置`DispatcherServlet`

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" id="WebApp_ID" version="3.1">
  <!-- 定义Spring MVC的前端控制器 -->
  <servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>/WEB-INF/springmvc-config.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
   <!-- 让Spring MVC的前端控制器拦截所有请求 -->
  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

## 基于Controller接口的控制器

配置`springmvc-config.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd">
       
    <!-- 配置Handle，映射"/hello"请求 -->
    <bean name="/hello" class="org.fkit.controller.HelloController"/>

	<!-- 处理映射器将bean的name作为url进行查找，需要在配置Handle时指定name（即url） -->
	<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

	<!-- SimpleControllerHandlerAdapter是一个处理器适配器，所有处理适配器都要实现 HandlerAdapter接口-->
	<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
	
	<!-- 视图解析器 -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"/>

</beans>
```

实现`Controller`接口

```
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

/**
 *  HelloController是一个实现Controller接口的控制器,
 *  可以处理一个单一的请求动作
 */
public class HelloController implements Controller{
	 private static final Log logger = LogFactory
	            .getLog(HelloController.class);
	 
	 /**
	  * handleRequest是Controller接口必须实现的方法。
	  * 该方法的参数是对应请求的HttpServletRequest和HttpServletResponse。
	  * 该方法必须返回一个包含视图路径或视图路径和模型的ModelAndView对象。
	  * */
	@Override
	public ModelAndView handleRequest(HttpServletRequest request,
			HttpServletResponse response) throws Exception {
		 logger.info("handleRequest 被调用");
		 // 创建准备返回的ModelAndView对象，该对象通常包含了返回视图的路径、模型的名称以及模型对象
		 ModelAndView mv = new ModelAndView();
		 // 添加模型数据 可以是任意的POJO对象  
	     mv.addObject("message", "Hello World!");  
	     // 设置逻辑视图名，视图解析器会根据该名字解析到具体的视图页面  
	     mv.setViewName("/WEB-INF/content/welcome.jsp"); 
		// 返回ModelAndView对象。
		return mv;
	}

}
```

## 基于注解的控制器
重新实现Controller类，不再需要继承接口

```
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

/**
 *  HelloController是一个基于注解的控制器,
 *  可以同时处理多个请求动作，并且无须实现任何接口。
 *  org.springframework.stereotype.Controller注解用于指示该类是一个控制器
 */
@Controller
public class HelloController{

	 private static final Log logger = LogFactory
	            .getLog(HelloController.class);
	 
	 /**
	  * org.springframework.web.bind.annotation.RequestMapping注解
	  * 用来映射请求的的URL和请求的方法等。本例用来映射"/hello"
	  * hello只是一个普通方法。
	  * 该方法返回一个包含视图路径或视图路径和模型的ModelAndView对象。
	  * */
	 @RequestMapping(value="/hello")
	 public ModelAndView hello(){
		 logger.info("hello方法 被调用");
		 // 创建准备返回的ModelAndView对象，该对象通常包含了返回视图的路径、模型的名称以及模型对象
		 ModelAndView mv = new ModelAndView();
		 //添加模型数据 可以是任意的POJO对象  
	     mv.addObject("message", "Hello World!");  
	     // 设置逻辑视图名，视图解析器会根据该名字解析到具体的视图页面  
	     mv.setViewName("/WEB-INF/content/welcome.jsp"); 
		// 返回ModelAndView对象。
		return mv;
	 }

}

```

修改配置文件，需要使用扫描

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd     
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-4.2.xsd">
        
    <!-- spring可以自动去扫描base-pack下面的包或者子包下面的java文件，
    	如果扫描到有Spring的相关注解的类，则把这些类注册为Spring的bean -->
    <context:component-scan base-package="org.fkit.controller"/>
    
    <!-- 配置处理映射器 -->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>
     
     <!-- 配置处理器适配器-->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>

	<!-- 视图解析器 -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"/>
	
</beans>
```

## 详解DispatcherServlet

```
protected void initStrategies(ApplicationContext context){
	initMultipartResolver(context); //初始化上传文件解析器
	initLocaleResolver(context); //初始化本地化解析器
	initThemeResolver(context); //初始化主题解析器
	initHandlerMappings(context); //初始化处理器映射器，将请求映射到处理器
	initHandlerAdapters(context); //初始化处理器适配器
	initHanlderExcpetionResolvers(context); //初始化处理器异常解析器，如果执行过程中遇到异常将交给HandlerExceptionResolver来解析
	initRequestToViewNameTranslator(context); //初始化请求到视图名称解析器
	initViewResolvers(context); //初始化视图解析器，通过ViewResolver解析逻辑视图名到具体视图实现
	initFlashMapManager(context); //初始化flash映射管理器
}
```

`org.springframework.web.servlet`路径下的`DispatcherServlet.properties`配置文件指定了`DispatcherServlet`所使用的默认组件


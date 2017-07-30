# 常用注解

## @Controller
用于标记一个类，标记后该类就是一个Spring MVC Controller对象。Spring扫描机制会查找所有基于注解的控制器类。分发处理器会扫描使用了该注解的类的方法，检测该方法是否使用@RequestMapping注解

步骤

1. 在Spring MVC的配置文件的头文件中引入`spring-context`

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns="http://xmlns.jcp.org/xml/ns/javaee" 
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
	http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" 
	id="WebApp_ID" version="3.1">
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
  <!-- 编码过滤器 -->
  <filter>
		<filter-name>characterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
 </filter>
	<filter-mapping>
		<filter-name>characterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
</web-app>
```

2. 使用`<context:component-scan/>`元素，注册带有@Controller @Service @repository @Component等注解的类成为Spring的Bean

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
    
    <!-- 视图解析器  -->
     <bean id="viewResolver"
          class="org.springframework.web.servlet.view.InternalResourceViewResolver"> 
        <!-- 前缀 -->
        <property name="prefix">
            <value>/WEB-INF/content/</value>
        </property>
        <!-- 后缀 -->
        <property name="suffix">
            <value>.jsp</value>
        </property>
    </bean>
    
</beans>
```

```
@Controller
public class HelloWorldController{
	 
	 @RequestMapping("/helloWorld")
	 public String helloWorld(Model model) {
	     model.addAttribute("message", "Hello World!");
	     return "helloWorld";
	 }

}
```

## @RequestMapping
可以用来注释在类上，也可以注释在方法上

```
@Controller
@RequestMapping(value="/user")
public class UserController{
	@RequestMapping(value="/register") //请求地址是/user/register
	public String register(){
		return "register";
	}
}
```

* value

用于将指定请求的实际地址映射到方法上，可以多个，只有一个时可以简写成`@RequestMapping("/hello")`

* method

指定请求方法，如`@RequestMapping(method={RequestMethod.POST, RequestMathod.GET})`

* consumes

指定提交内容类型，如`@RequestMapping(consumes="application/json")`

* produces  

指定返回内容类型，返回的内容类型必须是request请求头（Accept）中所包含的类型，如`@RequestMapping(produces="application/json")`

* params

request中包含某些参数时才让该方法处理

* headers

request中必须包含某些指定的header时才让该方法处理

请求方法的参数会自动将对象依赖注入，只要将需要用的对象在方法中声明即可，比如`public String login(HttpSession session)`，一共支持有

* javax.servlet.ServletRequest或javax.servlet.http.HttpServletRequest
* javax.servlet.ServletResponse或javax.servlet.http.HttpServletResponse
* javax.servlet.http.HttpSession
* org.springframework.web.context.request.WebRequest或org.springframework.web.context.request.NativeWebRequest
* java.util.Locale
* java.io.InputStream或java.io.Reader
* java.io.OutputStream或java.io.Writer
* java.security.Principal
* HttpEntity<?>
* java.util.Map
* org.springframework.ui.Model
* org.springframework.ui.ModalMap
* org.springframework.web.servlet.mvc.support.RedirectAttributes
* org.springframework.validation.Errors
* org.springframework.validation.BindingResult
* org.springframework.web.bind.support.SessionStatus
* org.springframework.web.util.UriComponentsBuilder
* @PathVariable @@MatrixVariable
* @RequestParam @RequestHeader @RequestBody @RequestPart

请求处理方法可返回的类型

* org.springframework.web.portlet.ModelAndView
* org.springframework.ui.Model
* java.util.Map<k,v>
* org.springframework.web.servlet.View
* java.lang.String
* HttpEntity或ResponseEntity
* java.util.concurrent.Callable
* org.springframework.web.context.request.async.DefferedResult
* void

Modal和ModalMap

两个接口`org.springframework.ui.Model`和`org.springframework.ui.ModalMap`，使用如下方法添加数据`addObject(String attributeName, Object attributeValue)`

```
@Controller
public class User1Controller{
	
	private static final Log logger = LogFactory.getLog(User1Controller.class);
	
	// @ModelAttribute修饰的方法会先于login调用，该方法用于接收前台jsp页面传入的参数
	@ModelAttribute
	public void userModel(String loginname,String password, Model model){
		logger.info("userModel");
		// 创建User对象存储jsp页面传入的参数
		User user = new User();
		user.setLoginname(loginname);
		user.setPassword(password);
		// 将User对象添加到Model当中
		model.addAttribute("user", user);
	}
	
	@RequestMapping(value="/login1")
	 public String login(Model model){
		logger.info("login");
		// 从Model当中取出之前存入的名为user的对象
		User user = (User) model.asMap().get("user");
		System.out.println(user);
		// 设置user对象的username属性
		user.setUsername("测试");
		return "result1";
	}
}
```

```
@Controller
public class User2Controller{
	private static final Log logger = LogFactory.getLog(User2Controller.class);
	
	@ModelAttribute
	public void userMode2(String loginname,String password, ModelMap modelMap){
		logger.info("userMode2");
		// 创建User对象存储jsp页面传入的参数
		User user = new User();
		user.setLoginname(loginname);
		user.setPassword(password);
		// 将User对象添加到ModelMap当中
		modelMap.addAttribute("user", user);
	}
	
	@RequestMapping(value="/login2")
	 public String login2(ModelMap modelMap){
		logger.info("login2");
		// 从ModelMap当中取出之前存入的名为user的对象
		User user = (User) modelMap.get("user");
		System.out.println(user);
		// 设置user对象的username属性
		user.setUsername("测试");
		return "result2";
	}
}
```

ModelAndView

如果处理方法返回值是`ModelAndView`，则其既包含模型数据信息也包含视图信息。使用`addObject(String attributeName, Object attributeValue)`添加数据，使用`setViewName(String viewName)`设置视图

```
@Controller
public class User3Controller{
	private static final Log logger = LogFactory.getLog(User3Controller.class);
	
	@ModelAttribute
	public void userMode3(String loginname,String password, ModelAndView mv){
		logger.info("userMode3");
		User user = new User();
		user.setLoginname(loginname);
		user.setPassword(password);
		// 将User对象添加到ModelAndView的Model当中
		mv.addObject("user", user);
	}
	
	@RequestMapping(value="/login3")
	 public ModelAndView login3(ModelAndView mv){
		logger.info("login3");
		// 从ModelAndView的Model当中取出之前存入的名为user的对象
		User user = (User) mv.getModel().get("user");
		System.out.println(user);
		// 设置user对象的username属性
		user.setUsername("测试");
		// 设置返回的视图名称
		mv.setViewName("result3");
		return mv;
	}
}
```

## @RequestParam注解
`org.springframework.web.bind.annotation.RequestParam`注解类型用于将指定的请求参数赋值给方法中的形参

| 属性        | 类型         | 是否必要  | 说明 |
| ----- |------| ------ |  ----- |
| name  | String |   否   |   指定请求头绑定的名称  |
| value  | String |   否   |   name属性的别名  |
| required  | boolean |   否   |  指示参数是否必须绑定 默认为true |
| defaultValue  | String |   否   |   如果没有传递参数而使用的默认值  |

```
@Controller
// RequestMapping可以用来注释一个控制器类，此时，所有方法都将映射为相对于类级别的请求, 
// 表示该控制器处理所有的请求都被映射到 value属性所指示的路径下
@RequestMapping(value="/user")
public class UserController{
	
	// 静态List<User>集合，此处代替数据库用来保存注册的用户信息
	private static List<User> userList;
	
	// UserController类的构造器，初始化List<User>集合
	public UserController() {
		super();
		userList = new ArrayList<User>();
	}

	// 静态的日志类LogFactory
	private static final Log logger = LogFactory
            .getLog(UserController.class);

	// 该方法映射的请求为http://localhost:8080/context/user/register，该方法支持GET请求
	@RequestMapping(value="/register",method=RequestMethod.GET)
	 public String registerForm() {
		 logger.info("register GET方法被调用...");
		 // 跳转到注册页面
	     return "registerForm";
	 }
	
	// 该方法映射的请求为http://localhost:8080/RequestMappingTest/user/register，该方法支持POST请求
	 @RequestMapping(value="/register",method=RequestMethod.POST)
	 // 将请求中的loginname参数的值赋给loginname变量,password和username同样处理
	 public String register(
			 @RequestParam("loginname") String loginname,
			 @RequestParam("password") String password,
			 @RequestParam("username") String username) {
		 logger.info("register POST方法被调用...");
		 // 创建User对象
		 User user = new User();
		 user.setLoginname(loginname);
		 user.setPassword(password);
		 user.setUsername(username);
		 // 模拟数据库存储User信息
		 userList.add(user);
		 // 跳转到登录页面
	     return "loginForm";
	 }
	 
	// 该方法映射的请求为http://localhost:8080/RequestMappingTest/user/login
	 @RequestMapping("/login")
	 public String login(
			// 将请求中的loginname参数的值赋给loginname变量,password同样处理
			 @RequestParam("loginname") String loginname,
			 @RequestParam("password") String password,
			 Model model) {
		 logger.info("登录名:"+loginname + " 密码:" + password);
		 // 到集合中查找用户是否存在，此处用来模拟数据库验证
		 for(User user : userList){
			 if(user.getLoginname().equals(loginname) 
					 && user.getPassword().equals(password)){
				 model.addAttribute("user",user);
				 return "welcome";
			 }
		 }
	     return "loginForm";
	 }

}

```

## @PathVariable注解
`org.springframework.web.bind.annotation.PathVariable`注解类型可以方便获得请求URL中的动态参数，它只支持一个属性`value`，类型为`String`，如果省略则默认绑定同名参数
```
@Requestmapping(value="/pathVariableTest/{userId}")
public void pathVariableTest(@PathVariable Integer userId)
```

## @RequestHeader注解
`org.springframework.web.bind.annotation.RequestHeader`注解类型用于将请求的头信息区数据映射到处理方法的参数上

| 属性        | 类型         | 是否必要  | 说明 |
| ----- |------| ------ |  ----- |
| name  | String |   否   |   指定请求头绑定的名称  |
| value  | String |   否   |   name属性的别名  |
| required  | boolean |   否   |  指示参数是否必须绑定 默认为true |
| defaultValue  | String |   否   |   如果没有传递参数而使用的默认值  |

```
@RequestMapping(value="/requestHeaderTest")
public requestHeaderTest(@RequestHeader("User-Agent" String userAgent, @RequestHeader(value="Accept" String[] accepts))
```

## @CookieValue注解
`org.springframework.web.bind.annotation.CookieValue`用于将请求的Cookie数据映射到处理方法的参数上

| 属性        | 类型         | 是否必要  | 说明 |
| ----- |------| ------ |  ------ |
| name  | String |   否   |   指定请求头绑定的名称  |
| value  | String |   否   |   name属性的别名  |
| required  | boolean |   否   |  指示参数是否必须绑定 默认为true |
| defaultValue  | String |   否   |   如果没有传递参数而使用的默认值  |

```
@RequestMapping(value="/cookieValueTest")
public void cookieValueTest(@CookieValue(value="JSESSIONID", defaultValue="") String sessionId)
```

## @SessionAttribute注解
`org.springframework.web.bind.annotation.SessionAttribute`注解类型用于绑定HttpSession对象

| 属性        | 类型         | 是否必要  | 说明 |
| ----- |------| ------ |  ------ |
| name  | String[] |   否   |   指定请求头绑定的名称  |
| value  | String[] |   否   |   name属性的别名  |
| types  | Class<?>[] |   否   |  指示参数是否必须绑定 |

```
// Controller注解用于指示该类是一个控制器，可以同时处理多个请求动作
@Controller
// 将Model中的属性名为user的放入HttpSession对象当中
@SessionAttributes("user")
public class SessionAttributesController{

	// 静态的日志类LogFactory
	private static final Log logger = LogFactory.getLog(SessionAttributesController.class);
	
	// 该方法映射的请求为http://localhost:8080/DataBindingTest/{formName}
	@RequestMapping(value="/{formName}")
	 public String loginForm(@PathVariable String formName){
		// 动态跳转页面
		return formName;
	}

	// 该方法映射的请求为http://localhost:8080/DataBindingTest/login
	@RequestMapping(value="/login")
	 public String login(
			 @RequestParam("loginname") String loginname,
			 @RequestParam("password") String password,
			 Model model ) {
		 // 创建User对象，装载用户信息
		 User user = new User();
		 user.setLoginname(loginname);
		 user.setPassword(password);
		 user.setUsername("admin");
		 // 将user对象添加到Model当中
		 model.addAttribute("user",user);
		 return "welcome";
	 }
}
```

## @ModelAttribute注解
`org.springframework.web.bind.annotation.ModelAttribute`注解将请求参数绑定到Model对象，只支持一个属性`value`，类型为`String`

```
@Controller
public class ModelAttribute1Controller{

	// 使用@ModelAttribute注释的value属性，来指定model属性的名称,model属性对象就是方法的返回值
	@ModelAttribute("loginname")
	public String userModel1(@RequestParam("loginname") String loginname){
		return loginname;
	}

	@RequestMapping(value="/login1")
	 public String login1() {
		 return "result1";
	 }

}
```

被`@ModelAttribute`注解的`userModel1`会先于`login1`调用，它把请求参数`loginname`的值赋给`loginname`变量，并设置一个属性`loginname`到`Model`中，在`result1`中使用`${requestScope.loginname}`访问

多个属性赋值，jsp中的访问同前

```
@Controller
public class ModelAttribute2Controller{

	// model属性名称和model属性对象由model.addAttribute()实现，前提是要在方法中加入一个Model类型的参数。
	// 注意：当URL或者post中不包含对应的参数时，程序会抛出异常。
	@ModelAttribute
	public void userModel2( 
			@RequestParam("loginname") String loginname,
			@RequestParam("password") String password,
			 Model model){
		model.addAttribute("loginname", loginname);
		model.addAttribute("password", password);
	}

	@RequestMapping(value="/login2")
	 public String login2() {
		 return "result2";
	 }
}
```

赋值类对象，jsp中用`${requestScope.user.username}`访问

```
@Controller
public class ModelAttribute3Controller{
	
	// 静态List<User>集合，此处代替数据库用来保存注册的用户信息
	private static List<User> userList;
	
	// UserController类的构造器，初始化List<User>集合
	public ModelAttribute3Controller() {
		super();
		userList = new ArrayList<User>();
		User user1 = new User("test","123456","测试用户");
		User user2 = new User("admin","123456","管理员");
		// 存储User用户，用于模拟数据库数据
		userList.add(user1);
		userList.add(user2);
	}
	
	// 根据登录名和密码查询用户，用户存在返回包含用户信息的User对象，不存在返回null
	public User find(String loginname,String password){
		for(User user: userList){
			if(user.getLoginname().equals(loginname) && user.getPassword().equals(password)){
				return user;
			}
		}
		return null;
	}

	// model属性的名称没有指定，它由返回类型隐含表示，如这个方法返回User类型，那么这个model属性的名称是user。
    // 这个例子中model属性名称由返回对象类型隐含表示，model属性对象就是方法的返回值。它不需要指定特定的参数。
	@ModelAttribute
	public User userModel3( 
			@RequestParam("loginname") String loginname,
			@RequestParam("password") String password){
		return find(loginname, password);
	}

	@RequestMapping(value="/login3")
	 public String login3() {
		 return "result3";
	 }

}
```

同时注释`@ModelAttribute`和`@RequestMapping`，jsp中用`${requestScope.username}`访问

```
@Controller
public class ModelAttribute4Controller{
	
	 // 这时这个方法的返回值并不是表示一个视图名称，而是model属性的值，视图名称是@RequestMapping的value值。
	 // Model属性名称由@ModelAttribute(value=””)指定，相当于在request中封装了username（key）=admin（value）。
	@RequestMapping(value="/login4")
	@ModelAttribute(value="username")
	 public String login4() {
		 return "admin";
	 }

}
```

用`@ModelAttribute`注释方法的参数

```
@Controller
public class ModelAttribute5Controller{

		// model属性名称就是value值即”user”，model属性对象就是方法的返回值
		@ModelAttribute("user")
		public User userModel5( 
				@RequestParam("loginname") String loginname,
				@RequestParam("password") String password){
			User user = new User();
			user.setLoginname(loginname);
			user.setPassword(password);
			return user;
		}

    	// @ModelAttribute("user") User user注释方法参数，参数user的值来源于userModel5()方法中的model属性。
    	@RequestMapping(value="/login5")
		 public String login5(@ModelAttribute("user") User user) {
			user.setUsername("管理员");
			 return "result5";
		 }
}
```

## 信息转换

### HttpMessageConverter<T>接口
负责将请求信息转换为一个对象，并将对象绑定到请求方法的参数中或输出为响应信息

`DispatcherServlet`默认已经装配了`RequestMappingHandlerAdapter`作为`HandlerAdapter`组件的实现类，即`HttpMessageConvert`由`RequestMappingHandlerAdapter`使用，将请求信息转换为对象，或将对象转换为响应信息

接口中定义的若干方法

* canRead
* canWrite
* getSupportedMediaTypes
* read
* write

接口的若干实现类

* StringHttpMessageConverter
* FormHttpMessageConverter
* XmlAwareFormHttpMessageConverter
* ResourceHttpMessageConverter
* BufferedImageHttpMessageConverter
* ByteArrayHttpMessageConverter
* SourceHttpMessageConverter
* MarshallingHttpMessageConverter
* Jaxb2RootElementHttpMessageConverter
* MappingJackson2HttpMessageConverter
* RssChannelHttpMessageConverter
* AtomFeedHttpMessageConverter

`RequestMappingHandlerAdapter`默认装备了一下`HttpMessageConverter`

* StringHttpMessageConverter
* ByteArrayHttpMessageConverter
* SourceHttpMessageConverter
* XmlAwareFormHttpMessageConverter

如果要装配其他类型的`HttpMessageConverter`，可以在Spring的Web容器上下文中自行定义一个`RequestMappingHandlerAdapter`

```
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
	<property name="messageConverters">
		<list>
			<bean class="org.springframework.http.converter.StringHttpMessageConverter" />
			<bean class="org.springframework.http.converter.xml.xmlAwareFormHttpMessageConverter" />
			<bean class="org.springframework.http.converter.ByteArrayHttpMessageConverter" />
			<bean class="org.springframework.http.converter.BufferedImageHttpMessageConverter" />
		</list>
	</property>
</bean>
```

### 转换JSON数据
默认使用MapperingJackson2HttpMessageConverter

`org.springframework.web.bind.annotation.RequestBody`注解用于读取`Request`请求的body部分数据，使用系统默认配置的`HttpMessageConverter`解析，然后把数据绑定到Controller中方法的参数上

* application/x-www-form-unlencoded 可以用`@RequestParam` `@ModelAttribute` `@RequestBody`处理
* multipart/form-data 不能用`@RequestBody`处理
* application/json application/xml 必须用`@RequestBody`处理

```
@Controller
@RequestMapping("/json")
public class BookController {
	
	private static final Log logger = LogFactory.getLog(BookController.class);
	
	// @RequestBody根据json数据，转换成对应的Object
    @RequestMapping(value="/testRequestBody")
    public void setJson(@RequestBody Book book, HttpServletResponse response) throws Exception{
    	// ObjectMapper类是Jackson库的主要类。它提供一些功能将Java对象转换成对应的JSON格式的数据
    	ObjectMapper mapper = new ObjectMapper();
    	// 将book对象转换成json输出
    	logger.info(mapper.writeValueAsString(book) );
    	book.setAuthor("肖文吉");
    	response.setContentType("text/html;charset=UTF-8");
    	// 将book对象转换成json写出到客户端
    	response.getWriter().println(mapper.writeValueAsString(book));
    }
}
```

POJO的`Book`类实现`Serializable`接口

springmvc-config.xml配置

```
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
	<!-- 设置配置方案  -->
	<mvc:annotation-driven/>
	<!-- 使用默认的Servlet来响应静态文件 -->
    <mvc:default-servlet-handler/>
	

    <!-- 视图解析器  -->
     <bean id="viewResolver"
          class="org.springframework.web.servlet.view.InternalResourceViewResolver"> 
        <!-- 前缀 -->
        <property name="prefix">
            <value>/WEB-INF/content/</value>
        </property>
        <!-- 后缀 -->
        <property name="suffix">
            <value>.jsp</value>
        </property>
    </bean>
    
</beans>
```

`<mvc:annotation-driven>`会自动注册`RequestMappingHandlerMapping`与`RequestMappingHandlerAdapter`两个Bean，这是Spring MVC为@Controller分发请求所必需的，并提供了`@NumberFormatannotation`支持、`@DateTimeFormat`支持、`@Valid`支持，读写XML和JSON支持

`<mvc:default-servlet-handler/>`使用默认的Servlet响应静态文件

配置`fastjson`接收json数据

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:util="http://www.springframework.org/schema/util"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd     
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-4.2.xsd
        http://www.springframework.org/schema/util
        http://www.springframework.org/schema/util/spring-util-4.2.xsd">
        
    <!-- spring可以自动去扫描base-pack下面的包或者子包下面的java文件，
    	如果扫描到有Spring的相关注解的类，则把这些类注册为Spring的bean -->
    <context:component-scan base-package="org.fkit.controller"/>
	<!-- 使用默认的Servlet来响应静态文件 -->
    <mvc:default-servlet-handler/>
	<!-- 设置配置方案 -->
    <mvc:annotation-driven>
    	<!-- 设置不使用默认的消息转换器 -->
        <mvc:message-converters register-defaults="false">
        	<!-- 配置Spring的转换器 -->
        	<bean class="org.springframework.http.converter.StringHttpMessageConverter"/>
    		<bean class="org.springframework.http.converter.xml.XmlAwareFormHttpMessageConverter"/>
    		<bean class="org.springframework.http.converter.ByteArrayHttpMessageConverter"/>
    		<bean class="org.springframework.http.converter.BufferedImageHttpMessageConverter"/>
            <!-- 配置fastjson中实现HttpMessageConverter接口的转换器 -->
            <bean id="fastJsonHttpMessageConverter" 
            	class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter">
                <!-- 加入支持的媒体类型：返回contentType -->
                <property name="supportedMediaTypes">
                    <list>
                        <!-- 这里顺序不能反，一定先写text/html,不然ie下会出现下载提示 -->
                        <value>text/html;charset=UTF-8</value>
                        <value>application/json;charset=UTF-8</value>
                    </list>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
    
    <!-- 视图解析器  -->
     <bean id="viewResolver"
          class="org.springframework.web.servlet.view.InternalResourceViewResolver"> 
        <!-- 前缀 -->
        <property name="prefix">
            <value>/WEB-INF/content/</value>
        </property>
        <!-- 后缀 -->
        <property name="suffix">
            <value>.jsp</value>
        </property>
    </bean>
    
</beans>
```

返回JSON格式的数据

```
@Controller
@RequestMapping("/json")
public class BookController {
    @RequestMapping(value="/testRequestBody")
    // @ResponseBody会将集合数据转换json格式返回客户端
    @ResponseBody
    public Object getJson() {
    	List<Book> list = new ArrayList<Book>();
    	list.add(new Book(1,"Spring MVC企业应用实战","肖文吉"));
    	list.add(new Book(2,"轻量级JavaEE企业应用实战","李刚"));
    	return list;
    }

}
```

### XML数据

```
@Controller
public class BookController {
	
	private static final Log logger = LogFactory.getLog(BookController.class);
	 
	// @RequestBody Book book会将传递的xml数据自动绑定到Book对象
	 @RequestMapping(value="/sendxml", method=RequestMethod.POST)  
	 public void sendxml(@RequestBody Book book) {  
		 logger.info(book);
		 logger.info("接收XML数据成功");
	 }  
	 
	// @ResponseBody 会将Book自动转成XML数据返回
	 @RequestMapping(value="/readxml", method=RequestMethod.POST)  
	 public @ResponseBody Book readXml()throws Exception { 
		 // 通过JAXBContext的newInstance方法，传递一个class就可以获得一个上下文
		 JAXBContext context = JAXBContext.newInstance(Book.class);  
		 // 创建一个Unmarshall对象
		 Unmarshaller unmar = context.createUnmarshaller();  
		 InputStream is = this.getClass().getResourceAsStream("/book.xml");
		 // Unmarshall对象的unmarshal方法可以进行xml到Java对象的转换
		 Book book = (Book) unmar.unmarshal(is);  
		 logger.info(book); 
    	 return book;
	 }  

}
```

POJO

```
// @XmlRootElement表示XML文档的根元素
@XmlRootElement
public class Book implements Serializable {

	private Integer id;
	private String name;
	private String author;

	public Book() {
		super();
		// TODO Auto-generated constructor stub
	}

	public Book(Integer id, String name, String author) {
		super();
		this.id = id;
		this.name = name;
		this.author = author;
	}
	
	public Integer getId() {
		return id;
	}
	// 该属性作为xml的element
	@XmlElement
	public void setId(Integer id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}
	@XmlElement
	public void setName(String name) {
		this.name = name;
	}

	public String getAuthor() {
		return author;
	}
	@XmlElement
	public void setAuthor(String author) {
		this.author = author;
	}

	@Override
	public String toString() {
		return "Book [id=" + id + ", name=" + name + ", author=" + author + "]";
	}

}
```
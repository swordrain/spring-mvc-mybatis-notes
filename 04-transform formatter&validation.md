# Spring MVC的数据转换、格式化和数据校验
## 数据绑定流程
数据绑定核心部件是`DataBinder`，Spring MVC框架将`ServletRequest`对象及处理方法的参数对象实例传递给`DataBinder`，`DataBinder`调用装配在Spring MVC上下文中的`ConversionService`组件进行数据类型转换、数据格式化工作，并将`ServletRequest`中的消息填充到参数对象中。然后调用`Validator`组件对已经绑定了请求消息参数对象进行合法性校验，并最终生成数据绑定结果`BindingResult`对象。`BindingResult`包含已完成数据绑定的参数对象，还包含响应的校验错误对象，Spring MVC抽取`BindingResult`中的参数对象及校验错误对象，将它们赋给处理方法的响应参数。

## 数据转换
**ConversionService**

`org.springframework.core.convert.ConversionService`和类型转换的核心接口

* canConvert
* convert

可以利用`org.springframework.context.support.ConversionServiceFactoryBean`在Spring上下文中定义一个`ConversionService`。Spring将自动识别出上下文中的`ConversionService`，并在Spring MVC处理方法的参数绑定中使用它进行数据转换

```
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
	<property name="converters">
		<list>
			<bean class="org.fkit.converter.StringToDateConverter">
		</list>
	</property>
</bean>
```

**Spring支持的转换器**

三种类型的转换器接口

* Converter<S, T> - 负责将S类型对象转为T类型对象
* ConverterFactory<S, R> - 将相同系列多个Converter封装在一起
* GenericConvert<S, T> - 将S类型转为T类型，不考虑类型对象上下文信息

**使用ConversionService将字符串`2016-01-01`转为`Date`类型示例**

```
//StringToDateConverter.java
// 实现Converter<S,T>接口
public class StringToDateConverter implements Converter<String, Date>{

	// 日期类型模板：如yyyy-MM-dd
	private String datePattern;
	
	public void setDatePattern(String datePattern) {
		this.datePattern = datePattern;
	}

	// Converter<S,T>接口的类型转换方法
	@Override
	public Date convert(String date) {
		try {
			SimpleDateFormat dateFormat = new SimpleDateFormat(this.datePattern);
			// 将日期字符串转换成Date类型返回
			return dateFormat.parse(date);
		} catch (Exception e) {
			e.printStackTrace();
			System.out.println("日期转换失败!");
			return null;
		}
		
	}
}
```

在springmvc-config.xml里配置

```
<!-- 自定义的类型转换器 -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
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
    
    <!-- 装配自定义的类型转换器 -->
    <mvc:annotation-driven conversion-service="conversionService"/>
    
    <!-- 自定义的类型转换器 -->
     <bean id="conversionService" 
    	class="org.springframework.context.support.ConversionServiceFactoryBean">
    	<property name="converters">
    		<list>
    			<bean class="org.fkit.converter.StringToDateConverter"
    			p:datePattern="yyyy-MM-dd"></bean>
    		</list>
    	</property>
    </bean>
    
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

**使用`@InitBinder`添加自定义编辑器转换数据**

```
public class DateEditor extends PropertyEditorSupport {

	// 将传如的字符串数据转换成Date类型
	@Override
	public void setAsText(String text) throws IllegalArgumentException {
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
		try {
			Date date = dateFormat.parse(text);
			setValue(date);
		} catch (ParseException e) {
			e.printStackTrace();
		}
	}
	
}
```

在Controller配置

```
@Controller
public class UserController{
	
	private static final Log logger = LogFactory.getLog(UserController.class);
	
	@RequestMapping(value="/{formName}")
	 public String loginForm(@PathVariable String formName){
		// 动态跳转页面
		return formName;
	}
	 // 在控制器初始化时注册属性编辑器
	 @InitBinder
	  public void initBinder(WebDataBinder binder){
		// 注册自定义编辑器
		binder.registerCustomEditor(Date.class, new DateEditor());
	  }
	 
	 @RequestMapping(value="/register",method=RequestMethod.POST)
	 public String register(
			 @ModelAttribute User user,
			 Model mode) {
	     logger.info(user);
	     mode.addAttribute("user", user);
	     return "success";
	 }
}
```

**使用`WebBindingInitializaer`注册全局自定义编辑器转换数据**

```
// 实现WebBindingInitializer接口
public class DateBindingInitializer implements WebBindingInitializer {
	@Override
	public void initBinder(WebDataBinder binder, WebRequest request) {
		// 注册自定义编辑器
		binder.registerCustomEditor(Date.class, new DateEditor());
	}
}
```

在springmvc-config.xml中配置

```
<!-- 通过AnnotationMethodHandlerAdapter装配自定义编辑器 -->
<bean
	class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
	<property name="webBindingInitializer">
		<bean class="org.fkjava.binding.DateBindingInitializer" />
	</property>
</bean>
```

**转换器的优先顺序**

1. 查询通过@InitBinder装配的自定义编辑器
2. 查询通过ConversionService装配的自定义编辑器
3. 查询通过WebBindingInitializer接口装配的全局自定义编辑器

## 数据格式化
`org.springframework.format`包里的若干接口

* Printer<T>
* parser<T>
* Formatter<T>
* FormatterRegistrar
* AnnotationFormatterFactor<A extends Annotation>

**使用Formatter格式化数据**

```
// 实现Converter<S,T>接口
public class DateFormatter implements Formatter<Date>{

	// 日期类型模板：如yyyy-MM-dd
	private String datePattern;
	// 日期格式化对象
	private SimpleDateFormat dateFormat;
	
	// 构造器，通过依赖注入的日期类型创建日期格式化对象
	public DateFormatter(String datePattern) {
		this.datePattern = datePattern;
		this.dateFormat = new SimpleDateFormat(datePattern);
	}

	// 显示Formatter<T>的T类型对象
	@Override
	public String print(Date date, Locale locale) {
		return dateFormat.format(date);
	}

	// 解析文本字符串返回一个Formatter<T>的T类型对象。
	@Override
	public Date parse(String source, Locale locale) throws ParseException {
		try {
			return dateFormat.parse(source);
		} catch (Exception e) {
			throw new IllegalArgumentException();
		}
	}
}
```

在springmvc-config.xml里的配置

```
<!-- 格式化 -->
<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
	<property name="formatters">
		<list>
			<bean class="org.fkit.formatter.DateFormatter" c:_0="yyyy-MM-dd"/>
		</list>
	</property>
</bean>
```

**使用FormatterRegister注册Formatter**

```
public class MyFormatterRegistrar implements FormatterRegistrar{
	
	private DateFormatter dateFormatter;

	public void setDateFormatter(DateFormatter dateFormatter) {
		this.dateFormatter = dateFormatter;
	}

	@Override
	public void registerFormatters(FormatterRegistry registry) {
		
		registry.addFormatter(dateFormatter);
		
	}

}
```

在springmvc-config.xml里修改

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:c="http://www.springframework.org/schema/c"
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
    
    <!-- 装配自定义格式化 -->
    <mvc:annotation-driven conversion-service="conversionService"/>
    
	<!-- DateFormatter bean -->
    <bean id="dateFormatter" class="org.fkit.formatter.DateFormatter" 
    c:_0="yyyy-MM-dd"/>
    
    <!-- 格式化 -->
     <bean id="conversionService" 
    	class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    	<property name="formatterRegistrars">
    		<set>
    			<bean class="org.fkit.formatter.MyFormatterRegistrar" 
    			p:dateFormatter-ref="dateFormatter"/>
    		</set>
    	</property>
    </bean>

    
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

**使用AnnotationFormatterFactory<A extends Annotation>格式化数据**

Spring为开发者提供了注解驱动的属性对象格式化功能：在Bean属性中设置、Spring MVC处理方法参数绑定数据、模型数据输出时自动通过注解应用格式化的功能

在`org.springframework.format.annotation`包下面定义了两个格式化的注解类型

* DateTimeFormat - 对java.util.Date、java.util.Calendar等时间类型属性标注，拥有以下几个互斥的属性
	* iso
		* DateTimeFormat.ISO.DATE：格式为yyyy-MM-dd
		* DateTimeFormat.ISO.DATE_TIME：格式为yyyy-MM-dd hh:mm:ss .SSSZ
		* DateTimeFormat.ISO.TIME：格式为hh:mm:ss .SSSZ
		* DateTimeFormat.ISO.NONE：表示不使用ISO格式的时间
	* pattern 类型为String，使用自定义的时间格式，如"yyyy-MM-dd hh:mm:ss"
	* style 类型为String，通过样式指定日期时间的格式，两个字符，第一位表示日期的样式，第二位表示时间的格式，以下是几个常用的可选值
		* S：短日期/时间的样式
		* M：中日期/时间的样式
		* L：长日期/时间的样式
		* F：完整日期/时间的样式
		* -：忽略日期/时间的样式
* NumberFormat - 对数字类型的属性进行标注，拥有两个互斥的属性
	* pattern：类型为String，使用自定义的格式化串，如"##, ###。 ##" 
	* style：
		* NumberFormat.CURRENCY
		* NumberFormat.NUMBER
		* NumberFormat.PERCENT

```
//User.java
public class User implements Serializable{
	
	// 日期类型
	@DateTimeFormat(pattern="yyyy-MM-dd")
	private Date birthday;
	// 正常数字类型
	@NumberFormat(style=Style.NUMBER, pattern="#,###")  
    private int total;  
	// 百分数类型
    @NumberFormat(style=Style.PERCENT)  
    private double discount;  
    // 货币类型
    @NumberFormat(style=Style.CURRENCY)  
    private double money;  
    ...
}
```

```
//UserController.java
@Controller
public class FormatterController{
	
	private static final Log logger = LogFactory.getLog(FormatterController.class);
	 
	@RequestMapping(value="/{formName}")
	 public String loginForm(@PathVariable String formName){
		
		// 动态跳转页面
		return formName;
	}
	 
	 @RequestMapping(value="/test",method=RequestMethod.POST)
	 public String test(
			 @ModelAttribute User user,
			 Model model) {
		 logger.info(user);
		 model.addAttribute("user", user);
	     return "success";
	 }
}
```

springmvc-config.xml要启用

```
<mvc:annotation-driven />
```

## 数据校验
两种方式：Spring自带的Validation校验框架，利用JSR 303

**Spring的Validation校验框架**

Spring的校验框架在`org.springframework.validation`包中，重要的接口和类如下

* Validator - 该接口有两个方法
	* boolean supports(Class<?> clazz) - 该校验器能够对clazz类型的对象进行校验
	* void validate(Object target, Errors errors) - 对目标类target进行校验，并将校验错误记录在errors中
* Errors - 存放错误信息的接口。校验后校验结果的参数对象必须是Errors或者BindingResult类型。一个Errors对象中包含了一系列的`FieldError`和`ObjectError`对象
* ValidationUtils - 校验的工具类，提供了多个给Errors对象保存错误的方法
* LocalValidatorFactoryBean - 位于`org.springframework.validation.beanvalidation`包中，实现了Spring Validator和JSR 303的Validator接口，只要在Spring容器中定义个`LocalValidatorFactoryBean`，即可将其注入要需要检验的Bean中。如`<bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean" />`

`<mvc:annotation-driven />`会默认装配好一个`LocalValidatorFactoryBean`

```
//jsp部分
<form:form modelAttribute="user" method="post" action="login" >
	<table>
		<tr>
			<td>登录名:</td>
			<td><form:input path="loginname"/></td>
			<!-- 显示loginname属性的错误信息 -->
			<td><form:errors path="loginname" cssStyle= "color:red"/></td>
		</tr>
		<tr>
			<td>密码:</td>
			<td><form:input path="password"/></td>
			<!-- 显示password属性的错误信息 -->
			<td><form:errors path="password" cssStyle= "color:red"/></td>
		</tr>
		<tr>
			<td><input type="submit" value="提交"/></td>
		</tr>
	</table>
</form:form>
```

```
//UserValidator.java
// 实现Spring的Validator接口
@Repository("userValidator")
public class UserValidator implements Validator {

	// 该校验器能够对clazz类型的对象进行校验。
	@Override
	public boolean supports(Class<?> clazz) {
		// User指定的 Class 参数所表示的类或接口是否相同，或是否是其超类或超接口。
		return User.class.isAssignableFrom(clazz);
	}

	// 对目标类target进行校验，并将校验错误记录在errors当中
	@Override
	public void validate(Object target, Errors errors) {
		/**
		使用ValidationUtils中的一个静态方法rejectIfEmpty()来对loginname属性进行校验，
		假若'loginname'属性是 null 或者空字符串的话，就拒绝验证通过 。
		*/
		ValidationUtils.rejectIfEmpty(errors, "loginname", null, "登录名不能为空");  
		ValidationUtils.rejectIfEmpty(errors, "password", null, "密码不能为空");  
		User user = (User)target;
		if(user.getLoginname().length() > 10){
			// 使用Errors的rejectValue方法验证
			errors.rejectValue("loginname", null, "用户名不能超过10个字符");
		}
		if(user.getPassword() != null 
				&& !user.getPassword().equals("") 
				&& user.getPassword().length() < 6){
			errors.rejectValue("password", null, "密码不能小于6位");
		}
	}

}
```

```
//Controller部分
@Controller
public class UserController{
	
	private static final Log logger = LogFactory.getLog(UserController.class);
	 
	// 注入UserValidator对象
	@Autowired
	@Qualifier("userValidator")
	private UserValidator userValidator;
	
	@RequestMapping(value="/{formName}")
	 public String loginForm(
			 @PathVariable String formName,
			 Model model){
		User user = new User();
		model.addAttribute("user",user);
		// 动态跳转页面
		return formName;
	}
	 
	 @RequestMapping(value="/login",method=RequestMethod.POST)
	 public String login(
			 @ModelAttribute User user,
			 Model model,
			 Errors errors) {
		 logger.info(user);
		 model.addAttribute("user", user);
		 // 调用userValidator的验证方法
		 userValidator.validate(user, errors);
		 // 如果验证不通过跳转到loginForm视图
		 if(errors.hasErrors()){
			 return "loginForm";
		 }
	     return "success";
	 }

}
```

**JSR 303校验**

[参考](http://jcp.org/en/jsr/detail?id=303)

核心接口是`javax.validation.Validator`，两个实现有`Hibernate Validator`和`Apache bval`

* @Null
* @NotNull
* @AssertTrue
* @AssertFalse
* @Max(value)
* @Min(value)
* @DecimalMax(value)
* @DecimalMin(value)
* @Digits(integer, fraction)
* @Size(min, max)
* @Past
* @Future
* @Pattern

Hibernate Validator实现还提供了

* @NotBlank
* @URL
* @Email
* @CreditCardNumber
* @Length(min, max)
* @NotEmpty
* @Range(min, max, message)

```
//POJO
public class User implements Serializable{
	
	@NotBlank
	private String loginname;
	
	@NotBlank
	@Length(min=6,max=8)
	private String password;
	
	@NotBlank
	private String username;
	
	@Range(min=15, max=60)
	private int age;
	
	@Email
	private String email;
	
	@DateTimeFormat(pattern="yyyy-MM-dd")
	@Past
	private Date birthday;
	
	@Pattern(regexp="[1][3,8][3,6,9][0-9]{8}")
	private String phone;
	
	...
}
```

```
//Controller
@Controller
public class UserController{
	
	private static final Log logger = LogFactory.getLog(UserController.class);
	
	@RequestMapping(value="/{formName}")
	 public String loginForm(
			 @PathVariable String formName,
			 Model model){
		User user = new User();
		model.addAttribute("user",user);
		// 动态跳转页面
		return formName;
	}
	 
	// 数据校验使用@Valid，后面跟着Errors对象保存校验信息
	 @RequestMapping(value="/login",method=RequestMethod.POST)
	 public String login(
			 @Valid @ModelAttribute  User user,
			 Errors  errors,
			 Model model) {
		 logger.info(user);
		 if(errors.hasErrors()){
			 return "registerForm";
		 }
		 model.addAttribute("user", user);
	     return "success";
	 }

}
```
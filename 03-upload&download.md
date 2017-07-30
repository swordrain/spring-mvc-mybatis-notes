# Spring MVC的文件上传和下载

## 文件上传
前端

```
<form action="upload" enctype="multipart/form-data" method="post">
	<input type="file" name="file">
</form>
```

控制器

```
@Controller
public class FileUploadController{	
	// 上传文件会自动绑定到MultipartFile中
	 @RequestMapping(value="/upload",method=RequestMethod.POST)
	 public String upload(HttpServletRequest request,
			@RequestParam("description") String description,
			@RequestParam("file") MultipartFile file) throws Exception{
		 
	    System.out.println(description);
	    // 如果文件不为空，写入上传路径
		if(!file.isEmpty()){
			// 上传文件路径
			String path = request.getServletContext().getRealPath(
	                "/images/");
			// 上传文件名
			String filename = file.getOriginalFilename();
		    File filepath = new File(path,filename);
			// 判断路径是否存在，如果不存在就创建一个
	        if (!filepath.getParentFile().exists()) { 
	        	filepath.getParentFile().mkdirs();
	        }
	        // 将上传文件保存到一个目标文件当中
			file.transferTo(new File(path+File.separator+ filename));
			return "success";
		}else{
			return "error";
		}
		 
	 }
}
```

Spring MVC将上传文件绑定到`MultipartFile`对象中，通过`transferTo`方法存储到硬件中

* getBytes
* getContentType
* getInputStream
* getName
* getOriginalFilename
* getSize
* isEmpty
* transferTo

springmvc-config.xml的配置

```
<bean id="multipartResolver"  
        class="org.springframework.web.multipart.commons.CommonsMultipartResolver">  
	<!-- 上传文件大小上限，单位为字节（10MB） -->
    <property name="maxUploadSize">  
        <value>10485760</value>  
    </property>  
    <!-- 请求的编码格式，必须和jSP的pageEncoding属性一致，以便正确读取表单的内容，默认为ISO-8859-1 -->
    <property name="defaultEncoding">
    	<value>UTF-8</value>
    </property>
</bean>
```

## 文件下载
```
@RequestMapping(value="/download")
 public ResponseEntity<byte[]> download(HttpServletRequest request,
		 @RequestParam("filename") String filename,
		 Model model)throws Exception{
	// 下载文件路径
	String path = request.getServletContext().getRealPath(
            "/images/");
	File file = new File(path+File.separator+ filename);
    HttpHeaders headers = new HttpHeaders();  
    // 下载显示的文件名，解决中文名称乱码问题  
    String downloadFielName = new String(filename.getBytes("UTF-8"),"iso-8859-1");
    // 通知浏览器以attachment（下载方式）打开图片
    headers.setContentDispositionFormData("attachment", downloadFielName); 
    // application/octet-stream ： 二进制流数据（最常见的文件下载）。
    headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
    // 201 HttpStatus.CREATED
    return new ResponseEntity<byte[]>(FileUtils.readFileToByteArray(file),    
            headers, HttpStatus.CREATED);  
}
```

## 拦截器
HandlerInterceptor接口

* preHandle - 返回true会继续下一个Interceptor的preHandle方法，如果是最后一个Interceptor，会调用Controller方法
* postHandle - 只在preHandle返回true后被调用，时机在Controller方法调用之后，在DispatcherServlet进行视图返回渲染之前，次序与preHandler相反，先声明的Interceptor的postHandle方法在后执行
* afterCompletion - 只在preHandle返回true后被调用，时机在整个请求结束之后，即在DispatcherServlet渲染了对应视图之后，主要作用是进行资源清理

用户权限验证

```
/** 
 * 拦截器必须实现HandlerInterceptor接口
 * */ 
public class AuthorizationInterceptor  implements HandlerInterceptor {

	// 不拦截"/loginForm"和"/login"请求
	private static final String[] IGNORE_URI = {"/loginForm", "/login"};
	
	 /** 
     * 该方法将在整个请求完成之后执行， 主要作用是用于清理资源的，
     * 该方法也只能在当前Interceptor的preHandle方法的返回值为true时才会执行。 
     */  
	@Override
	public void afterCompletion(HttpServletRequest request,
			HttpServletResponse response, Object handler, Exception exception)
			throws Exception {
		System.out.println("AuthorizationInterceptor afterCompletion --> ");
		
	}
	/** 
     * 该方法将在Controller的方法调用之后执行， 方法中可以对ModelAndView进行操作 ，
     * 该方法也只能在当前Interceptor的preHandle方法的返回值为true时才会执行。 
     */
	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response,
			Object handler, ModelAndView mv) throws Exception {
		System.out.println("AuthorizationInterceptor postHandle --> ");
		
	}

	 /** 
     * preHandle方法是进行处理器拦截用的，该方法将在Controller处理之前进行调用，
     * 该方法的返回值为true拦截器才会继续往下执行，该方法的返回值为false的时候整个请求就结束了。 
     */  
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
			Object handler) throws Exception {
		System.out.println("AuthorizationInterceptor preHandle --> ");
		// flag变量用于判断用户是否登录，默认为false 
		boolean flag = false; 
		//获取请求的路径进行判断
		String servletPath = request.getServletPath();
		// 判断请求是否需要拦截
        for (String s : IGNORE_URI) {
            if (servletPath.contains(s)) {
                flag = true;
                break;
            }
        }
        // 拦截请求
        if (!flag){
        	// 1.获取session中的用户 
        	User user = (User) request.getSession().getAttribute("user");
        	// 2.判断用户是否已经登录 
        	if(user == null){
        		// 如果用户没有登录，则设置提示信息，跳转到登录页面
        		 System.out.println("AuthorizationInterceptor拦截请求：");
        		 request.setAttribute("message", "请先登录再访问网站");
        		 request.getRequestDispatcher("loginForm").forward(request, response);
        	}else{
        		// 如果用户已经登录，则验证通过，放行
        		 System.out.println("AuthorizationInterceptor放行请求：");
        		 flag = true;
        	}
        }
        return flag;	
	}
}
```

在springmvc-config中的配置

```
<mvc:interceptors>
	<mvc:interceptor>
		<mvc:mapping path="/*"/>
		<!-- 使用bean定义一个Interceptor，直接定义在mvc:interceptors根下面的Interceptor将拦截所有的请求 -->  
	 	<bean class="org.fkit.interceptor.AuthorizationInterceptor"/>
	</mvc:interceptor>
</mvc:interceptors>
```


# SpringMVC基础

## 1、SpringMVC简介



## 2、SpringMVC开发基本流程

#### 2.1、原始SpringMVC开发基础流程

- 1、新建一个web项目

- 2、导入指定的jar包

- 3、在web.xml中配置前端控制器

  ~~~xml
  <!-- 配置SpringMVC的前端控制器 -->
  <servlet>
  	<servlet-name>springmvc</servlet-name>
  	<servlet-class>
          org.springframework.web.servlet.DispatcherServlet
      </servlet-class>
      <init-param>
          <!-- contextConfigLocation 是参数名，该参数的值是包含SpringMVC的配置文件路径 
       名字时固定的，不能随便写
     -->	
          <param-name>contextConfigLocation</param-name>
          <param-value>/WEB-INF/applicationContext.xml</param-value>
      </init-param>
      <!--  
     		配置当前Servlet被Tomcat加载的优先级，数字越低，优先级越高
    	-->
      <load-on-startup>1</load-on-startup>
  </servlet>
  
  <servlet-mapping>
      <servlet-name>springmvc</servlet-name>
      <url-pattern>/</url-pattern>  <!-- 所有的请求都先被DispatcherServlet拦下 -->
  </servlet-mapping>
  ~~~

- 4、书写一个控制器类实现 Controller

  ~~~java
  public class HelloController implements Controller{
  	@Override
  	public ModelAndView handleRequest(HttpServletRequest req,
  			HttpServletResponse resp) throws Exception {
          
  		String str = "Hello World";
  		Map<String,String> map=new HashMap<>();
  		 
  		map.put("param", str);
  		ModelAndView mav = new ModelAndView("hello",map);
  		return mav;
  	}
  }
  ~~~

- 5、在Spring的配置文件中进行配置

  ~~~xml
  <!--配置处理器Handler，这里的处理者就是 HelloController类 -->
  <bean id="helloController" class="cn.xiehe.controller.HelloController" />
  
  <!-- 配置处理器映射器HandlerMapping ，通过该组件，可以将用户对应请求映射到指定的Controller组件上 -->
  <bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
      <property name="mappings">
          <props>
              <prop key="/hello">helloController</prop>
          </props>
      </property>
  </bean>
  
  <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  		<!-- 配置视图解析器的前缀和后缀，前缀为路径，后缀为文件类型名 -->
  		<property name="prefix" value="/WEB-INF/" />
  		<property name="suffix" value=".jsp" />
  </bean>
  ~~~

- 6、运行项目，输入地址测试

  

#### 2.2、基于注解的SpringMVC开发

- 1、新建一个web项目

- 2、导入指定的jar包

- 3、在web.xml中配置前端控制器

- 4、==书写一个普通的java类，在该类上添加 @Controller注解，在方法上添加@RequestMapping注解

  ~~~java
  @Controller
  public class HelloController2{
  	@RequestMapping("/hello2")
  	public String toHello() {
  		return "hello";
  	}
  }
  ~~~

- 5、Spring中配置

  ~~~xml
   <!-- 配置视图解析器-->
  <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
      <!--配置是视图的后缀名-->
      <property name="suffix" value=".jsp"></property>
  
      <!-- 通过前缀 配置视图的路径 -->
      <property name="prefix" value="/WEB-INF/"></property>
  </bean>
  
  <!-- 启动springmvc  -->
  <mvc:annotation-driven/>
  <!-- 包扫描-->
  <context:component-scan base-package="cn.xiehe.controller"/>
  ~~~

- 6、运行测试



## 3、@Controller 和 @RequestMapping注解

- **3.1、@Controller注解：**

  该注解主要标记在一个类上，被它标记的类会被Springmvc自动识别为Controller对象。

- **3.2、@RequestMapping注解：**

  该注解用来指定用户请地址 和 Controller控制器中方法之间的映射关系，

  该注解主要添加在当前Controller类的方法上，也可以添加类上。

  ~~~java
  // 通过@Controller注解将当前类变成一个控制器类
  @Controller
  //@RequestMapping作用在类上，一般表示初步的请求映射信息，比如如下请求user资源路径下的一系列资源操作
  @RequestMapping("/user")
  public class UserController{
  	 /**
  	  *@RequestMapping定义在方法，表示具体的请求处理方法映射关系
  	  *
        *	@RequestMapping中的七个属性
        *    1、value：默认属性，用来指定请求路径，只给一个值的时候，可省略value属性名 （重点）
        *    2、method： 指定请求提交的方式      （重点）
        		  值可以是：   RequestMethod.GET     当前请求必须是一个get请求
        		  			 RequestMethod.POST    当前请求必须是一个post请求
        		  			 RequestMethod.DELETE
        		  			 RequestMethod.PUT
        *    3、params:  	用来指定请求包含某个内容时才进行处理
        *    4、headers：   包含指定的请求头才进行处理
        *    5、name： 		给映射地址取别名,相当于方法的注释，使方法更易理解    （了解）
        *    6、produces: 	指定返回的类型				（了解）
        *    7、consumes: 	用来指定请求数据内容的类型  	（了解）
  	  */
  	@RequestMapping("/getUserInfo")
  	public String test01(Map<String, Object> map) {
  		User user = new User("xyz",19,"13621056@qq.com");
          map.put("userInfo",user)
  		return "userInfoPage";
  	}
      
      //请求必须是get请求
      @RequestMapping(value="/test02",method=RequestMethod.GET)
  	public String test02() {
  		return "success";
  	}
      
      /**
       * 在请求时，必须包含age参数才允许被执行
       */
      @RequestMapping(value="/test03",params="age")
  	public String test03() {
  		return "success";
  	}
      
      /**
       * 在请求时，必须包含age=18参数才允许被执行
       */
      @RequestMapping(value="/test04",params="age=18")
  	public String test04() {
  		return "success";
  	}
  }
  ~~~



## 4、SpringMVC 实现前后端数据交互

#### 4.1、Springmvc获取前端提交数据（三种方式）

- **1、直接通过HttpServletRequest获取**

  ~~~java
  /**
   * 1：   
   *     通过HttpServletRequest对象提供的getParameter() 获取前端值
   * 	   直接在参数列表中写上HttpServletRequest 参数即可 ， Spring自动注入
   */
  @RequestMapping("/getParamter1")
  public void getParamter1(HttpServletRequest req) {
      String data = req.getParameter("data1");
      System.out.println("data: " + data );
  }
  ~~~

- **2、使用SpringMVC自动机机制封装成Bean对象**

  ~~~java
  /**
   * 2、使用自动机制封装成Bean属性实例来接收参数
   *		需要先定义一个实体类，属性名和form表单组件的name相同，
   */
  @RequestMapping("/getParamter2")
  public void getParamter2(User user) {
  	System.out.println("user: " + user );
  }
  ~~~

- **3、@RequestParam  和  @PathVariable 注解**

  - （1）、@RequestParam用于获取参数中的数据

    ~~~java
    /**
    	1、Springmvc 会自动的将请求地址中的参数的值赋值到对应的形参中，
    	形参名   和  请求路径中 的参数名一致
    */
    @RequestMapping("/getParamter3")
    public void getParamter3(@RequestParam String  data) {
    		System.out.println("data: " + data );
    }
    
    
    /**
    	2、当形参名 和 请求中的参数名不一致时，需要我们手动指定
    */
    @RequestMapping("/getParamter4")
    public void getParamter4(@RequestParam("data") String  datax) {
    		System.out.println("datax: " + datax );
    }
    
    
    /**
    	3、在进行赋值值，其还会进行数据类型的自动转换，
    	转换失败，则抛出对应的异常
    */
    @RequestMapping("/getParamter5")
    public void getParamter5(@RequestParam("data") int i) {
    		System.out.println("i: " + i );
    }
    
    /**
    	4、但是在传递参数的时候如果是url?userName=zhangsan&userName=wangwu时，
    		即两个同名参数，前台传递了两个一样的参数，可用如下方式
    */
    @RequestMapping("/getParamter5")
    public void getParamter6(@RequestParam(value="userName") String []  userNames) {
    	System.out.println(Arrays.toString(userNames) );
    }
    //或者
    public String getParamter7(@RequestParam(value="userName") List<String> list) {
        System.out.println(Arrays.toString(userNames) );
    }
    ~~~

    ~~~html
    <a href="getParamter3?data=1">测试@RequestParam注解接收参数1</a>
    <a href="getParamter4?data=1">测试@RequestParam注解接收参数2</a>
    <a href="getParamter5?data=1">测试@RequestParam注解接收参数3</a>
    ~~~

  - （2）、@PathVariable 

    ==其也可以接受路径中的参数值，但是要事先指定占位符，其主要是用来配合restFul 风格的路径==

    ~~~java
    //使用@PathVariable接收参数，参数值需要在url进行占位
    @RequestMapping("/edit/{id}/{name}")
    public void edit(@PathVariable long id,@PathVariable String name) {
        System.out.println("id: " + id );
        System.out.println("name: " + name );
    }
    ~~~

    ~~~html
    <a href="edit/1/cf">测试@PathVariable接收参数</a>
    ~~~

    

#### 4.2、SpringMVC 向前端响应数据（三种方式）

- **1、直接使用HttpRequestServlet 和 Session**

  ~~~java
  @RequestMapping(“/login”)
  public String  checkLogin (@RequestParam String userName,
                             @RequestParam String pswd,
                             HttpServletRequest req){
      
  	UserBean user=userService.login(userName ,pswd );
      if(user!=null){
          req.setAttribute("loginMesg", "登录成功");
  		req.getSession().setAttribute(“loginuser”,user);
  		return “userCenter“；
      }else{
          req.setAttribute("loginMesg", "登录失败");
          return “login“；
      }
  }
  ~~~

- **2、使用map，Model，ModelMap, ModelAndView实现**

  ①、map，Model，ModelMap都只封装要传递的数据，不指定页面

  ②、ModelAndView 需要封装传递的数据 和  显示数据的页面

  ~~~java
  //使用Map向页面传值，Springmvc会自动将map中的数据存放在request作用域中
  @RequestMapping("/sendMessage")
  public String sendMessage(Map<String, String> map){
      map.put("key1", "value-1"); 
      map.put("key2", "value-2"); 
      return "messagePage";
  }
  
  /**使用Model和ModelMap向页面传值：
  	在Controller处理方法中添加ModelMap或者Model参数，
  	两个作用效果都一样，Model 是一个接口，继承了ModelMap类。ModelMap也是继承了Map的。
  */
  @RequestMapping("/sendMessage1")
  public String  sendMessage1(ModelMap model){
  	UserBean user=userService.login(userName ,password );
  	// Model 的用法在这里也一样的，并且在他两中里也没法设定要跳转的jsp地址
  	model.addAttribute(“user”,user);
  	return "userInfoMessage";
  }
  
  
  /**
  	使用ModelAndView对象向前端传值 
  */
  @RequestMapping("/sendMessage2")
  public ModelAndView  checkLogin(){
  	Map<String,Object> data=new HashMap<>();
  	data.put("message","10");
  	return new ModelAndView("messagePage",data)；;
  }
  ~~~

- **3、@ModelAttribute和@SessionAttributes**

  - **1、@ModelAttribute注解**

    ​	该注解添加在方法上时，在我们调用当前Controller的任意@RequestMapping 方法之前，会先执行

    @ModelAttribute注解的方法

    ~~~java
    /*
    	@ModelAttribute注解应用场景举例：
    		假设我们现在要做一个修改用户基础数据的操作，
    		但是在修改数据的时候，可能有些数据有提交，有些数据没有，比如密码就不会有，
    		
    		此时我们去修改，用User去去接受前端数据，因为密码没有，
    		那么在修改数据时可能就会把密码该成空的了。
    		
    		那我们如果在改之前把原始数据从数据库中取出来存放到一个对象中，
    		然后用户提交的数据也封装到这个对象中，
    		只不过封装时，如果属性值不一样的，就修改掉，
    		如果为空 或者  属性值一样的，就采用原始的从数据库中取出的数据，
    		那么这个问题就得到的解决
    */
    
    //@ModelAttribute的方法会在下面的@RequestMapping方法执行前执行
    @ModelAttribute
    public void getUserInfo(ModelMap modelMap){
        System.out.println("先执行@ModelAttribute注解的方法");
        
        // 模拟从数据库中取出了数据
        User user = new User("cf","123456",19,"成都","1234526@qq.com");
        //将数据存入Mdoel模型中
        modelMap.put("user",user);
    }
    
    /*
    	整个代码执行过程：
    		1、用户发起请求，请求当前方法
    		2、当前方法执行前，先执行@ModelAttribute的方法
    		3、@ModelAttribute的方法获取数据存入到Spring的Mdoel模型中
    		4、然后执行当前方法，此时因为Mdoel中已经有了User对象，所以不会新建，
    		  直接从模型中取出，直接封装数据。
    */
    @RequestMapping("/udapteUserInfo")
    public void update(User user){
        System.out.println(user);
    }
    
    
    
    
    //@ModelAttribute注解在参数前，与@RequestParam类似
    @RequestMapping("/testModelAttribute")
    public void update(@ModelAttribute Strintg data){
        System.out.println(data);
    }
    ~~~

    ~~~html
    <form action="udapteUserInfo" method="post">
        <div>用户名：<input type="text" name="userName"></div>
        <div>年龄：<input type="text" name="age"></div>
        <div>地址：<input type="text" name="address"></div>
        <div>邮箱：<input type="text" name="email"></div>
        <div><input type="submit" value="修改"></div>
     </form>
    ~~~

  - **2、@SessionAttributes**

    该注解是注释在类上，然后Springmvc会将数据 放到HttpSession中。

    ~~~java
    /*
    	此时Springmvc 会即将模型中的数存入到request作用域中，也会存入到session中,
    	如果没有向模型中存入数据这个动作，会抛出相应的异常
    */
    @SessionAttributes({"user"})
    @Controller
    public class UserController {
        @RequestMapping("/getUserInfo")
        public String getUserInfo(ModelMap modelMap){
        	User user = new User("cf","123456",19,"成都","1234526@qq.com");
       	 	//将数据存入Mdoel模型中
        	modelMap.put("user",user);
            return "test";
    	} 
        
        /*
          清除@SessionAttributes向session中添加的值
        		如果需要清除通过@SessionAttribute添加至 session 中的数据，
        		则需要在controller 的 handler method中添加 SessionStatus参数，
        		在方法体中调用SessionStatus#setComplete。
        		
         注： 此时清除的只是该Controller通过@SessionAttribute添加至session的数据
         	（当然，如果不同controller的@SessionAttribute拥有相同的值，则也会清除）
        */
        @RequestMapping("/removeSessionAttributes")
        public String removeSessionAttributes(SessionStatus sessionStatus){
            sessionStatus.setComplete();
            return "result";
        }
    }
    ~~~



## 5、Springmvc解决解决中文乱码问题

只需在web.xml中要添加Spring提供的编码过滤器即可

~~~xml
<!--通过Spring官方提供的过滤器解决乱码问题  -->
<filter>
   <filter-name>encodingFilter</filter-name>
   <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <!--encoding变量名是固定的-->
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
</filter>
<filter-mapping>
     <filter-name>encodingFilter</filter-name>
     <url-pattern>/*</url-pattern>
</filter-mapping>
~~~



## 6、Springmvc 实现转发和重定向

在SpringMVC中默认采用转发的方式定位视图，如想实现重定向，可以路径前加redirect前缀

~~~java
@RequestMapping("/test")
public String test(SessionStatus sessionStatus){
    sessionStatus.setComplete();
    return "redirect:testPage";
}
~~~

注： Springmvc中的重定向 / 表示从当前项目路径下找起

​		 不加则从将当前路径后一个资源符替换 

## 7、Springmvc拦截器

​	拦截器是SpringMVC中强大的控件，它可以使得一个请求在进入处理器之前做一些操作，或者在处理器执行了以后进行操作，甚至是在整视图已经被渲染完毕以后再做操作。

​	**拦截器的实现方式：**

- 1、第一种是实现HandlerInterceptor接口；
- 2、第二种是实现WebRequestInterceptor接口。

~~~java
/**
  通过实现HandlerInterceptor接口 实现自定义拦截器，实现了以后，需要我们实现其中的三个方法
*/
public class SessionInterceptors implements HandlerInterceptor{
    
    /**
    	此方法是在前端控制器收到请求以后，配置有拦截器则会先调用拦截器的preHandle方法，
    	如果该方法的返回值true，表示可以继续向后调用
    	如果返回值为false，表示中断请求（不再向后调用）。
    	后续方法的执行与否都建立在preHandle() 是否返回为true之上。
    */
    public boolean preHandle(HttpServletRequest request, 
                             HttpServletResponse response, 
                             Object handler)throws Exception {
        HttpSession session=request.getSession();
        Object obj=session.getAttribute("admin");
        if(obj==null){    //为空表示未登录，重定向到登录页面
            response.sendRedirect("tologin.do");
            return false;
        }
        return true;//已经登录，可以继续后面的请求
    }
    
    
    /**
    	该方法的作用是进行处理器拦截用的，它的执行时间是在处理器进行处理之 后，也就是在						Controller的方法调用之后执行，但是它会在DispatcherServlet进行视图的渲染之前执					行，也就是说在这个方法中你可以对ModelAndView进行操作。
    */
	public void postHandle(HttpServletRequest  request, 
                           HttpServletResponse  response, 
                           Object handler,
                           ModelAndView modelAndView) throws Exception {
		
	}
    
    //整个请求结束以后执行。可进行一些资源清理工作
    public void afterCompletion(HttpServletRequest request, 
                                HttpServletResponse  response, 
                                Object handler, 
                                Exception ex)throws Exception {
		
	}
}
~~~

在Springmvc.xml配置文件中对拦截器进行配置

~~~xml
<!-- 配置拦截器 如果有多个拦截器，则依据配置的先后顺序来分别执行-->
<mvc:interceptors>
    <mvc:interceptor>
        <!-- 指定拦截哪些请求,                
			如  path="/**" 表示拦截所有请求                  
 				path="/user/**" 表示拦截user路径下的所有请求             -->
        <mvc:mapping path="/**"/>  
        <mvc:exclude-mapping path="/login/*"/   <!--放过某些请求-->
        <bean class="com.gok.interceptors.SomeInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
~~~

3、拦截器和过滤器比较 

​	（1）、相似点： 二者都是AOP思想（面向切面编程）的体现，都能实现权限检查，日志记录等功能 

​	（2）、不同点： 

​			1）、过滤器是Servlet规范当中定义的特殊的类，拦截器是Spring框架提供的一种特殊 的类 		

​			2）、拦截的深度不同，过滤器Filter只在Servlet前后起作用，而拦截器可以深入到方法 的前后，异常抛出					  前后等，因此拦截器有更大的弹性，故而在Spring架构程序里优先使 用拦截器。 

​			3）、前者只能用在web程序里，拦截器既可以用在web 也可以用在Application中。

## 8、Springmvc 异常处理

​	当我们书写的web程序出现异常时，比如空指针异常，如果我们不坐任何处理，则程序会将比较难看的异常信息输出到用户的浏览器界面，影响用户的使用体验，此时我们需要对这些异常进处理，那就要用到Springmvc中的异常处理机制，在Springmvc中提供了三种方式进行异常处理：

- **1、使用SpringMVC提供的简单异常处理器（只需要在Springmvc.xml 文件配置即可）**

  通过SimpleMappingExceptionResolver可以将不同的异常映射到不同的jsp页面（通过exceptionMappings属性的配置），同时我们也可以为所有的异常指定一个默认	的异常提示页面（通过defaultErrorView属性的配置），如果所抛出的异常在exceptionMappings中没有对应的映射，则Spring将用此默认配置显示异常信息。

  ~~~xml
  <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
  	<!--定义默认的异常处理页面，当该异常类型的注册时使用-->
      <property name="defaultErrorView" value="error"></property>
      <property name="exceptionMappings">
          <props>
              <!-- key:异常类别（非全称),   值为视图名称 -->
              <prop key="java.lang.Exception">error</prop>
          </props>
      </property>
  </bean>
  ~~~

- **2、实现HandlerExceptionResolver接口自定义异常处理器**

  ~~~java
  public class myException implements  HandlerExceptionResolver {
      public ModelAndView resolveException(HttpServletRequest request,  
                                           HttpServletResponse response,
                                           Object object, Exception exception) {  
          Map<String Object> data=new HashMap<String Object>();
          data.put(“exception”,exception);
          // 根据不同类型异常返回不同视图
          return new ModelAndView（“error”,data）；
      }
  }
  ~~~

  在Springmvc.xml进行配置

  ~~~xml
  <bean class="com.qunar.advertisement.exception.QADHandlerExceptionResolver"></bean> 
  ~~~

- 3、使用@ExceptionHandler注解实现异常处理（最常使用）

  直接将@ExceptionHandler放在Controller中的自定义的异常处理方法上，如果该Controller组件出现异常，则直接按约定的异常方法处理。

  ~~~java
  /**
   *通过 @ExceptionHandler注解实现对异常的处理
   */
  @Controller
  public class TestException1 {
  
      @RequestMapping("testException")
      public String test(){
          int i = 10 / 0;
          return "success";
      }
  
      /*
       *@ExceptionHandler 添加在方法上以后，
       * 只有当前Controller中出了对应的异常（比如此处的ArithmeticException算术异常），
       * 就会此方法处理。
       *
       * 注：现在只能处理当前Controller中的异常，
       *    别的Controller出了异常，管不到。
       */
      @ExceptionHandler(ArithmeticException.class)
      public String exceptionHandler(){
          return "error";
      }
  }	
  ~~~

  ~~~java
  /**
   * 定义全局异常异常处理类，处理当前项目中的所有异常
   *
   * @ControllerAdvice 将当前类中定义的功能（方法），
   *  增强给当前项目中的所有的Controller
   */
  @ControllerAdvice
  public class GlobalExceptionHandler {
  
      @ExceptionHandler(Exception.class)
      public String globalExceptionHandler(){
          return "error";
      }
  }
  ~~~

  ​	在上述异常处理的过程中，我们通过Springmvc提供的异常处理机制实现了程序出现问题以后，跳转到指定的错误页面，但是异常的相关信息被流失掉了，并没有记录下来，而开发人员在修复程序时，往往需要这些信息，所有需要将这些异常信息显示出来。在以前我们都是通过异常提供的printStackTrace() 将异常信息打印到控制台上，但是实际项目部署后，这种操作是做不到的，所以我们程序日志就成了我们很重要的手段。

  

## 9、Log4j日志

#### 1、简介：

​	Log4j是[Apache](https://baike.baidu.com/item/Apache/8512995)的一个开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是[控制台](https://baike.baidu.com/item/控制台/2438626)、文件、[GUI](https://baike.baidu.com/item/GUI)组件，甚至是套接口服务器、[NT](https://baike.baidu.com/item/NT/3443842)的事件记录器、[UNIX](https://baike.baidu.com/item/UNIX) [Syslog](https://baike.baidu.com/item/Syslog)[守护进程](https://baike.baidu.com/item/守护进程/966835)等；我们也可以控制每一条日志的输出格式；通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。最令人感兴趣的就是，这些可以通过一个[配置文件](https://baike.baidu.com/item/配置文件/286550)来灵活地进行配置，而不需要修改应用的代码。



#### 2、应用场景

​	2.1、监视代码中变量的变化情况，周期性的记录到文件中供其他应用进行统计分析工作；  

​	2.2、跟踪代码运行时轨迹，作为日后审计的依据；

​	2.3、担当集成开发环境中的调试器的作用，向文件或控制台打印代码的调试信息。调试Java代码更加清晰

​	2.4、记录程序的异常信息



#### 3、Log4j 简单实现

- （1）、导入jar包 

  log4j-1.2.17.jar

- （2）、书写 log4j.properties（文件名固定） 配置文件（如下为 把Debug信息输出到控制台和本地文件）

  ~~~properties
  log4j.rootLogger=DEBUG, Console ,File  
     
  #Console  
  log4j.appender.Console=org.apache.log4j.ConsoleAppender  
  log4j.appender.Console.layout=org.apache.log4j.PatternLayout  
  log4j.appender.Console.layout.ConversionPattern=%d [%t] %-5p [%c] - %m%n
     
  #File
  log4j.appender.File = org.apache.log4j.FileAppender
  log4j.appender.File.File = D://log.log
  log4j.appender.File.layout = org.apache.log4j.PatternLayout
  log4j.appender.File.layout.ConversionPattern =%d [%t] %-5p [%c] - %m%n
  ~~~

- （3）、书写java代码

  ~~~java
  /**
   * Log4j测试类
   * @author user
   */
  public class Test {
   
      private static Logger logger=Logger.getLogger(Test.class); // 获取logger实例
       
      public static void main(String[] args) {
          log.info("普通信息");
          log.debug("debug");
          log.warn("警告信息");
  		log.error("错误信息");
          log.fatal("严重错误信息");
      }
  }
  ~~~



#### 4、log4j配置文件简介

- **4.1、使用配置主要配置哪些内容？**

  - （1）rootLogger：根配置，配置如何处理日志，整个配置文件中只能有一个。
  - （2）appender：日志输出目的地，可以同时指定多个输出目的地，
  - （3）layout：日志格式化，负责对输出的日志格式化（以什么形式展现）

  ~~~properties
  # log4j的根配置rootLogger（只能有一个），
  #  语法： log4j.rootLogger =  level, appenderName1，appenderName2 ,。。。
      #  level表示的日志的等级：
          #   log4j中给出了 如下 7个日志级别  OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL
          #   Log4j建议只使用四个级别，优先级从高到低分别是ERROR、WARN、INFO、DEBUG
      # appenderName1：   表示的是日志输出目的地，一般自定义 ，方便我们在后面的配置中引用
  
  #表示将ERROR、WARN、INFO、DEBUG 这个几个级别的日志信息   输出到控制台 和 文件中
  #log4j.rootLogger=DEBUG, Console ,File
  
  #表示将ERROR、WARN、INFO这个几个级别的日志信息   输出到控制台 和 文件中 （常用）
  log4j.rootLogger=INFO, Console ,File  
  
  #表示将ERROR这个个级别的日志信息   输出到控制台 和 文件中
  #log4j.rootLogger=ERROR, Console ,File，DailyRollingFile ,RollingFile
  
  
  
  #Console  输出目的地为控制台时的配置------------------------------------------------------
  
  # appender ：日志信息的输出目的地：
      #  语法： log4j.appender. 自定义appenderName = appender全类名
          #常见的appender有： org.apache.log4j.ConsoleAppender（控制台）
          #org.apache.log4j.FileAppender（文件）
          #org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件）
          #org.apache.log4j.RollingFileAppender（（文件大小到达指定尺寸的时候产生一个新的文件）
          #org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方）
  log4j.appender.Console=org.apache.log4j.ConsoleAppender  
  
  # layout ： 日志输出样式
      # 常见的样式有： org.apache.log4j.HTMLLayout（以HTML表格形式布局）
      # org.apache.log4j.PatternLayout（可以灵活地自定义指定布局模式）
      # org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串
      # org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等等信息），
  log4j.appender.Console.layout=org.apache.log4j.PatternLayout
  
  # 当使用PatternLayout布局以后候，需要通过ConversionPattern对打印的数据进行设置。
  # ConversionPattern参数值及含义如下：
      #1、-X号: X信息输出时左对齐；
      #2、%p: 输出日志信息优先级，即DEBUG，INFO，WARN，ERROR，FATAL,
      #3、%d: 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，
      	#比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921
      #4、%c: 输出日志信息所属的类目，通常就是所在类的全名
      #5、%t: 输出产生该日志事件的线程名
      #6、%l: 输出日志事件的发生位置，相当于%C.%M(%F:%L)的组合,包括类目名、发生的线程，以及行数。
      	#举例：Testlog4.main(TestLog4.java:10)
      #7、%%: 输出一个”%”字符
      #8、%F: 输出日志消息产生时所在的文件名称
      #9、%L: 输出代码中的行号
      #10、%m: 输出代码中指定的消息,产生的日志具体信息
      #11、%n: 输出一个回车换行符，Windows平台为”\r\n”，Unix平台为”\n”输出日志信息换行
      #可以在%与模式字符之间加上修饰符来控制其最小宽度、最大宽度、和文本的对齐方式。
  log4j.appender.Console.layout.ConversionPattern=%d [%t] %-5p [%c] - %m%n
  
  
  
  #File，输出到日志文件中------------------------------------------------------------------
  log4j.appender.File = org.apache.log4j.FileAppender
  #指定日志文件的输出路径地址
  log4j.appender.File.File = E://log/log1.log
  log4j.appender.File.layout = org.apache.log4j.PatternLayout
  log4j.appender.File.layout.ConversionPattern =%d [%t] %-5p [%c] - %m%n
  
  
  
  #DailyRollingFileAppender 输出到日志文件中（每天产生新的）---------------------------------
  log4j.appender.DailyRollingFile = org.apache.log4j.DailyRollingFileAppender
  log4j.appender.DailyRollingFile.File = E://log/log2.log
  # 配置产日志文件的频次，可以不按每天产生，值如下：
      # 1、yyyy-MM: 每月产生新文件
      # 2、yyyy-ww: 每周产生新文件
      # 3、yyyy-MM-dd: 每天
      # 4、yyyy-MM-dd-a: 每天两次
      # 5、yyyy-MM-dd-HH: 每小时
      # 6、yyyy-MM-dd-HH-mm: 每分钟
  log4j.appender.DailyRollingFile.DatePattern = yyyy-MM-dd
  log4j.appender.DailyRollingFile.layout = org.apache.log4j.PatternLayout
  log4j.appender.DailyRollingFile.layout.ConversionPattern =%d [%t] %-5p [%c] - %m%n
  
  
  
  #RollingFile  输出到日志文件中（达到指定大小产生一个新的）-----------------------------------
  log4j.appender.RollingFile = org.apache.log4j.RollingFileAppender
  log4j.appender.RollingFile.File =  E://log/log3.log
  # 指定大小。后缀可以是KB, MB 或者是 GB. 在日志文件到达该大小时
  log4j.appender.RollingFile.MaxFileSize=1KB
  #maxBackupIndex用来指定要保留的日志文件数的最大值，可以间接实现删除N天前的日志文件
  log4j.appender.RollingFile.MaxBackupIndex=10
  log4j.appender.RollingFile.layout = org.apache.log4j.PatternLayout
  log4j.appender.RollingFile.layout.ConversionPattern =%d [%t] %-5p [%c] - %m%n
  ~~~

  

#### 5、通用配置，记录不同级别的信息

~~~properties
#root日志
log4j.rootLogger=INFO,stdout,info,warn,error

#控制台日志
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %-5p %c{1}:%L - %m%n

#info级别日志
log4j.appender.info=org.apache.log4j.RollingFileAppender
#表示当前文件只打印info级别的信息
log4j.appender.info.Threshold=INFO
#日志文件的输出地址
log4j.appender.info.File=E://log/info.log
#当日志文件满200M的时候就新建一个日志文件
log4j.appender.info.MaxFileSize=200MB
log4j.appender.info.MaxBackupIndex=5
log4j.appender.info.layout=org.apache.log4j.PatternLayout
log4j.appender.info.layout.ConversionPattern=%d %-5p %l - %m%n
#过滤器，按照日志等级过滤打印信息，如果不添加，则会将INFO以上级别的信息都打印到当前文件中
log4j.appender.info.filter.infoFilter = org.apache.log4j.varia.LevelRangeFilter
log4j.appender.info.filter.infoFilter.LevelMin=INFO
log4j.appender.info.filter.infoFilter.LevelMax=INFO

#warn级别日志
log4j.appender.warn=org.apache.log4j.RollingFileAppender
log4j.appender.warn.Threshold=WARN
log4j.appender.warn.File=E://log/warn.log
log4j.appender.warn.MaxFileSize=200MB
log4j.appender.warn.MaxBackupIndex=5
log4j.appender.warn.layout=org.apache.log4j.PatternLayout
log4j.appender.warn.layout.ConversionPattern=%d %-5p %l - %m%n
log4j.appender.warn.filter.warnFilter = org.apache.log4j.varia.LevelRangeFilter
log4j.appender.warn.filter.warnFilter.LevelMin=WARN
log4j.appender.warn.filter.warnFilter.LevelMax=WARN

#error级别日志
log4j.appender.error=org.apache.log4j.RollingFileAppender
log4j.appender.error.Threshold=ERROR
log4j.appender.error.File=E://log/error.log
log4j.appender.error.MaxFileSize=200MB
log4j.appender.error.MaxBackupIndex=5
log4j.appender.error.layout=org.apache.log4j.PatternLayout
log4j.appender.error.layout.ConversionPattern=%d %-5p %l - %m%n
log4j.appender.error.filter.errorFilter = org.apache.log4j.varia.LevelRangeFilter
log4j.appender.error.filter.errorFilter.LevelMin=ERROR
~~~

~~~java
@ControllerAdvice
public class GlobalExceptionHandler {

    private static Logger log = Logger.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(Exception.class)
    public String globalExceptionHandler(HttpServletRequest request,
                                         HttpServletResponse response,
                                         Object obj,
                                         Exception ex){
        log.error("================ start 异常记录==================================");
        String url_error = "http://" + request.getServerName() // 服务器地址
                + ":" + request.getServerPort() // 端口号
                + request.getContextPath() // 项目名称
                + request.getServletPath() // 请求页面或其他地址
                + "?" + (request.getQueryString()); // 参数
        log.error("错误的来源完整异常地址：" + url_error);
        Map<String, String[]> pars = request.getParameterMap();
        Set<String> parameterKey2 = pars.keySet();
        Iterator<String> iterator2 = parameterKey2.iterator();
        while (iterator2.hasNext()) {
            String tKey = iterator2.next();
            String tValue = pars.get(tKey)[0];
            log.error("request提交参数key:" + tKey + " , value值：" + tValue);
        }

        log.error(ex.getMessage(), ex);// 可以把所有的异常堆栈信息打印出来
        log.error("================ end 异常记录==================================");
        return "error";
    }
}
~~~





## 10、Springmvc实现文件上传

 SpringMVC为文件上传提供了良好的支持。首先SpringMVC的文件上传是通过 MultipartResolver（Multipart解析器）处理的，它是一个接口，其下有两个实现类，也分别代表了两 种实现方式：

（1）、CommonsMultipartResolver实现文件上传： 该方式依赖于Apache下的commonsFileupload项目来处理multipart请求，所以在使用时，必须 要引入相应的jar包commons-io-2.6.jar 和commons-ﬁleupload-1.3.3.jar 具体实现步骤如下

​	 ① 、在Spring配置文件中进行如下配置：

```xml
<bean id="multipartResolver"                                class="org.springframework.web.multipart.commons.CommonsMultipartResol ver">        <property name="defaultEncoding" value="utf-8" />    限制文件上传大小最大为1MB      <property name="maxUploadSize" value="1048576" />
</bean>
```

​		②、书写form表单，一定要将其指定为多媒体类型，否则上传失败

```xml
	<form action="fileUp" method="post" enctype="multipart/form-data">            		请选择文件：<input type="file" name="imagefile">                        
       	 </br>                    
        <input type="submit" value="提交"> 
    </form>
```

​		③、书写Controller组件进行文件上传（这里实现的是单文件上传）

```java
/** 
 * 用来处理文件上传的控制器类 
 *    使用    commons-io-2.6.jar和commons-fileupload-1.3.3.jar 实现单文件上传 
 */ 
@Controller 
public class FileUpController {
@RequestMapping("/fileUp")    //一个文件会被封装在MultipartFile对象中    
    public String fileUpLoad(HttpServletRequest req,@RequestParam(value = 		"image") MultipartFile mf) {                
        //获取项目的真实路径        
        String path = req.getServletContext().getRealPath("/");        				System.out.println("path: " + path);                        
        File file=new File(path);        
        if(!file.exists()){                 
            file.mkdir(); //判断是否有指定的文件夹，没有则创建一个        
        }                  
        //获取文件的真实名字        
        String originalFilename = mf.getOriginalFilename();                
        File targerFile = new File(path + File.separator + originalFilename );                try { 
            //将文件输出指定的位置            
            mf.transferTo(targerFile);                    
        } catch (IllegalStateException e) {            
            e.printStackTrace();        
        } catch (IOException e) {           
            e.printStackTrace();        
        }                
        req.getSession().setAttribute("successMessage", "文件上传成功");        			return "success";            
    } 
}

```
（2）、StandarServletMultipartResolver实现文件上传 它是Spring3.1以后的产物，不需要引入其他jar包，就可以实现文件上传操作，但是该种方式要依 赖Servlet3.1及其以上的容器才可以能实现。 步骤具体如下： 

​		① 、书写Springmvc.xml配置文件

```xml
<bean id="multipartResolver"    class="org.springframework.web.multipart.support.StandardServletMultipa rtResolver"> </bean>`
```

​		②、在web.xml中进行配置，即在前端控制器中添加multipart-conﬁg 标签

```xml
<servlet>     
    <servlet-name>springmvc</servlet-name>          
    <servlet-class>              													org.springframework.web.servlet.DispatcherServlet       
    </servlet-class>     
    <multipart-config>        
        <location>D:/</location>   <!--临时文件的目录-->        
        <max-file-size>2097152</max-file-size>    <!-- 上传文件最大2M -->       			<!-- 上传文件整个请求不超过4M -->            
        <max-request-size>4194304</max-request-size>          
    </multipart-config> 
</servlet>
```

​		③、书写form表单，演示多文件上传

```jsp
<form action="fileUp2" method="post" enctype="multipart/form-data">           	  <input type="file" name="image1">        
    <br>        
    <input type="file" name="image2">        
    <br>        
    <input type="submit" value="提交"/> 
</form>
```

​		④、书写控制器类进行处理（这里是多文件上传）

```java
/** 
* 使用Spring官方提供的StandarServletMultipartResolver 进行多文件上传 
* 
*/ @Controller public class FileUpController2 {        
    /*     
    * MultipartHttpServletRequest  在其中封装着多个文件对象     
    */    @RequestMapping("/fileUp2")    
    public String fileUpLoad(MultipartHttpServletRequest request) {                //获取项目的真实路径        
        String path = request.getServletContext().getRealPath("/image2");        System.out.println("path: " + path);                 
        File file=new File(path);        
        if(!file.exists()){                 
            file.mkdir();   //判断是否有指定的文件夹，没有则创建一个        
        }              
        /*         
        * 通过MultipartHttpServletRequest提供的方法获取存放多个文件的map对 象，         		 *      该map中的  key  为表单提交时，input标签中的name值         
        *              value  为具体的文件对象MultipartFile         
        */        
        Map<String, MultipartFile> multiFileMap = request.getFileMap();                
        //通过遍历MultiValueMap所有的key，获取对应 MultipartFile的文件对象
        Set<String> keySet = multiFileMap.keySet();              
        try {            
            for (String key : keySet) {                
                MultipartFile multipartFile = multiFileMap.get(key);                					String originalFilename = multipartFile.getOriginalFilename();                           File targetFile =new File(path + File.separator + originalFilename );                     //传入指定的位置                  
                multipartFile.transferTo(targetFile);            
            }        
        } catch (IllegalStateException e) {            
            e.printStackTrace();        
        } catch (IOException e) {            
            e.printStackTrace();        
        }        				
        request.getSession().setAttribute("successMessage", "文件上传成 功");       
        return "success";
    }
}
```


---
layout: post
title:  "springMVC笔记"
date:   2020-04-29 20:31:06 +0800--
categories: [Java]
tags: [JavaEE, spring, ]  

---

## 三层架构和MVC

1. 三层架构
   1. 开发服务器端程序，一般都基于两种形式，一种C/S架构程序，一种B/S架构程序 
   2. 使用Java语言基本上都是开发B/S架构的程序，B/S架构又分成了三层架构 
   3. 三层架构
      - 表现层:WEB层，用来和客户端进行数据交互的。表现层一般会采用MVC的设计模型 
      - 业务层:处理公司具体的业务逻辑的
      - 持久层:用来操作数据库的

2. MVC模型
   1. MVC 全名是Model View Controller 模型视图控制器，每个部分各司其职。
   2. Model：数据模型，JavaBean的类，用来进行数据封装。
   3. View：指JSP、HTML用来展示数据给用户
   4. Controller：用来接收用户的请求，整个流程的控制器。用来进行数据校验等。

## SpringMVC的入门案例

1. 需求

   ![image-20200429210647020](/assets/imgs/image-20200429210647020.png)

2. 入门案例的执行流程

   1. 当启动Tomcat服务器的时候，因为配置了load-on-startup标签，所以会创建DispatcherServlet对象， 就会加载springmvc.xml配置文件
   2. 开启了注解扫描，那么HelloController对象就会被创建
   3. 从index.jsp发送请求，请求会先到达DispatcherServlet核心控制器，根据配置@RequestMapping注解

   找到执行的具体方法

   4. 根据执行方法的返回值，再根据配置的视图解析器，去指定的目录下查找指定名称的JSP文件

   5. Tomcat服务器渲染页面，做出响应

![image-20200429220240757](/assets/imgs/image-20200429220240757.png)

3. SpringMVC官方提供图形

4. 入门案例中的组件分析
   1. 前端控制器(DispatcherServlet) 
   2. 处理器映射器(HandlerMapping) 
   3. 处理器(Handler)
   4. 处理器适配器(HandlerAdapter)
   5. 视图解析器(View Resolver)
   6. 视图(View)

![image-20200921103327350](/assets/imgs/image-20200921103327350-0655883.png)



## RequestMapping注解

1. `@RequestMapping`注解的作用是建立请求URL和处理方法之间的对应关系 
2.  RequestMapping注解可以作用在方法和类上
   1. 作用在类上:第一级的访问目录
   2. 作用在方法上:第二级的访问目录
   3. 细节:路径可以不编写 / 表示应用的根目录开始
   4. 细节:`${ pageContext.request.contextPath }`也可以省略不写，但是路径上不能写 `/`
3. RequestMapping的属性
   - path/value：指定请求路径的url
   - method：指定该方法的请求方式
   - params：指定限制请求参数的条件
   - headers：发送请求中必须包含的请求头

## 请求参数绑定

> /Users/silince/Develop/MagicDontTouch/IdeaProjects/Java EE/springMVC/springmvc_day01_01_start

1. 请求参数的绑定说明
   1. 绑定机制
      - 表单提交的数据都是k=v格式的 `username=haha&password=123`
      - SpringMVC的参数绑定过程是把表单提交的请求参数，作为控制器中方法的参数进行绑定的 
      - 要求:提交表单的name和参数的名称是相同的
   2. 支持的数据类型
      - 基本数据类型和字符串类型
      - 实体类型（JavaBean）
      - 集合数据类型（List、map集合等）

2. 基本数据类型和字符串类型

   1. 提交表单的name和参数名称是相同的
   2. 区分大小写

3. 实体类型（JavaBean）

   1. 提交表单的name和JavaBean中的属性名称需要一致

   2. 如果一个JavaBean类中包含其他的引用类型，那么表单的name属性需要编写成:对象.属性 例如:

      address.name

4. 给集合属性封装数据

   1. JSP页面编写方式：list[0].属性

5. 请求参数中文乱码的解决

   1. 在web.xml中配置Spring提供的过滤器类

   ```xml
    
   <!-- 配置过滤器，解决中文乱码的问题 --> <filter>
   <filter-name>characterEncodingFilter</filter-name>
   <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter- class>
   <!-- 指定字符集 --> <init-param>
   <param-name>encoding</param-name>
   <param-value>UTF-8</param-value> </init-param>
   </filter> <filter-mapping>
   <filter-name>characterEncodingFilter</filter-name>
   <url-pattern>/*</url-pattern> </filter-mapping>
   ```

   

6. 自定义类型转换器

   1. 表单提交的任何数据类型全部都是字符串类型，但是后台定义Integer类型，数据也可以封装上，说明

      Spring框架内部会默认进行数据类型转换。

   2. 如果想自定义数据类型转换，可以实现Converter的接口

      1. 自定义类型转换器

         ```java
         package cn.itcast.utils;
         import java.text.DateFormat; import java.text.SimpleDateFormat; import java.util.Date;
         import org.springframework.core.convert.converter.Converter;
         /**
         * 把字符串转换成日期的转换器 * @author rt
         */
         public class StringToDateConverter implements Converter<String, Date>{
         /**
         * 进行类型转换的方法 */
         public Date convert(String source) { // 判断
         if(source == null) {
         throw new RuntimeException("参数不能为空");
         }
         try {
         DateFormat df = new SimpleDateFormat("yyyy-MM-dd"); // 解析字符串
         Date date = df.parse(source);
         return date;
         } catch (Exception e) {
         throw new RuntimeException("类型转换错误");
         } }
         }
             
         ```

         

      2. 注册自定义类型转换器，在springmvc.xml配置文件中编写配置

         ```xml
          <!-- 注册自定义类型转换器 -->
         <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
         <property name="converters"> <set>
         <bean class="cn.itcast.utils.StringToDateConverter"/> </set>
                 </property>
             </bean>
         <!-- 开启Spring对MVC注解的支持 -->
         <mvc:annotation-driven conversion-service="conversionService"/>
         ```

7. 在控制器中使用原生的ServletAPI对象

   1. 只需要在控制器的方法参数定义HttpServletRequest和HttpServletResponse对象

## springMVC常用注解

### RequestParam

1. 作用:把请求中的指定名称的参数传递给控制器中的形参赋值
2. 属性：
   - value:请求参数中的名称
   - required:请求参数中是否必须提供此参数，默认值是true，必须提供
3. 代码如下

```java
/**
* 接收请求 * @return */
@RequestMapping(path="/hello")
public String sayHello(@RequestParam(value="username",required=false)String name) {
System.out.println("aaaa"); System.out.println(name); return "success";
}
```

### RequestBody

1. 作用:用于获取请求体的内容(注意:get方法不可以)
2. 属性:
   - required:是否必须有请求体，默认值是true
3. 代码如下

```java
 /**
* 接收请求 * @return */
@RequestMapping(path="/hello")
public String sayHello(@RequestBody String body) {
System.out.println("aaaa"); System.out.println(body); return "success";
}
```



### PathVariable

1.  作用:拥有绑定url中的占位符的。例如:url中有/delete/{id}，{id}就是占位符

2. 属性

   - value:指定url中的占位符名称

3. Restful风格的URL

   1. 请求路径一样，可以根据不同的请求方式去执行后台的不同方法

      ![image-20200503144715724](/assets/imgs/image-20200503144715724.png)

   2. restful风格的URL优点

      - 结构清晰
      - 符合标准
      - 易于理解
      - 扩展方便

4. 代码如下

```java
 <a href="user/hello/1">入门案例</a>
/**
* 接收请求 * @return */
@RequestMapping(path="/hello/{id}")
public String sayHello(@PathVariable(value="id") String id) {
System.out.println(id);
return "success"; }
```

### RequestHeader

1. 作用:获取指定请求头的值
2. 属性
   - value:请求头的名称
3. 代码如下

```java
@RequestMapping(path="/hello")
public String sayHello(@RequestHeader(value="Accept") String header) {
System.out.println(header);
return "success"; }
```

### CookieValue

1. 作用:用于获取指定cookie的名称的值
2. 属性
   - value:cookie的名称
3. 代码

```java
@RequestMapping(path="/hello")
public String sayHello(@CookieValue(value="JSESSIONID") String cookieValue) {
System.out.println(cookieValue);
return "success"; }
```

### ModelAttribute

1. 作用
   - 出现在方法上:表示当前方法会在控制器方法执行前先执行。
   - 出现在参数上:获取指定的数据给参数赋值。
2. 应用场景: 当提交表单数据不是完整的实体数据时，保证没有提交的字段使用数据库原来的数据。
3. 代码

#### 修饰的方法有返回值

```java
 /**
* 作用在方法，先执行 
* @param name
* @return
*/
@ModelAttribute
public User showUser(String name) {
System.out.println("showUser执行了..."); // 模拟从数据库中查询对象
User user = new User(); user.setName("哈哈"); user.setPassword("123"); user.setMoney(100d);
    return user;
}
/**
* 修改用户的方法
* @param cookieValue * @return
*/
@RequestMapping(path="/updateUser") 
public String updateUser(User user) {
System.out.println(user);
return "success"; }
```



#### 修饰的方法没有返回值

```java
/**
* 作用在方法，先执行 
* @param name
* @return
*/
@ModelAttribute
public void showUser(String name,Map<String, User> map) {
System.out.println("showUser执行了..."); // 模拟从数据库中查询对象
User user = new User(); user.setName("哈哈"); user.setPassword("123"); user.setMoney(100d);
map.put("abc", user); }
/**
* 修改用户的方法
* @param cookieValue 
* @return
*/
 
@RequestMapping(path="/updateUser")
public String updateUser(@ModelAttribute(value="abc") User user) {
System.out.println(user);
return "success"; }
```

### SessionAttributes

1. 作用:用于多次执行控制器方法间的参数共享
2. 属性. value:指定存入属性的名称
3. 代码

```java
@Controller
@RequestMapping(path="/user")
@SessionAttributes(value= {"username","password","age"},types=
{String.class,Integer.class}) public class HelloController {
/**
* 向session中存入值 * @return
*/
// 把数据存入到session域对象中
@RequestMapping(path="/save") public String save(Model model) {
System.out.println("向session域中保存数据"); model.addAttribute("username", "root"); model.addAttribute("password", "123"); model.addAttribute("age", 20);
return "success"; }
/**
* 从session中获取值 * @return
*/
@RequestMapping(path="/find")
public String find(ModelMap modelMap) {
String username = (String) modelMap.get("username"); String password = (String) modelMap.get("password"); Integer age = (Integer) modelMap.get("age"); System.out.println(username + " : "+password +" : "+age); return "success";
}
/**
* 清除值
* @return */
@RequestMapping(path="/delete")
public String delete(SessionStatus status) {
status.setComplete();
return "success"; }
}
  
```

## 响应数据和结果视图

### 返回值分类

1. 返回字符串

   1. Controller方法返回字符串可以指定逻辑视图的名称，根据视图解析器为物理视图的地址。

      ```java
      @RequestMapping(value="/hello") 
      public String sayHello() {
      System.out.println("Hello SpringMVC!!"); // 跳转到XX页面
      return "success";
      }
      ```

   2. 应用场景

      ```java
      @Controller 
      @RequestMapping("/user") 
      public class UserController {
      /**
      * 请求参数的绑定 */
      @RequestMapping(value="/initUpdate") 
      public String initUpdate(Model model) {
      // 模拟从数据库中查询的数据
      User user = new User(); 
      	user.setUsername("张三"); 
      	user.setPassword("123"); 
        user.setMoney(100d); 
        user.setBirthday(new Date()); 
        model.addAttribute("user", user); return "update";
      } 
      }
      
      <h3>修改用户</h3>
      ${ requestScope }
      <form action="user/update" method="post">
      		姓名:<input type="text" name="username" value="${ user.username }"><br> 
        	密码:<input type="text" name="password" value="${ user.password }"><br> 
          金额:<input type="text" name="money" value="${ user.money }"><br> 
          <input type="submit" value="提交">
      </form>
        
      ```

2. 返回值是void

   1. 如果控制器的方法返回值编写成void，执行程序报404的异常，默认查找JSP页面没有找到。默认会跳转到@RequestMapping(value="/initUpdate") initUpdate的页面。

   2. 可以使用请求转发或者重定向跳转到指定的页面

      ```java
      @RequestMapping(value="/initAdd")
      public void initAdd(HttpServletRequest request,HttpServletResponse response) throws
      Exception { 
        System.out.println("请求转发或者重定向");
      	// 请求转发
      	// request.getRequestDispatcher("/WEB-INF/pages/add.jsp").forward(request,
      response);
      	// 重定向
      	// response.sendRedirect(request.getContextPath()+"/add2.jsp");
      	response.setCharacterEncoding("UTF-8"); 			 	response.setContentType("text/html;charset=UTF-8");
      // 直接响应数据 
        response.getWriter().print("你好"); 
        return;
      }
      ```

   3. 返回值是ModelAndView对象

      1. ModelAndView对象是Spring提供的一个对象，可以用来调整具体的JSP视图

         ```java
         /**
         * 返回ModelAndView对象
         * 可以传入视图的名称(即跳转的页面)，还可以传入对象。 
         * @return
         * @throws Exception
         */
         @RequestMapping(value="/findAll")
         public ModelAndView findAll() throws Exception {
         	ModelAndView mv = new ModelAndView(); 
           // 跳转到list.jsp的页面 	
           mv.setViewName("list");
         	// 模拟从数据库中查询所有的用户信息 
           List<User> users = new ArrayList<>(); 
           User user1 = new User(); 		
           user1.setUsername("张三"); 
           user1.setPassword("123");
         	User user2 = new User(); 
           user2.setUsername("赵四");
         	user2.setPassword("456");
           users.add(user1); users.add(user2);
         	// 添加对象 mv.addObject("users", users);
         	return mv; 
         }
         
         <%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
         <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
         <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
         <html>
         <head>
         <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"> <title>Insert title here</title>
         </head>
         <body>
         	<h3>查询所有的数据</h3>
         	<c:forEach items="${ users }" var="user">
         	${ user.username } </c:forEach>
         </body>
         </html>
           
         ```

### SpringMVC框架提供的转发和重定向

1. forward请求转发.controller方法返回String类型，想进行请求转发也可以编写成

   ```java
   /**
   * 使用forward关键字进行请求转发
   * "forward:转发的JSP路径"，不走视图解析器了，所以需要编写完整的路径 * @return
   * @throws Exception
   */
   @RequestMapping("/delete")
   public String delete() throws Exception {
   	System.out.println("delete方法执行了...");
   	// return "forward:/WEB-INF/pages/success.jsp"; 
     return "forward:/user/findAll";
   }
   ```

2. redirect重定向

   1. controller方法返回String类型，想进行重定向也可以编写成

      ```java
      /**
      * 重定向
      * @return
      * @throws Exception */
      @RequestMapping("/count")
      public String count() throws Exception {
      System.out.println("count方法执行了..."); return "redirect:/add.jsp";
      // return "redirect:/user/findAll";
      }
      ```

### ResponseBody响应json数据

1. DispatcherServlet会拦截到所有的资源，导致一个问题就是静态资源(img、css、js)也会被拦截到，从而

   不能被使用。解决问题就是需要配置静态资源不进行拦截，在springmvc.xml配置文件添加如下配置

    	1. mvc:resources标签配置不过滤
         	1. location元素表示webapp目录下的包下的所有文件
         	2. mapping元素表示以/static开头的所有请求路径，如/static/a 或者/static/a/b

   ```xml
    <!-- 设置静态资源不过滤 -->
   <mvc:resources location="/css/" mapping="/css/**"/> <!-- 样式 --> 
   <mvc:resources location="/images/" mapping="/images/**"/> <!-- 图片 --> 
   <mvc:resources location="/js/" mapping="/js/**"/> <!-- javascript -->
   ```

2. 使用@RequestBody获取请求体数据

   1. 

   ```javascript
   // 页面加载
   // 页面加载 $(function(){
   // 绑定点击事件 $("#btn").click(function(){
   $.ajax({
   url:"user/testJson", contentType:"application/json;charset=UTF-8", data:'{"addressName":"aa","addressNum":100}', dataType:"json",
   type:"post",
   success:function(data){
             alert(data);
   					alert(data.addressName); }
   		}); 
   	});
   });
   
   /**
   * 获取请求体的数据 * @param body
   */
   @RequestMapping("/testJson")
   public void testJson(@RequestBody String body) {
   System.out.println(body); }
   ```

3. 使用@RequestBody注解把json的字符串转换成JavaBean的对象

4. 使用@ResponseBody注解把JavaBean对象转换成json字符串，直接响应

   ```java
   @RequestMapping("/testJson")
   public @ResponseBody Address testJson(@RequestBody Address address) {
   	System.out.println(address); 
     address.setAddressName("上海"); 
     return address;
   }
     
   ```

   

5. json字符串和JavaBean对象互相转换的过程中，需要使用jackson的jar包

   ```xml
    <dependency> 
      <groupId>com.fasterxml.jackson.core</groupId> 
      <artifactId>jackson-databind</artifactId> 
      <version>2.9.0</version>
       </dependency>
       <dependency>
   <groupId>com.fasterxml.jackson.core</groupId> 
         <artifactId>jackson-core</artifactId> 
         <version>2.9.0</version>
       </dependency>
       <dependency>
   	<groupId>com.fasterxml.jackson.core</groupId> 
         <artifactId>jackson-annotations</artifactId> 
         <version>2.9.0</version>
       </dependency>
   ```

## springMVC文件上传

![image-20200503210254910](/assets/imgs/image-20200503210254910.png)



## SpringMVC的异常处理

系统中异常包括两类:预期异常和运行时异常 RuntimeException，前者通过捕获异常从而获取异常信息， 后者主要通过规范代码开发、测试通过手段减少运行时异常的发生。

系统的 dao、service、controller 出现都通过 throws Exception 向上抛出，最后由 springmvc 前端控制器交由异常处理器进行异常处理，如下图:

![image-20200921130008896](/assets/imgs/image-20200921130008896.png)

### SpringMVC的异常处理

1.自定义异常类

```java
public class SysException extends Exception{

  private static final long serialVersionUID = 4055945147128016300L;
  // 异常提示信息
	private String message; 
  
  public SysException(String message) {
		this.message = message; 
  }
  
  public String getMessage() {
    return message;
	}
  
	public void setMessage(String message) { 
  	this.message = message;
	}
} 
```

2.自定义异常处理器

```java
/**
 * 异常处理器
 */
public class SysExceptionResolver implements HandlerExceptionResolver{

    /**
     * 处理异常业务逻辑
     */
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        // 获取到异常对象
        SysException e = null;
        if(ex instanceof SysException){
            e = (SysException)ex;
        }else{
            e = new SysException("系统正在维护....");
        }
        // 创建ModelAndView对象
        ModelAndView mv = new ModelAndView();
        mv.addObject("errorMsg",e.getMessage());
        mv.setViewName("error");
        return mv;
    }
}
```

3.配置异常处理器

```xml
<!-- 配置异常处理器 -->
<bean id="sysExceptionResolver" class="cn.itcast.exception.SysExceptionResolver"/>
```



## SpringMVC 中的拦截器

### 拦截器的作用

***Spring MVC 的处理器拦截器类似于 Servlet 开发中的过滤器 Filter，用于对处理器进行预处理和后处理。***

用户可以自己定义一些拦截器来实现特定的功能。 

谈到拦截器，还要向大家提一个词——拦截器链(Interceptor Chain)。拦截器链就是将拦截器按一定的顺序联结成一条链。在访问被拦截的方法或字段时，拦截器链中的拦截器就会按其之前定义的顺序被调用。 

说到这里，可能大家脑海中有了一个疑问，这不是我们之前学的过滤器吗?是的***它和过滤器是有几分相似，但是也有区别***，接下来我们就来说说他们的区别: 

- 过滤器是servlet规范中的一部分，任何java web工程都可以使用。  
- 拦截器是 SpringMVC 框架自己的，只有使用了 SpringMVC 框架的工程才能用。  
- 过滤器在 url-pattern 中配置了/*之后，可以对所有要访问的资源拦截。 
- 拦截器它是只会拦截访问的控制器方法，如果访问的是 jsp，html,css,image 或者 js 是不会进行拦截的。  

它也是 AOP 思想的具体应用。 ***我们要想自定义拦截器， 要求必须实现:HandlerInterceptor 接口。*** 							 					 				

### 自定义拦截器

第一步:编写一个普通类实现 HandlerInterceptor 接口

```java
/**
 * 自定义拦截器
 */
public class MyInterceptor1 implements HandlerInterceptor{

    /**
     * 预处理，controller方法执行前
     * return true 放行，执行下一个拦截器，如果没有，执行controller中的方法
     * return false不放行
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyInterceptor1执行了...前1111");
        // request.getRequestDispatcher("/WEB-INF/pages/error.jsp").forward(request,response);
        return true;
    }

    /**
     * 后处理方法，controller方法执行后，success.jsp执行之前
     * @param request
     * @param response
     * @param handler
     * @param modelAndView
     * @throws Exception
     */
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("MyInterceptor1执行了...后1111");
        // request.getRequestDispatcher("/WEB-INF/pages/error.jsp").forward(request,response);
    }

    /**
     * success.jsp页面执行后，该方法会执行
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("MyInterceptor1执行了...最后1111");
    }

}
```



第二步:配置拦截器

```xml
<!--配置拦截器-->
<mvc:interceptors>
  <!--配置拦截器-->
  <mvc:interceptor>
    <!--要拦截的具体的方法-->
    <mvc:mapping path="/user/*"/>
    <!--不要拦截的方法
            <mvc:exclude-mapping path=""/>
            -->
    <!--配置拦截器对象-->
    <bean class="cn.itcast.controller.cn.itcast.interceptor.MyInterceptor1" />
  </mvc:interceptor>

  <!--配置第二个拦截器-->
  <mvc:interceptor>
    <!--要拦截的具体的方法-->
    <mvc:mapping path="/**"/>
    <!--不要拦截的方法
            <mvc:exclude-mapping path=""/>
            -->
    <!--配置拦截器对象-->
    <bean class="cn.itcast.controller.cn.itcast.interceptor.MyInterceptor2" />
  </mvc:interceptor>
</mvc:interceptors>
```

运行结果：

![image-20200921135110800](/assets/imgs/image-20200921135110800.png)

### 拦截器的放行

放行的含义是指，如果有下一个拦截器就执行下一个，如果该拦截器处于拦截器链的最后一个，则执行控制器中的方法。

![image-20200921135227220](/assets/imgs/image-20200921135227220.png)

### 拦截器中方法的说明

#### preHandle

- 预处理，controller方法执行前
- return true 放行，执行下一个拦截器，如果没有，执行controller中的方法
- return false不放行

#### postHandle

- 后处理方法，在业务处理器处理完请求后按拦截器定义逆序调用调用，但是 DispatcherServlet 向客户端返回响应前被调用，

#### afterCompletion

- 只有 preHandle 返回 true 才调用

- 在 DispatcherServlet 完全处理完请求后被调用
- 可以在该方法中进行一些资源清理的操作。

### 拦截器的作用路径

```xml
作用路径可以通过在配置文件中配置。 <!-- 配置拦截器的作用范围 -->
<mvc:interceptors>
    <mvc:interceptor>
			<mvc:mapping path="/**" /><!-- 用于指定对拦截的 url --> 
      <mvc:exclude-mapping path=""/><!-- 用于指定排除的 url--> 
      <bean id="handlerInterceptorDemo1"
		class="com.itheima.web.interceptor.HandlerInterceptorDemo1"></bean> 
  </mvc:interceptor>
</mvc:interceptors>
```

### 多个拦截器执行顺序

![image-20200921150925485](/assets/imgs/image-20200921150925485.png)
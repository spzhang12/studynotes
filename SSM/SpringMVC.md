# 1. Spring MVC

## 1.1 Spring MVC简介

### 1）Spring MVC的特点

Spring MVC的特点：

- 轻量级，简单易学
- 高效，基于请求响应的MVC框架
- 与Spring兼容性好，无缝结合
- 约定优于配置
- 功能强大：RESTful、数据验证、格式化、本地化、主题等
- 简介灵活

### 2）DispatcherServlet

- Spring 的 Web MVC 框架是由请求驱动的，围绕**中央 Servlet（DispatcherServlet）** 设计，该 Servlet 将请求分发给控制器并提供其他功能来促进 Web 应用程序的开发。
-  Spring 的`DispatcherServlet`不仅能做到这一点。它与 Spring IoC 容器完全集成在一起，因此，您可以使用 Spring 拥有的所有其他功能。

下图说明了 Spring Web MVC `DispatcherServlet`的请求处理工作流程。

- DispatcherServlet`是“ Front Controller”设计模式的表达(这是 Spring Web MVC 与许多其他领先的 Web 框架共享的模式)。

![mvc](https://www.docs4dev.com/images/spring-framework/4.3.21.RELEASE/mvc.png)

## 1.2 Spring MVC基本使用

### 1）添加依赖

- junit
- spring-webmvc
- servlet-api
- javax.servlet.jsp-api
- jstl

```xml
<!-- 依赖 -->
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.0.8.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>javax.servlet.jsp-api</artifactId>
        <version>2.3.1</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>jstl</artifactId>
        <version>1.1.2</version>
    </dependency>
</dependencies>
```

### 2）配置web.xml

> 在普通的web项目中，需要把所有的servlet都注册到web.xml中
>
> 在Spring MVC中，只需要把DispatcherServlet注册到web中即可
>
> 同时将springmvc-servlet.xml配置文件传递给DispatcherServlet即可

1. 注册DispatcherServlet

   - 注册org.springframework.web.servlet.DispatcherServlet
   - 关联一个springmvc的配置文件：contextConfigLocation（应该放在资源路径resources在）
   - 设置启动级别

   ```xml
   <!-- 1. 注册DispatcherServlet -->
   <servlet>
       <servlet-name>springmvc</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <!-- 关联一个springmvc的配置文件【servlet-name】-servlet.xml -->
       <init-param>
           <param-name>contextConfigLocation</param-name>
           <param-value>classpath:springmvc-servlet.xml</param-value>
       </init-param>
       <!-- 启动级别 -->
       <load-on-startup>1</load-on-startup>
   </servlet>
   <!-- /  匹配所有的请求（不包括.jsp） -->
   <!-- /*匹配所有的请求(包括.jsp) -->
   <servlet-mapping>
       <servlet-name>springmvc</servlet-name>
       <url-pattern>/</url-pattern>
   </servlet-mapping>
   ```

2. 创建springmvc的配置文件

   - 添加处理器映射器
   - 添加处理器适配器
   - 添加视图解析器

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd">
       
       <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" />
       <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter" />
       
       <!-- 视图解析器 -->
       <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
           <!-- 前缀,这里WEB-INF前边必须加上/，否则会变成全路径 -->
           <property name="prefix" value="/WEB-INF/jsp" />
           <!-- 后缀 -->
           <property name="suffix" value=".jsp" />
       </bean>
   </beans>
   ```

3. 编写业务Controller

   - 实现Controller接口/增加@Controller注解；

   - 需要返回一个ModelAndView，装数据封视图

   ```java
   //这里先采用实现Controller接口的方法
   public class HelloController implements Controller {
       public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
           //ModelAndView  模型和视图
           ModelAndView mv = new ModelAndView();
   
           //封装对象，放在ModelAndView中，
           mv.addObject("msg", "HelloSpringMVC");
           //封装要跳转的视图，放在ModelAndView中
           mv.setViewName("hello");        //   /WEB-INF/jsp/hello.jsp
   
           return mv;
       }
   }
   ```

4. 将Controller交给Spring IOC容器，注册bean 

   - id：以/ 开头，表示要处理来自    http://ip:8080/项目名(即配置tomcat的applicationContext)/id    的访问请求

   ```xml
   <!-- Handler -->
   <bean id="/hello" class="com.spzhang.controller.HelloController" />
   ```

5. 编写视图

## 1.3 Spring MVC执行原理

controller调用service层服务，service层调用dao层，dao层与数据库交互

### 1）执行原理1

![image-20200725154947197](C:\Users\spzha\AppData\Roaming\Typora\typora-user-images\image-20200725154947197.png)



Spring MVC框架主要有**DispatcherServlet**、处理器映射、控制器、视图解析器、视图组成。其中，主要包括DispatcherServlet、HandlerMapping、Controller和ViewResolver四个接口。

**Spring MVC所有请求都经过DispatcherServlet来统一分发**，DispatcherServlet借助**HandlerMapping将请求定位到具体的Controller**，然后由Controller处理用户请求。Controller处理完请求后，将**返回ModelAndView对象给DispatcherServlet前端控制器**。ModelAndView中包含了模型和视图。

DispatcherServlet是整个Web应用的控制器，Controller是单个http请求处理过程中的控制器，ModelAndView是http请求过程中返回的模型和视图。

ViewResolver负责在Web应用中查找View对象，从而将相应结果渲染给客户。

### 2）执行原理2

![img](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7KwPOPWq00pMJiaK86lF6BjIbmPOkY8TxF6qvGAGXxC7dArYcr8uJlWoVC4aF4bfxgCGCD8sHg8mgw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 虚线为程序员需要完成的工作
- 实线Spring MVC自动完成

图为SpringMVC的一个较完整的流程图，实线表示SpringMVC框架提供的技术，不需要开发者实现，虚线表示需要开发者实现。

**简要分析执行流程**

1. DispatcherServlet表示前置控制器，是整个SpringMVC的控制中心。用户发出请求，DispatcherServlet接收请求并拦截请求。

   我们假设请求的url为 : http://localhost:8080/SpringMVC/hello

   **如上url拆分成三部分：**

   http://localhost:8080服务器域名

   SpringMVC部署在服务器上的web站点

   hello表示控制器

   通过分析，如上url表示为：请求位于服务器localhost:8080上的SpringMVC站点的hello控制器。

2. HandlerMapping为处理器映射。DispatcherServlet调用HandlerMapping,HandlerMapping根据请求url查找Handler。

3. HandlerExecution表示具体的Handler,其主要作用是根据url查找控制器，如上url被查找控制器为：hello。

4. HandlerExecution将解析后的信息传递给DispatcherServlet,如解析控制器映射等。

5. HandlerAdapter表示处理器适配器，其按照特定的规则去执行Handler。

6. Handler让具体的Controller执行。

7. Controller将具体的执行信息返回给HandlerAdapter,如ModelAndView。

8. HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet。

9. DispatcherServlet调用视图解析器(ViewResolver)来解析HandlerAdapter传递的逻辑视图名。

10. 视图解析器将解析的逻辑视图名传给DispatcherServlet。

11. DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图。

12. 最终视图呈现给用户

# 2. Spring MVC注解开发

## 2.1 编写web.xml

1. 配置DispatcherServlet
   - 注入springmvc-servlet.xml
   - 配置响应的请求路径

```xml
<servlet>
  <servlet-name>springmvc</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:springmvc-servlet.xml</param-value>
  </init-param>
  <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
  <servlet-name>springmvc</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

## 2.2 编写springmvc-servlet.xml

1. 设置自动扫描包
2. 设置不处理静态资源
3. 开启spring mvc注解支持
4. 配置视图解析器

```xml
<!-- 自动扫描包，让指定包下的注解生效，由IOC容器统一管理 -->
<context:component-scan base-package="com.spzhang.controller" />
<!-- 让Spring MVC不处理静态资源 -->
<mvc:default-servlet-handler />
<!-- 开启注解支持
1. 不用再配置视图映射器和适配器，可以通过@Request等注解直接寻找controller
-->
<mvc:annotation-driven />
<!-- 视图解析器 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/jsp/" />
    <property name="suffix" value=".jsp" />
</bean>
```

## 2.3 编写Controller

1. 类上添加@Controller注解（开启自动扫描包后会被自动配置为bean）：这个类中的所有方法，如果返回值为string，并且有具体页面可以跳转，将会被视图解析器解析
2. 类/方法上添加@RequstMapping等注解，设置类/方法处理的资源请求
3. 编写方法
   1. 封装数据
   2. 直接返回字符串时，会被视图解析器解析（自动添加前缀和后缀）

```java
@Controller
public class HelloController {

    @RequestMapping("/h1")
    public String hello(Model model){
        //封装数据
        model.addAttribute("msg", "Hello Spring MVC Annotation 03");
        return "hello";     //会被视图解析器处理
    }
}
```

# 3. Spring MVC注解

## 3.1 @RequestMapping

- `@RequestMapping`注解 将 URL(例如`/appointments`)映射到整个类或特定的处理程序方法上。
- 通常，类级别的 Comments 将特定的请求路径(或路径模式)Map 到表单控制器上
- 方法级别的 Comments 使特定的 HTTP 方法请求方法(“ GET”，“ POST”等)的主 Map 范围变窄。或 HTTP 请求参数条件。
  - Petcare *示例的以下示例显示了使用该 Comments 的 Spring MVC 应用程序中的控制器：

- 

# 4. RestFul风格

## 4.1 RestFul风格简介

RestFul就是一个资源定位及资源操作的风格，不是标准也不是协议，只是一种风格。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

功能：

- 资源：互联网所有事物都可以被抽象为资源
- 资源操作：使用POST、DELETE、PUT、GET，使用不同方法对资源进行操作
- 分别对应添加、删除、修改、查询

传统方式操作资源：通过不同的参数来实现不同的效果！方法弹衣：post和get

- 查询，GET：http://ip:8080/item/queryItem.action?id=1     
- 新增，POST：http://ip:8080/item/saveItem.action     
- 查询，POST：http://ip:8080/item/updateItem.action     
- 删除，GET或POST：http://ip:8080/item/deleteItem.action?id=1

使用RestFul操作资源：可以通过不同的请求方式（GET、POST、PUT、DELETE）来实现不同的效果！如下，请求地址一样，但是功能可以不同！

-  查询，GET：http://ip:8080/item/1
-  新增，POST：http://ip:8080/item
-  更新，PUT：http://ip:8080/item
-  删除，DELETE：http://ip:8080/item/1

好处：

1. 使路径更加简洁
2. 隐蔽参数，更加安全
3. 获得参数更加方便，框架会自动进行类型转换

## 4.2 具体使用

### 1）方法一

1. 设置@RequstMapping属性：

   - value：具体要处理的资源路径
   - method：此方法响应的请求类型：
     - RequestMethod.GET
     - RequestMethod.POST
     - RequestMethod.PUT
     - RequestMethod.DELETE

2. 如果方法有参数，则给参数添加@PathVariable注解

   ```java
   @RequestMapping(value = "/sum/{a}/{b}", method = RequestMethod.GET)
   public String addTwoNumber( @PathVariable int a,     @PathVariable int b, Model model){
   	...
   }
   ```

### 2）方法二

1. 使用@GetMapping/@PutMapping/@PostMapping/DeleteMapping来取代RequestMapping
2. 如果方法有参数，则给参数添加@PathVariable注解

```java
@GetMapping("/sum/{a}/{b}")
public String addTwoNumber( @PathVariable int a,     @PathVariable int b, Model model){
	...
}
```

http://localhost:8080/sum/1/2的访问会转到addTwoNumber方法

# 5. 重定向和转发

ModelAndView

设置ModelAndView对象，根据其中view属性的名称和视图解析器设置的前缀和后缀跳转到指定的页面

- 页面：{视图解析器前缀}+viewName+{视图解析器后缀}



ServletAPI

通过设置ServletAPI，不需要视图解析器

1. 通过HttServletResponse进行输出

   ```java
   @RequestMapping("/printTest")
   public String test(HttpServletRequest request, HttpServletResponse response) throws IOException {
       response.getWriter().println("Hello, Spring by Servlet API");
       return "test";
   }
   ```

2. 通过HttpServletResponse实现重定向

   - 这种方式视图解析器中的前缀和后缀不会起作用

   ```java
   response.sendRedirect("/index.jsp");
   ```

3. 通过HttpServletResponse实现转发

   - 这种方式视图解析器中的前缀和后缀不会起作用

     ```java
     request.getRequestDispatcher("/WEB-INF/jsp/test.jsp").forward(request, response);
     ```

## 5.2 Spring MVC

### 1）配置视图解析器时

- 转发

  - 配置了视图解析器前缀后缀时：只需要return视图文件名即可
  - 没有配置视图解析器前缀后缀时：需要return全路径，可以访问/WEB-INF下的视图

  ```java
  @RequestMapping("/test")
  public String test1(){
      //默认为转发方式，相当于 return "forward:/WEB-INF/jsp/test.jsp";浏览器地址栏不会发生变化
      return "/WEB-INF/jsp/test.jsp";
  }
  ```

- 重定向

  - 转发不能访问/WEB-INF下的资源
  - 转发可以忽视视图解析器的前缀和后缀，跳转到http://ip:8080/工程名/redirect后的资源路径，然后寻找重新发起请求寻找新的controller方法来处理请求
  - 如何指定请求得方法呢？GET？PUT？POST？

  ```java
  @RequestMapping("/test")
  public String test1(){
      //转发，浏览器中地址栏会跳转到转发的地址，http://localhost:8080/工程名/sum/1/2
      return "redirect:/sum/1/2";
  }
  ```

  # 6. 接收请求参数及数据回显

  ## 6.1 处理提交的数据

  ### 1）提交数据为普通类型

  - 提交的域名称和处理方法的参数名一致

    - 传递的域名称与方法参数名称一致时，可以直接使用
    - 这种方式没有传递参数时，仍可以执行，只是对应参数为null

    ```java
    @GetMapping("/t1")
    public String test1(String name, Model model){
        System.out.println(name);
        model.addAttribute("msg", "test1");
        return "test";
    }
    ```

  - 提交的域名称和处理方法的参数名不一致

    - @RequestParam("域名称")

    - 传递的域名称与方法参数名称不一致时，需要借助注解@RequestParam

    - 这种方式不能不传递参数，如果没有传递参数，则会出现400错误

    - ```java
      @GetMapping("/t2")
      public String test2(@RequestParam("name") String username, Model model){
          System.out.println(username);
          model.addAttribute("msg", "test2");
          return "test";
      }
      ```

  ### 2）提交一个对象

  - 传递一个对象时，如果各提交的参数名与对象属性名相同，则自动填入相应字段，没有找到对应的参数的字段为null
  - 这种方式没有传递参数时，仍可以执行，只是对象各字段被设置为null

  ```java
  @GetMapping("/t3")
  public String test3(User user, Model model){
      System.out.println(user);
      model.addAttribute("msg", "test3");
      return "test";
  }
  ```

  ## 6.2 数据显示到前端

  ### 1）ModelAndView

  - 通过setViewName实现视图跳转
  - 返回ModelAndView对象

  ```java
  //使用ModelAndView返回数据
  @GetMapping("/t5")
  public ModelAndView test4(HttpServletRequest request, HttpServletResponse response){
      //返回一个模型视图对象
      ModelAndView mv = new ModelAndView();
      mv.addObject("msg", "Controller test5");
      mv.setViewName("test");
      return mv;
  }
  ```

  ### 2）ModelMap

  - 继承了LinkedHashMap，拥有LinkedHashMap的全部功能

    ```java
    //使用ModelMap返回数据
    @GetMapping("/t4")
    public String test4(String name, ModelMap model){
        //封装要显示到视图中的数据
        //相当于req.setAttribute("msg",name)
        model.addAttribute("msg", name);
        System.out.println(name);
        return "test";
    }
    ```

  ### 3）Model

  - ModelMap简化版，适用于大部分场景

  ```java
  //使用ModelMap返回数据
  @GetMapping("/t4")
  public String test4(String name, ModelMap model){
      //封装要显示到视图中的数据
      //相当于req.setAttribute("msg",name)
      model.addAttribute("msg", name);
      System.out.println(name);
      return "test";
  }
  ```

  

  ### 4）区别

  - Model：只有寥寥几个方法只适合用于存储数据，简化了新手对于Model对象的操作和理解
  - ModelMap：继承了LinkedMap，除了实现了自身的一些方法，同样的继承LinkedMap的方法和特性
  - ModelAndView：可以在存储数据的同时，可以进行设置返回的逻辑视图，进行控制展示层的跳转

  

# 7. 乱码问题

## 7.1 过滤器解决乱码问题

通过配置springmvc提供的过滤器CharacterEncodingFilter可以解决中文乱码问题。

```xml
<filter>
  <filter-name>encoding</filter-name>
  <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  <init-param>
    <param-name>encoding</param-name>
    <param-value>utf-8</param-value>
  </init-param>
</filter>
<!-- /会拦截.jsp文件，需要设置为/* -->
<filter-mapping>
  <filter-name>encoding</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

# 8. Spring MVC使用JSON返回数据

## 8.1 什么是JSON

JSON是一种轻量级的数据交换格式，目前使用广泛

- 采用完全独立于编程语言的文本格式来存储和表示数据
- 易于阅读，易于及其解析和生成，并有效地提升网络传输效率

JSON键值对是用来保存JavaScript对象的一种方式，前端可以获取JSON，并将其转换为js对象

```json
{"id":10,"name":"?"}
```

前后端分离时代：

后端部署后端，提供接口

前端独立部署，负责渲染后端数据

## 8.2 Jackson

- 添加依赖

  ```xml
  <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.11.2</version>
  </dependency>
  ```

- 将对象转换为JSON字符串

  - 使用ObjectMapper对象的writeValueAsString(Object obj) 方法完成转换

  ```java
  JsonMapper mapper = new JsonMapper();
  String str = mapper.writeValueAsString(user);
  ```

- 将Date对象转换为JSON字符串

  - 使用SimpleDateFormat

  ```java
  JsonMapper mapper = new JsontMapper();
  Date date = new Date();
  SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
  mapper.setDateFormat(sdf);
  String str = mapper.writeValueAsString(date);
  ```

  - 使用mapper的configure方法

    ```java
    JsonMapper mapper = new JsonMapper();
    mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
    String str = mapper.writeValueAsString(date);
    return   str;
    ```

- 可以将Jackson封装为一个utils工具类

  ```java
  public class JsonUtils {
  
      public static String getJson(Object object) throws JsonProcessingException {
          return getJson(object, null);
      }
      public static String getJson(Object object, String format) throws JsonProcessingException {
          JsonMapper mapper = new JsonMapper();
          SimpleDateFormat sdf = new SimpleDateFormat(format);
          mapper.setDateFormat(sdf);
          try {
              return mapper.writeValueAsString(object);
          } catch (JsonProcessingException e) {
              e.printStackTrace();
          }
          return null;
      }
  }
  ```

- @ResponseBody

  - 该注解注解在方法上时，表示此方法返回的string对象不会作为视图被解析，而是会绑定到response中返回



## 8.3 Fastjson

### 1）添加依赖



### 2）使用

#### I. 将Java对象转换为JSON字符串

```java
String str = JSON.toJSONString(user);
```

#### II. 将JSON字符串转换为Java对象

```
User user = JSON.parseObject(str, User.class);
```

#### III. 将Java对象转换为JSON对象

```java
JSON json = (JSON) JSON.toJSON(list);
```

#### III. 将JSON对象转换为Java对象

```java
User user = JSON.toJavaObject(json, User.class);
```

# 9. Ajax初体验

ajax的全称是***\*A\*****synchronous****\*J\*****avascript****+\**\*X\*****ML。*
异步传输+js+xml。



所谓异步，在这里简单地解释就是：向服务器发送请求的时候，我们不必等待结果，而是可以同时做其他的事情，等到有了结果我们可以再来处理这个事。（当然，在其他语境下这个解释可能就不对了）
这个很重要，如果不是这样的话，我们点完按钮，页面就会死在那里，其他的数据请求不会往下走了。这样比等待刷新似乎更加讨厌。
（虽然提供异步通讯功能的组件默认情况下都是异步的，但它们也提供了同步选项，如果你好奇把那个选项改为false的话，你的页面就会死在那里）
xml只是一种数据格式，在这件事里并不重要，我们在更新一行字的时候理论上说不需要这个格式，但如果我们更新很多内容，那么格式化的数据可以使我们有条理地去实现更新。

现在大部分人其实是用JSON这种格式来代替XML的，因为前者更加简洁，据说目前的解析速度也更快。多快好省，能省则省啊。


**总结：只要是JS调用异步通讯组件并使用格式化的数据来更新web页面上的内容或操作过程，那么我们用的方法就可算是AJAX。**

# 9. Spring MVC入门

> MVC的设计思想以及Spring MVC的工作原理

MVC思想将一个应用分成3个基本部分，即Model（模型）、View（视图）和Controller（控制器），让这3个部分以最低的耦合进行协同工作，从而提高应用的可扩展性及可维护性。Spring MVC是一款优秀的基于MVC思想的应用框架，它是Spring提供的一个实现了Web MVC设计模式的轻量级Web框架。

## 9.1　MVC模式与Spring MVC工作原理

### 1）MVC模式

#### 1. MVC的概念

MVC分别代表Web应用程序中三种职责：

- M（Model，模型）：用于**存储用户数据**以及**处理用户请求**的业务逻辑
- V（View，视图）：**向控制器提交数据**，并**显示模型中的数据**
- C（Controller，控制器）：根据视图提出的请求**判断将请求和数据交给哪个模型处理**，**将处理后的有关结果交给哪个视图更新显示**。

#### 2. 基于Servlet的MVC模式

- **M（Model，模型）**：一个或多个**JavaBean对象**，用于存储数据和处理业务逻辑
- V（View，视图）：一个或多个JSP页面，向控制器提交数据和为模型提供数据显示
- **C（Controller，控制器）**：一个或多个**Servlet对象**，根据视图提交的请求进行控制，即将请求转发给处理业务逻辑的JavaBean，并将处理结果存放到实体模型JavaBean中，输出给视图显示

### 2）Spring MVC的工作原理

![image-20200725154947197](1. Spring MVC入门.assets/image-20200725154947197.png)

Spring MVC框架主要有**DispatcherServlet**、处理器映射、控制器、视图解析器、视图组成。其中，主要包括DispatcherServlet、HandlerMapping、Controller和ViewResolver四个接口。

**Spring MVC所有请求都经过DispatcherServlet来统一分发**，DispatcherServlet借助**HandlerMapping将请求定位到具体的Controller**，然后由Controller处理用户请求。Controller处理完请求后，将**返回ModelAndView对象给DispatcherServlet前端控制器**。ModelAndView中包含了模型和视图。

DispatcherServlet是整个Web应用的控制器，Controller是单个http请求处理过程中的控制器，ModelAndView是http请求过程中返回的模型和视图。

ViewResolver负责在Web应用中查找View对象，从而将相应结果渲染给客户。

## 9.2 第一个Spring MVC应用

项目结构：![image-20200725110949419](1. Spring MVC入门.assets/image-20200725110949419.png)

### 1）创建web应用并添加需要的依赖

创建Spring MVC应用时，除了Spring的核心依赖，还需要添加spring-web、spring-webmvc以及servlet-api这三个依赖。

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.0.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.0.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
</dependency>
```

### 2）在web.xml文件中部署DispatcherServlet

部署之后Spring会自动创建一个名为springmvc的servlet对象，并且其在初始化时将在应用程序的WEB-INF目录下查找一个名为springmvc-servlet.xml的配置文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
		 http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <!-- 部署DispatcherServlet -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 加载springmvc的配置文件 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/springmvc-servlet.xml</param-value>
        </init-param>
        <!--表示容器在启动时立即加载servlet -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!-- 处理所有的URL -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

### 3）创建Web应用首页

Web应用首页index.jsp应该位于webapp目录下，其他jsp页面应根据其对应类别分别存放于webapp目录下的文件夹中。

![image-20200725152605756](1. Spring MVC入门.assets/image-20200725152605756.png)

```jsp
<%@ page language="java" contentType="text/html;   charset=UTF-8"
    pageEncoding="UTF-8"%>
<html>
<body>
<h2>Hello World!</h2>
<a href="${pageContext.request.contextPath }/register">注册</a>
<a href="${pageContext.request.contextPath }/login">登录</a>
</body>
</html>

```

### 4）创建Controller类

此处使用实现Controller接口的方式来创建控制器类，每个控制器需要在Spring MVC配置文件中进行配置。其文件结构如下：

![image-20200725152751267](1. Spring MVC入门.assets/image-20200725152751267.png)

LoginController类的具体实现如下：

```java
public class LoginController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        //其返回的视图路径为从webapp下一个目录开始
        return new ModelAndView("/jsp/login.jsp");
    }
}
```

### 5）创建Spring MVC配置文件并配置Controller映射信息

使用实现Controller接口方式创建的控制器类需要在Spring MVC配置文件中进行映射，该配置文件可以存放在webapp/WEB-INF中。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- LoginController控制器类，将来自/login的请求转交给LoginController处理" -->
    <bean name="/login" class="controller.LoginController"/>
    <!-- ResisterController控制器类" -->
    <bean name="/register" class="controller.RegisterController"/>
</beans>
```

同时，可以在配置文件中定义一个视图解析器，可以配置控制器返回视图路径的前缀和后缀。

```xml
<!-- 视图解析器，提供路径的前缀和后缀 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
    <!-- 前缀 -->
    <property name="prefix" value="/jsp/" />
    <!-- 后缀 -->
    <property name="suffix" value=".jsp" />
</bean>
```

# 10. Spring MVC的Controller

## 10.1 基于注解的控制器

> 传统创建控制器的方式不灵活，且复杂，通过注解创建控制器比较灵活并且简单

Spring MVC中最重要的两个注解类型：@Controller和@RequestMapping

### 1）Controller注解类型

该注解声明某类的实例是一个控制器，需要在Spring MVC中使用扫描机制可以找到应用中所有基于注解的控制器类。即，需要在springmvc.xml配置文件中添加开启扫描的声明。

### 2）编写请求处理方法

控制类中每个请求处理方法可以有多个不同类型的参数，以及一个多种类型的返回结果。

请求处理方法中常出现的参数类型有：

- Servlet API
- 输入输出流
- Spring框架相关的类型，如Model

请求处理方法常见的返回类型：

- 代表逻辑视图名称的String类型
- ModelAndView
- Mode
- View

## 10.2 Controller接受请求参数的常见方式

> 如何将来自客户端的请求中的参数传递给Controller

### 1）通过实体Bean接受请求参数

### 2）通过处理方法的形参接受请求参数

### 3）通过HttpServletRequest接收请求参数

### 4）通过@PathVariable接收URL中的请求参数

### 5）通过@RequestParam接受请求参数

### 6）通过@ModelAttribute接受请求参数

## 10.3 重定向与转发

## 10.4 应用@Autowired进行依赖注入

## 10.5 @ModelAttribute

@ModelAttribute注解可以修饰请求处理方法的参数，也可以注解一个非请求处理方法。

### 1）注解请求处理方法的参数

使用@ModelAttribute注解对请求处理方法的参数进行注解时，其具有两个功能：

1. 将请求参数的输入封装到对应的参数对象中
2. 创建对象实例，以其定义的键值存储在Model对象中

```java
//如果没有指定"user"，则默认创建UserForm实例时以"userFomr"为键值存储在Model对象中
@ModelAttribute("user") UserForm user
```

### 2）注解一个非请求处理方法

被@ModelAttribute注解的方法将在每次调用该控制器的请求处理方法前被调用。这种特性可以用来控制登陆权限。

1. 绑定请求参数到实体对象（表单的命令对象）
2. 注解一个非请求处理方法

# 11. 类型转换和格式化

> 将请求参数转换为相应的数据类型

# 12. Spring MVC的拦截器

> 主要用于访问权限控制

Spring MVC的处理器拦截器类似于Servlet开发中的过滤器Filter，用于对处理器进行预处理和后处理。

过滤器和拦截器的区别：拦截器是AOP思想的具体应用

过滤器：

- 在servlet规范中的一部分，任何java web工程都可以使用
- 在url-pattern中配置了/*之后，可以对所有要访问的资源进行拦截

拦截器

- 拦截器是Spring MVC框架自己的，只有使用了Spring MVC框架的工程才能使用
- 拦截器只会拦截访问的控制器方法，如果访问的是jsp/html/css/image/js  是不会进行拦截的

 自定义拦截器

要自定义拦截器，必须实现HandlerInterceptor接口

1. 实现HandlerInterceptor接口，重写其中三个方法

   - preHandler()：先于请求前执行
   - postHeanlder()：请求执行完，渲染结束前执行
   - afterCompletion()：请求执行完，渲染结束后执行

2. 在springmvc-servlet中进行拦截器配置

   - 直接在\<mvc:interceptors>标签下添加\<bean>元素，表示该bean对应的拦截器会拦截所有的请求
   - \<mvc:intercepter>该子元素配置一个拦截器拦截特定的请求（不拦截特定的请求）
     - \<mvc:mapping path="/admin/**">：对应的bean拦截/admin/下的全部请求
     - \<mvc:unmapping path="/admin/**">：对应的bean不拦截/admin下的全部请求
   - 多个拦截器会按照其配置的顺序来执行对应的方法（post、after执行顺序与其配置顺序相反）

   ```xml
   <!-- 配置拦截器 -->
   <mvc:interceptors>
       <!-- 拦截所有请求 -->
       <bean class="com.spzhang.intercepter.LoginIntercepter" />
       <!-- **包括这个请求下的下边的所有请求 -->
       <mvc:interceptor>
           <mvc:mapping path="/**"/>
           <bean class="com.spzhang.intercepter.LoginIntercepter" />
       </mvc:interceptor>
   </mvc:interceptors>
   ```

   
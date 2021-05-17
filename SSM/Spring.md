# 1. Spring入门

## 1.1 Spring的体系结构

Spring框架已经集成了20多个模块，这些模块分布在核心容器、数据访问/集成层、Web层、AOP模块、植入模块、消息传递和测试模块中。

> https://www.cnblogs.com/xiaobaizhiqian/p/7616453.html

![image-20200827091839948](C:\Users\spzha\AppData\Roaming\Typora\typora-user-images\image-20200827091839948.png)

### 1）核心容器

Spring的核心容器是其他模块建立的基础，核心容器主要包括以下几个部分：

- **spring-core**：提供框架基本组成，包括**控制反转（IoC）和依赖注入（DI）**功能；
- **spring-beans**：提供了**BeansFactory**，是**工厂模式的一个经典实现**，Spring将管理对象称为Bean；
- **spring-context**模块：建立在spring-beans和spring-core基础之上，**提供了一个框架式的对象访问方式**，**ApplicationContext接口是该模块的焦点**；
- spring-context-support：支持整合第三方库到spring应用程序上下文中；
- spring-expression：提供了强大的表达式语言去支持运行时查询和操作对象图；

## 2）数据访问/继承

数据访问/集成层由JDBC、ORM、OXM、JSM和事务模块组成：

- spring-jdbc：
- spring-tx：
- spring-orm：
- spring-oxm

### 3）Web层

web层主要包括以下几个模块：

- spring-web：
- spring-webmvc：
- spring-websocket：
- porlet：

### 4）AOP和植入

aop模块主要包括：

- spring-aop：
- spring-aspects：

植入模块主要包括：

- spring-instrument：

### 5）消息

spring 4.0之后新增了消息模块，该模块提供了对消息传递体系结构和协议的支持。

### 6）测试

测试模块主要包括spring-test，支持使用JUnit或TestNG对Spring组件进行单元测试和集成测试。 

## 1.2 Spring的优点

- Spring是一个开源的免费的框架（容器）
- Spring是一个轻量级的、**非入侵式**（在其他项目中集成Spring后不会导致原项目不可用）的框架
- 控制反转（IoC）、面向切面编程（AOP）
- 支持事务的处理，对框架整合的支持

总结：Spring就是一个**轻量级**的**控制反转**和**面向切面编程**的框架！

Spring发展太久，违背了原来的理念！配置十分繁琐，

## 1.3 拓展

- Spring Boot（构建一切）
  - 快速开发的脚手架
  - 基于Spring Boot可以快速的开发单个微服务
  - **约定大于配置**

- Spring Cloud（协调一切）
  - Spring Cloud是基于SpringBoot实现的
  - 微服务的整合

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128215449383.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FudGljbHFs,size_16,color_FFFFFF,t_70)

大多公司都在使用Spring Boot进行快速开发，学习Spring Boot的前提，需要完全掌握Spring及Spring MVC。

# 2. Spring IoC

## 2.1 IoC理论推导

### 1）由业务层负责创建对象

在之前的业务中，用户的需求可能会影响我们原来的代码，我们需要根据用户的需求去修改源代码，如果程序代码量十分大，修改一次的成本代价十分昂贵。

```java
private UserDao userDao = new UserDaoImpl();
//如果由业务层负责创建对象，则修改对象实现时，需要手动修改源代码
//pivate UserDao userDao = new UserDaoIMySqlmpl();
```

### 2）使用set方法，由用户创建对象

```java
private UserDao userDao;

public void setUserDao(UserDao userDao){
	this.userDao = userDao;
}
//对于用户层，可以通过setUserDao来传递不同UserDao接口实现对象
```

- 之前**业务层通过new主动创建对象**，控制权在业务层
  - **耦合性较大**，不利于后期代码的升级与维护
- 使用set注入后，程序不再具有主动性，变为被动接收对象（**IoC的原型！**）
  - **业务层不再负责管理对象的创建**，降低系统的耦合性，**更加关注于业务的实现**

## 2.2 Spring IoC

当一个对象要调用另一个对象的方法时，传统的方法是使用new的方式创建一个对象，但这样会增加代码之间的耦合度，不利于后期代码的升级与维护。**在Spring框架中，对象实例（在Spring中被称为Bean）不再由调用者来创建，而是由Spring容器创建**，Spring容器负责控制程序之间的关系。这种控制权由调用者转移到Spring容器的方式叫做控制反转。

Spring中实现控制反转的是IoC容器，实现方式是依赖注入。**Spring IoC容器（创建Bean、如何获取Bean）的设计主要是基于BeanFactory和ApplicationContext这两个接口**。

- 控制：谁来控制对象的创建，传统应用程序的对象是由程序本身控制创建的，使用Spring后，对象是由Spring来创建的
- 反转：程序本身不创建对象，而变成被动的接收对象
- **依赖注入：利用set方法进行注入创建对象变成被动的接收**。

## 2.3 IoC创建Bean的方式

### 1）无参构造方法创建对象

- **要求类中不存在有参构造方法**
- 默认的创建bean的方法

### 2）有参构造方法

#### I. 基于下标

```xml
<bean id="user" class="com.spzhang.pojo.User">
    <constructor-arg index="0" value="spzhang" />
</bean>
```

#### II. 基于类型（不推荐）

```xml
<!-- 基于有参构造方法, 通过类型 -->
<bean id="user" class="com.spzhang.pojo.User">
    <constructor-arg type="java.lang.String" value="spzhang" />
</bean>
```

#### III. 基于参数名

```xml
<bean id="user" class="com.spzhang.pojo.User">
    <constructor-arg name="name" value="spzhang" />
</bean>
```

## 2.4 Spring配置文件（applicationContext.xml）的配置

### 1）别名

> **别名有什么用？**

- 在一个applicationContext.xml配置文件中，可以**将多个bean的ID设置为相同的一个别名**，在别处引用这个别名时，**默认选择配置文件中最后声明的那个**。

```xml
<!-- 引用aliasUser时，默认引用的是userDaoMySql -->
<alias name="userDao" alias="aliasUser"/>
<alias name="userDaoMySql" alias="aliasUser"/>
```

### 2）import

一般用于团队开发时使用，可以将多个配置文件导入合并为一个。

```xml
<import resource="beans.xml" />
<import resource="beans2.xml" />
```

# 3. 依赖注入

## 3.1 手动注入

### 1）构造器注入

- 有参构造器
  - 下标
  - 参数名
  - 类型

### 2）Set方式注入

- 依赖注入：Set注入
  - 依赖：bean对象的创建依赖于容器
  - 注入：bean对象中的所有属性，由容器来注入

**要求**：

- **要注入的属性必须有set方法**

【环境搭建】

- 创建要进行注入的类Student与Address

  ```java
  //记得添加getter和setter函数    
  private String name;
  private Address address;
  private String[] books;
  private List<String> hobbies;
  private Map<String, String> card;
  private Set<String> games;
  private String wife;
  private Properties info;
  ```

  ```java
  public class Address {
      private String address;
  
      public String getAddress() {
          return address;
      }
  
      public void setAddress(String address) {
          this.address = address;
      }
  }
  ```

【注入属性类型】

- 普通值注入

  ```xml
  <!-- 普通值注入 -->
  <property name="name" value="spzhang"/>
  ```

- bean对象注入

  ```xml
  <!-- Bean注入 -->
  <property name="address" ref="address" />
  ```

- 数组注入

  ```xml
  <!-- 数组注入 -->
  <property name="books">
      <array>
          <value>Effective Java</value>
          <value>并发编程</value>
          <value>Java核心技术卷</value>
      </array>
  </property>
  ```

- List注入

  ```xml
  <!-- List注入 -->
  <property name="hobbies">
      <list>
          <value>打篮球</value>
          <value>写代码</value>
          <value>不唱歌</value>
      </list>
  </property>
  ```

- Map注入

  ```xml
  <!-- Map注入 -->
  <property name="card">
      <map>
          <entry key="开户行" value="工商银行"/>
          <entry key="账号" value="62202***" />
      </map>
  </property>
  ```

- Set注入

  ```xml
  <!-- Set注入 -->
  <property name="games">
      <set>
          <value>LOL</value>
          <value>2K</value>
      </set>
  </property>
  ```

- **null注入**

  ```xml
  <!-- null注入 --><property name="wife">    <null/></property>
  ```

- **Properties注入**

  ```xml
  <!-- Properties注入 --><property name="info">    <props>        <prop key="driver">10142510267</prop>        <prop key="url">2014年</prop>        <prop key="username">男</prop>        <prop key="password">1025</prop>    </props></property>
  ```

### 3）拓展方式注入

#### I. p 命名空间注入

依赖于第三方的约束，可以世界在bean标签中使用p来注入简单的属性。

-    先添加以下依赖： xmlns:p="http://www.springframework.org/schema/p"
-    在bean标签中通过p:属性名=""的方式对bean对象的简单属性进行注入 

```xml
<bean id="" class="" p:name="spzhang" p:age="18" />
```

### II. c 命名空间注入

依赖于第三方的约束，并且要被注入的bean必须有有参构造器。

- 先添加以下依赖：xmlns:c="http://www.springframework.org/schema/c"
- 在bean标签中通过c:构造方法参数名=""的方式对bean对象的简单属性进行注入 

```xml
<bean id="user" class="com.spzhang.pojo.User" c:name="spzhang" c:age="18" />
```

## 3.2 Bean的自动装配（注入）

Spring中有三种装配的方式：

- 在xml中显示配置
- 在java中显示配置
- **隐式的自动装配**

自动装配是Spring满足bean依赖的一种方式，Spring会在上下文中自动寻找，并自动给bean装配属性！

### 1）byName自动装配

- **要注入的属性必须有set方法**

- 自动在容器上下文中查找，和自己对象**set方法后面的值**相同的bean

```xml
<bean id="" class="" autowire="byName">
```

### 2）byType自动装配

- **要注入的属性必须有set方法**

- 自动在容器上下文中查找，和自己对象属性类型相同的bean

  ```xml
  <bean id="" class="" autowire="byType">
  ```

- byName的时候，需要保证**所有bean的id唯一（否则会报错）**，并且这个bean需要和自动注入的属性的set方法的值一致

- byType的时候，需要保证**所有bean的class唯一（否则会报错）**，并且这个bean需要和自己对象属性类型相同的bean

## 3.3 基于注解的自动注入

Spring提供了通过注解的方式自动装配Bean对象属性的方式。该方法只需要三项项配置：

1. 导入context的约束

   ```xml
   <beans xmlns:context="http://www.springframework.org/schema/context"       xsi:schemaLocation="http://www.springframework.org/schema/context        http://www.springframework.org/schema/context/spring-context.xsd">
   ```

2. 在配置文件配置注解的支持

```xml
<context:annotation-config />
```

3. 可以使用@Autowried或者@Resource注解来实现自动的依赖注入。



### **@Autowired**

- 直接在属性上使用即可，也可以在set方式上使用！

- 使用@Autowired可以省去Set方法，前提是这个自动装配的属性在IoC容器中存在且符合名字（byName）

- @Autowired(required=false)（默认为true）：如果显示定义了Autowired的require属性为false，则该属性可以为null，否则不允许为null

- 可以**与@Qualifier注解配合使用**，来实现按照名称来装配注入

- **出现两个满足条件的bean对象会报错**

  ```java
  @Autowired@Qualifier("dog")		//自动注入类型为Dog且id为dog的beanDog dog;
  ```

**@Nullable**：被该注解注解的字段标识该字段可以为null。

注意：**Java变量的初始化顺序**为：**静态变量或静态语句块–>实例变量或初始化语句块–>构造方法–>@Autowired**；所以如果在@Autowried注入之前，就使用了@Autowried注解的变量，那么就可能报错。这种情况下，可以使用@Autowired注解构造方法，在构造方法中先对@Autowried变量初始化（**建议写法：在构造函数中完成注入依赖**）。

```java
private User user;private String school;@Autowiredpublic UserAccountServiceImpl(User user){    this.user = user;    this.school = user.getSchool();}
```

@Autowried可以实现**list、map、set等类型的依赖注入**，因为@Autowried会自动将list、set、map对应的类型的组件组装到一起注入到list、map、set中。其中，对map对象进行自动注入时，对象名会作为key。

```java
/*** Base接口两个实现类BaseImplOne和BaseImplTwo,这两个实现类被扫描分别创建一个bean组件**/@Componentpublic class AutowriedTest {    @Autowired    List<Base> list;    //只允许String作为key，会将对应类型的对象名作为key传进去    @Autowired    Map<String, Base> map;	//必须添加@Qualifier注解指明具体注入那一个组件    @Autowired    @Qualifier("baseImplOne")    Base base;    public void test(){        if(list != null){            for(Base base:list){                System.out.println(base.getClass().getName());            }        }else {            System.out.println("list为空，表示没有注入");        }        if(map != null){            for(Map.Entry<String, Base> m:map.entrySet()){                System.out.println(m.getKey()+"    "+m.getValue().getClass().getName());            }        }else{            System.out.println("map为空，表示没有注入");        }        if(base != null){            System.out.println(base.getClass().getName());        }else{            System.out.println("testDao注入失败");        }    }    public static void main(String[] args){        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("appContext.xml");        AutowriedTest test = (AutowriedTest) applicationContext.getBean("autowriedTest");        test.test();    }}
```



### **@Resource注解**

- **该注解默认按照名称来装配注入**，只有当找不到与名称匹配的Bean时，才会按照类型来装配注入
- @Resource(name="dog")：按照名称dog来装配注入



### **小结**

- 都是用来自动装配的，都可以放到属性字段上
- **@Autowired通过byType的方式实现，而且必须要求对象存在**（除非设置required=false）
- **@Resource默认通过byName的方式实现，如果找不到名字，则通过byType方式实现**



# 4. Spring Bean

> 在Spring应用中，Spring IoC容器可以创建、装配和配置应用组件对象，这里的组件对象称为Bean。本章主要介绍如何将Bean装配到Spring IoC容器中。

## 4.1 Bean的XML配置

Spring框架支持XML和Properties两种格式的配置文件，常采用XML文件的方式进行配置。

### 1）XML配置方式

> applicationContext.xml文件专门用来配置bean组件，该文件一般存放到resources资源路径下，因为ClassPathXmlApplicationContext默认从在该目录中查找bean组件配置文件。

bean配置文件的根元素是<beans\>，<beans\>元素中包含了多个<bean\>子元素，每一个<bean\>元素定义一个Bean，并描述Bean如何被装配到Spring容器中。

#### I. \<bean>的属性：

- **id**：唯一标识，相当于变量名

- **class**：具体的类

- **name**：别名，比\<alias>更高级，可以取多个别名（别名可以通过逗号、分号、空格分隔）：

  ```xml
  <bean id="user" class="com.spzhang.UserDaoImpl" name="u1, u2, u3" />
  ```

- **scope**：指定bean组件的作用域

- **autowire**：指定自动装配的类型

### II. Bean的作用域

在xml中配置bean组件时，可以指定该bean组件的作用域。作用域一共包括6种，默认为singleton，即容器仅生成一个Bean实例。此外，还有prototype作用域，作用域被设置为prototype的bean，Spring IoC容器将为每次请求创建一个新的实例。

- singleton（默认机制，并发时出现问题）

  ```xml
  <bean id="" class="" scope="singleton">
  ```

- prototype（每次从容器中get时都会产生一个新对象，浪费资源）

  ```xml
  <bean id="" class="" scope="prototype">
  ```

- 其余的request、session、application，这些只能在web开发中使用到。

## 4.2 Bean的基于注解的自动装配

Spring提供了通过注解的方式自动扫描来创建Bean组件的方法。该方法只需要三项项配置：

1. 通过对类添加以下几个注解：@Component/@Respository/@Service/@Controller

2. 导入context的约束

   ```xml
   <beans xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd">
   ```

3. 在配置文件中配置扫描路径


```xml
<context:component-scan base-package="要扫描的路径">
```

## 3.2 Bean的实例化

在Spring框架中，基于XML配置Bean组件时，Spring容器有三种实例化Bean的方式：1）构造方法实例化；2）静态工厂实例化；3）实例工厂实例化。

### 1）构造方法实例化

Spring容器可以通过调用组件类的无参构造函数来创建一个实例化组件，传统在xml文件中添加bean元素即为这种方式。

### 2）静态工厂实例化

这种方式首先需要在组件类中提供了一个静态方法来创建Bean的实例，然后在xml文件中添加<bean\>元素时要通过factory-method属性来指定工厂类中的静态方法。

```xml
<bean id="" class="" factory-method="静态方法名" />
```

### 3）实例工厂实例化

这种方式首先需要在组件类中提供了一个实例方法来创建Bean的实例，然后在xml文件中添加了构造方法实例化的<bean\>组件元素后，还需要添加一个<bean\>元素，设置其factory-method及factory-bean属性。

```xml
<bean id="myFactory" class="" />
<bean id="instanceFactory" factory-bean="myFactory" factory-method="实例方法" />
```

## 3.4 Bean的生命周期

### 获取Bean组件的方式

#### 1） BeanFactory

BeanFactory提供了完整的IoC服务支持，是一个管理Bean的工厂，负责初始化各种Bean。该接口有多个实现类，最常用的是XmlBeanFactory，该类会根据XML的配置定义来装配Bean。

在创建BeanFactory实例时需要提供XML文件在磁盘上的绝对路径。

```java
BeanFactory beanFactory = new XmlBeanFactory("绝对路径");
TestDao  testDao = (TestDao) beanFactory.getBean("test");
```

#### 2）ApplicationContext

> applicationContext.xml文件专门用来配置bean组件

**ApplicationContext是BeanFactory的子接口**，该接口额外添加了对国际化、资源访问、事件传播等内容的支持。创建ApplicationContext接口实例来获取Bean对象的有以下三种方式：

1. 通过ClassPathXmlApplicationContext：需要提供xml文件名，将从resources目录中查找

2. 通过FileSystemXmlApplicationContext：需要提供xml配置文件在磁盘上的绝对路径。

3. 通过Ｗeb服务器实例化ApplicationContext容器（稍后再看）

```java
ApplicationContext appCon = new ClassPathXmlApplicationContext("appContext.xml");	TestDIService testDIService = (TestDIService)appCon.getBean("testDIService");
```

# 5. 使用注解开发

在Spring 4后，要使用注解开发，必须保证导入了aop包。

使用注解，首先要导入context依赖，并且添加注解支持。

## 5.1 自动装配bean

**@Component**注解可以用来实现bean的自动扫描和装配，首先需要导入context依赖，并且添加自动扫描支持：

```xml
<context:component-scan base-package="要扫描的包路径">
```

Component的衍生注解：

- dao：**@Repository**
- service：**@Service**
- controller：**@Controller**

以上四个注解功能都是一样的，都是代表将某个类注册到Spring中，装配Bean。

## 5.2 自动注入属性

使用@Value注解来注入属性，该注解可以直接注解属性，也可以注解set方法

```java
@Component
public class User {
    //  @Value("spzhang")
    private String name;

    public String getName() {
        return name;
    }

    //相当于<property name="name" value="spzhang" />
    @Value("spzhang")
    public void setName(String name) {
        this.name = name;
    }
}

```

## 5.3 作用域

@Scope注解可以设置bean对象的作用域：

```java
@Component
@Scope("prototype")
```

## 5.4 小结

xml与注解的比较：

- xml功能更加强大，适用于任何场合，维护简单方便
- 注解不是自己类使用不了，维护相对复杂

xml与注解最佳实践：

- xml负责管理bean
- 注解只负责完成属性的注入

在使用时，勿忘记开启注解支持。



# 6. 使用Java的方式配置Spring

JavaConfig是Spring的一个子项目，在Spring 4之后，JavaConfig成为了一个核心功能。

实体类：

```java
@Component
public class User {
    private String name;
    private String age;

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age='" + age + '\'' +
                '}';
    }

    public String getName() {
        return name;
    }
    @Value("spzhang")
    public void setName(String name) {
        this.name = name;
    }

    public String getAge() {
        return age;
    }

    @Value("18")
    public void setAge(String age) {
        this.age = age;
    }
}

```

配置类：

- @Configuration本身也是一个@Component，它所注解的类会被注册到容器中，并且@Configuration代表这是一个这是一个配置类，类似于beans.xml

```java
//@Configuration本身也是一个@Component，它所注解的类会被注册到容器中
//@Configuration代表这是一个这是一个配置类，类似于beans.xml
@Configuration
@ComponentScan("com.spzhang.pojo")
@Import(AppConfig2.class)
public class AppConfig {

    //相当于<bean id="user", class="com.spzhang.pojo.User">
    //方法的名字相当于id，方法返回值相当于class
    //返回值即为要注入到bean中的对象
    @Bean
    public User user() {
        return new User();
    }
}
```

测试类：

-    如果完全使用配置类方式配置bean，则只能**通过AnnotationConfig上下文来获取容器**，**通过配置类的class对象加载**

```java
public class MyTest {
    public static void main(String[] args) {
        //如果完全使用配置类方式配置bean，则只能通过AnnotationConfig上下文来获取容器，通过配置类的class对象加载
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        User getUser = (User) context.getBean("ser");
        System.out.println(getUser.toString());
    }
}
```

# 7. 代理模式

------

为什么要学习代理模式？因为这就是Spring AOP的底层！【Spring AOP和Spring MVC】

代理模式的分类：

- 静态代理
- 动态代理

## 7.1 静态代理

角色分析：

- 抽象角色：一般会使用接口或者抽象类来解决
- 真实角色：被代理的角色
- 代理角色：代理真实角色，代理真实角色后，一般会做一些附属操作
- 客户：访问代理对象的人

代码步骤：

1. 接口

   ```java
   public interface Rent {
       public void rent();
   }
   ```

2. 真实角色

   ```java
   public class Host implements Rent{
       @Override
       public void rent() {
           System.out.println("房东要出租房子");
       }
   }
   ```

3. 代理角色

   ```java
   public class Proxy implements Rent{
       private Host host;
   
       public Proxy() {
       }
   
       public Proxy(Host host) {
           this.host = host;
       }
   
       @Override
       public void rent() {
           seeHouse();
           host.rent();
           hetong();
           fare();
       }
   
       private void seeHouse() {
           System.out.println("看房");
       }
   
   
       private void hetong() {
           System.out.println("签合同");
       }
   
       private void fare() {
           System.out.println("收中介费");
       }
   }
   ```

4. 客户

   ```java
   public class Client {
       public static void main(String[] args) {
           Proxy proxy = new Proxy(new Host());
           proxy.rent();
       }
   }
   ```

静态代理模式的好处：

- 可以使真实角色 的操作更加纯粹，不用去关注一些公共的业务（如房东只需要关注出租即可）
- 公共业务交给代理角色，实现了业务的分工（房屋中介负责带用户看房等公共业务）
- 公共业务发生扩展的时候，方便集中管理

缺点：一个真实角色就会产生一个代理角色；代码量会翻倍，开发效率较低。

**为什么不直接在原来的业务代码上边扩展呢？额外编写代理类不是增加代码量么？**

不用去修改原来的业务代码，避免出错！

## 7.2 动态代理

通过Java的反射机制，解决静态代理中存在的代码量翻倍问题。

- 动态代理和静态代理角色一样

- 动态代理的代理类是动态生成的，不是直接写好的

动态代理分为两大类：

- 基于接口的动态代理
  - JDK的动态代理（原生的）
- 基于类的动态代理
  - cglib
- Java字节码：JAVAssist（拓展）

### 1）JDK动态代理

需要了解两个类：Proxy（代理）、InvocationHandler（调用处理程序）

代码步骤：

1. 接口

   ```java
   public interface Rent {
       public void rent();
   }
   ```

2. 真实角色

   ```java
   public class Host implements Rent{
       @Override
       public void rent() {
           System.out.println("房东要出租房子");
       }
   }
   ```

3. 动态代理生成程序

   1. 实现InvocationHandler接口
   2. 创建一个抽象角色成员变量
   3. 实现invoke()方法，在该方法中完成动态代理要做的任务
   4. 创建获取代理的方法，通过Proxy的静态方法newProxyInstance()返回一个代理，该方法需要三个参数
      1. 当前类加载器
      2. 抽象角色的接口（可以动态生成所有实现该接口的类的代理类）
      3. 一个实现了InvocationHandler接口的对象，可传入this

   ```java
   public class ProxyInvocationHandler implements InvocationHandler {
       private Rent rent;
   
       public void setRent(Rent rent) {
           this.rent = rent;
       }
       public Object getProxy(){
           Object proxy = Proxy.newProxyInstance(this.getClass().getClassLoader(), rent.getClass().getInterfaces(),this);
           return proxy;
       }
       @Override
       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
           Object result = method.invoke(rent, args);
           return result;
       }
   }
   ```

4. 客户

   ```java
   public class Client {
       public static void main(String[] args) {
           //创建真实角色
           Host host = new Host();
           //创建代理创建对象
           ProxyInvocationHandler pih = new ProxyInvocationHandler();
           pih.setRent(host);
           //获取代理对象
           Rent proxy = (Rent) pih.getProxy();
           proxy.rent();
       }
   }
   ```

动态代理的好处：

- 可以使真实角色 的操作更加纯粹，不用去关注一些公共的业务（入房东只需要关注于出租即可）
- 公共业务交给代理角色，实现了业务的分工（房屋中介负责带用户看房等公共业务）
- 公共业务发生扩展的时候，方便集中管理

- **一个动态代理类代理的是一个接口，一般对应一类业务**

- 一个动态代理类可以代理多个类，只要是实现了同一个接口

**proxy这个参数有什么作用？**



# 8. Spring AOP

# AOP

## 1. 背景

AOP：（Aspect Oriented Programming）面向切面编程；

基于OOP基础之上新的编程思想；

指在程序运行期间，**将某段代码动态的切入**到**指定方法的指定位置**进行运行的这种编程方式，面向切面编程；

场景：计算器运行计算方法时进行日志记录

- 编写计算器接口：加减乘除方法
- 编写数学计算器实现计算器接口
- 编写测试器：进行加减乘除运算，同时添加日志记录

加日志记录：

方法1：直接编写在方法内部

- 修改维护麻烦
- 大量冗余的工作
- 日志记录：系统的辅助功能；业务逻辑：核心功能；将日志记录写在业务逻辑中，两者耦合

方法2：抽取创建一个日志工具

- 需要考虑方法的兼容性
- 业务逻辑和系统辅助模块仍然是耦合的

我们希望：

- 日志模块在核心功能运行期间，自己动态的加上

## 2. 动态代理

详情请参考[动态代理](/设计模式/动态代理.md)部分

### 2.1 JDK方式实现动态代理

#### 代码示例

动态代理工厂类

```java
public class CalculatorProxyFactory {

    public static Object getProxyInstance(final Object target) {
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new InvocationHandler() {
                    Object obj = target;
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println("JDK Proxy代理方式");
                        Object value = method.invoke(obj, args);
                        return value;
                    }
                });
    }
}
```

测试代码

```java
public void test1() {
    Calculator proxy = (Calculator) CalculatorProxyFactory.getProxyInstance(new MyMathCalculator());
    proxy.add(1, 2);
}
```

### 2.2 Cglib实现代理工厂类

#### 代码示例

代理工厂类

```java
public class CglibCalculatorProxyFactory implements MethodInterceptor {
    Object target;

    public CglibCalculatorProxyFactory(Object target) {
        this.target = target;
    }

    public Object getProxyInstance() {
        Enhancer en = new Enhancer();
        en.setSuperclass(target.getClass());
        en.setCallback(this);
        return en.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("使用Cglib方式创建动态代理");
        Object value = method.invoke(target, objects);
        return value;
    }
}
```

测试代码

```java
public void test2() {
    CglibCalculatorProxyFactory proxyFactory = new CglibCalculatorProxyFactory(new MyMathCalculator());
    Calculator proxy = (Calculator) proxyFactory.getProxyInstance();
    proxy.add(1, 2);
}
```

Spring AOP是Spring框架体系结构中非常重要的功能模块之一，该模块提供了面向切面编程实现。面向切面编程在事务处理、日志记录、安全控制等操作中被广泛使用。

## 8.1 AOP的概念

AOP即面向切面编程，它**采取横向抽取机制**，将分散在各个方法中的重复代码（大多是**日志记录、性能统计、安全控制、事务处理、异常处理等**）提取出来，然后**在程序编译或运行阶段**将这些抽取出来的代码应用到要执行的地方。

Spring AOP框架常用术语如下：

1. 切面（Aspect）：指封装横切到系统功能的类；
2. 连接点（JoinPoint：程序运行的一些时间点）；
3. 切入点：需要处理的连接点。在Spring AOP中，所有方法执行都是连接点，通过切入点确定哪些连接点需要被处理；
4. 通知：由切面添加到特定的连接点（满足切入点规则）的一段代码；
5. 引入：
6. 目标对象：所有被通知的对象；
7. 代理：通知应用到目标对象之后被动态创建的对象；
8. 织入：将切面代码插入到目标对象上，从而生成代理对象的过程。AOP有三种实现：编译器织入（AspectJ采用），需要有特殊的Java编译器；类装载器织，需要有特殊的类装载器入；动态代理织入（Spring AOP框架默认）。

## 4.2 基于Spring原生API的AOP实现

> 主要是Spring API接口的实现

### 1）添加aspectjweaver依赖

```xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.13</version>
</dependency>
```

### 2）创建抽象接口及真实角色类

### 3）通过实现不同类型的Advice接口来定义横切逻辑

Spring AOP支持5种类型的Advice：

| 通知类型     | 连接点               | 实现接口                                        |
| ------------ | -------------------- | ----------------------------------------------- |
| 前置通知     | 方法执行前           | org.springframework.aop.MethodBeforeAdvice      |
| 后置通知     | 方法执行后           | org.springframework.aop.AfterReturningAdvice    |
| 环绕通知     | 方法前后             | org.aopalliance.intercept.MehtodInterceptor     |
| 异常抛出通知 | 方法抛出异常         | org.springframework.aop.ThrowsAdvice            |
| 引介通知     | 类中增加新的方法属性 | org.springframework.aop.IntroductionInterceptor |

```java
public class AfterLog implements AfterReturningAdvice {
    @Override
    public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
        System.out.println("当前执行的方法：" + method.getName() + "执行结果为： " + returnValue);
    }
}
```

### 4）在xml中配置aop

```xml
<!-- 配置aop:需要导入aop的约束 -->
<aop:config>
    <!-- 切入点: expression:表达式，execution(要执行的位置！* * * *) -->
    <aop:pointcut id="pointcut" expression="execution(* com.spzhang.service.UserServiceImpl.*(..))"/>
    <!-- 执行环绕增加！ -->
    <aop:advisor advice-ref="log" pointcut-ref="pointcut" />
    <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut" />
</aop:config>
```

#### I. 配置切入点\<aop:pointcut>

- id：切入点的唯一标识
- expression：要切入的位置
  - 如：execution(* com.spzhang.service.UserServiceImpl.*(...))
    - 返回类型、方法名、参数列表

#### II. 配置环绕\<aop:advisor>

- advice-ref：横切面bean的id
- pointcut-ref：该横切面对应的切入点

## 4.3 使用自定义类实现Spring AOP

> 主要是切面定义

### 1）添加aspectjweaver依赖

```xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.13</version>
</dependency>
```

### 2）创建抽象接口及真实角色类

### 3）创建自定义的切面类

```java
public class DiyPointCut {
    public void before(){
        System.out.println("-----方法执行前------");
    }
    public void after(){
        System.out.println("-----方法执行后------");
    }
}
```

### 4）在xml中配置aop

```xml
    <!-- 方法二：自定义类 -->
    <bean id="diyPointCut" class="com.spzhang.diy.DiyPointCut" />

    <aop:config>
        <!-- 自定义切面，ref 要引用的类 -->
        <aop:aspect ref="diyPointCut">
            <!-- 切入点 -->
            <aop:pointcut id="pointcut" expression="execution(* com.spzhang.service.UserServiceImpl.*(..))"/>
            <!-- 通知 -->
            <aop:before method="before" pointcut-ref="pointcut"/>
            <aop:after-returning method="after" pointcut-ref="pointcut"/>
        </aop:aspect>
    </aop:config>
```

#### I. 自定义切面\<aop:aspect>

- ref：自定义的切面类的bean对象的id

##### ① 配置切入点\<aop:pointcut>

- id：切入点的唯一标识
- expression：要切入的位置
  - 如：execution(* com.spzhang.service.UserServiceImpl.*(...))
    - 返回类型、方法名、参数列表

##### ② 配置环绕\<aop:before>

- method：自定义切面中在切入点执行前执行的方法名
- pointcut-ref：该横切面对应的切入点

## 4.4 基于XML配置开发的AspectJ

AspectJ是一个基于Java语言的AOP框架。

## 4.5 基于注解的AspectJ

在使用AspectJ框架时，需要先添加相应的依赖：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>5.0.8.RELEASE</version>
</dependency>
```

主要注解如下：

- **@Aspect**：用于定义一个切面，注解在切面类上
- **@Pointcut**：定义切入点表达式，使用时需要定义一个切入点方法，该方法是一个返回值void且方法体伪空的普通方法
- **@Before**：前置通知
- **@AfterReturning**：用于定义后置返回通知
- **@Around**：用于定义环绕通知，在目标方法执行前后实施增强
- **@AfterThrowing**：用于定义异常通知，在方法抛出异常后实施增强，可用于处理异常，记录日志等
- **@After**：用于定义后置（最终通知），在目标方法执行后实施增强（不管是否发生异常都执行，可用于释放资源）

**执行顺序**：环绕前-->前置通知-->方法执行-->后置通知（AfterReturing）-->方法执行后（After最终通知）-->环绕后

基于注解开发AspectJ的具体步骤如下：

### 1）创建切面类

创建切面类时需要注意以下几点：

1. 类使用@Aspect、@Component进行注解

2. **使用@Pointcut注解切入点表达式**，并**通过定义方法来表示切入点名称**

   - 切入点表达式execution(* dynamic.jdk.\*.\*(...))：
     - 第一个*表示匹配任意的方法的返回类型
     - dynamic.jdk.*表示匹配dynamic.jdk包下的所有类
     - 第三个*表示匹配该类中的所有方法
     - (...)表示匹配方法的任意参数

3. 每个通知方法添加相应的注解，将切入点名称作为参数传递给要执行增强的通知方法

   ```java
   @Aspect
   @Component
   public class MyAspect {
       @Pointcut("execution(* dynamic.jdk.*.*(...))")
       private void myPointcut(){
   
       }
       @Before("myPointcut()")
       public void before(JoinPoint jp){
           out.println("前置通知：模拟权限控制");
           out.println(", 目标对象： " + jp.getTarget() +
                   ", 被增强处理的方法: " + jp.getSignature().getName());
       }
       @Around("myPointcut()")
       public Object around(ProceedingJoinPoint pjp) throws Throwable{
           out.println("环绕开始");
           Object obj = pjp.proceed();
           out.println("环绕结束");
           return obj;
       }
       @AfterThrowing(value="myPointcut()", throwing="e")
       public void except(Throwable e){
           e.printStackTrace();
       }
   }
   ```

### 3）创建配置文件

配置文件中指定需要扫描的包，还需要启动基于注解的AspectJ支持。

```xml
<!-- 需要引入：http://www.springframework.org/schema/aophttp://www.springframework.org/schema/aop/spring-aop.xsd--><aop:aspectj-autoproxy/>
```



# 5. Spring的事务管理

## 5.1 Spring的数据库编程

> 如何使用jdbcTemplate来完成对数据库的各种操作

Spring框架提供了**JDBC 模板模式**，即jdbcTemplate来进行数据库编程（不太常用）。

首先需要**导入相关的依赖**：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.0.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.11</version>
</dependency>
```

其中，主要使用Spring JDBC模块的**core包**和**dataSource包**。前者是JDBC的核心功能包，包括常用的JdbcTemplate类，dataSource包是访问数据源的工具类包。要使用Spring JDBC操作数据库，首先需要**在appContext.xml中配置数据源和JDBC模板**：

```xml
<!-- 配置数据源 -->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <!-- MySQL数据库驱动 -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <!-- 连接数据库的URL -->
    <property name="url" value="jdbc:mysql://localhost:3306/springtest?serverTimezone=UTC"/>
    <!-- 连接数据库的用户名 -->
    <property name="username" value="root"/>
    <!-- 连接数据库的密码 -->
    <property name="password" value="love..1028"/>
</bean>
<!-- 配置JDBC模板 -->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

其中**数据源（dataSource）**的配置主要完成**连接数据库**的功能，需要通过其子元素<property>来配置数据源连接数据库需要使用的**数据库驱动（driverClassName）**、**数据库URL(url)**、**用户名（username）**、**密码（password）**。

- driverClassName：com.mysql.jdbc.Driver
- url：jdbc:mysql://localhost:3306/databaseName?serverTimezone=UTC
- username：root
- passwork：密码

**JDBC模板的配置**，需要**将数据源dataSource作为依赖注入到jdbcTemplate中**：

JdbcTemplate类常用的方法：

- public int **update(String sql, Obejct[] args)**: 对数据表完成**增加、修改、删除**等操作。args[]设置SQL语句中的参数。函数返回更新的行数
- public List<T> **query(String sql, RowMapper<T> rowMapper, Object[] args)**：对数据表进行查询操作。**rowMapper将结果集映射到用户自定义的类中**（要求自定义类中的属性要与数据表的字段对应）

### 使用实例：

#### 1. 创建应用并添加依赖

要使用JDBCTemplate进行数据库编程，需要添加依赖：spring-jdbc和mysql-connector-java。

注意：mysql 8.x和mysql 5.x的验证模块不同，**倾向使用较新版本的mysql来避免出现登陆问题**，如java.sql.SQLException: Unable to load authentication plugin 'caching_sha2_password'。

此外，**mysql jdbc 6.0以上的版本，在提供url时，必须配置serverTimeZone参数**。该参数表示数据库记录时间时会采用的时区。可以将其配置为北京时区。

```java
jdbc:mysql://localhost:3306/ssh?serverTimezone=Asia/Shanghai
```

#### 2. 创建配置文件

在配置文件中主要**完成dataSource和jdbcTemplate的配置**。此外，需要**指定要扫描的包**，使得注解生效。

```xml
<context:component-scan base-package="com.ch5"/>
```

#### 3. 创建实体类

要求实体类的属性与数据表中的字段一致。

#### 4. 创建数据访问层Dao

- **TestDao接口**
- **TestDaoImpl实现类**：使用@Repository("testDao")进行注解（配置文件中添加扫描声明），并使用JdbcTemplae访问数据库

```java
@Repository("testDao")
public class TestDaoImpl implements TestDao{
    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Override
    public int update(String sql, Object[] param) {
        return jdbcTemplate.update(sql, param);
    }

    @Override
    public List<MyUser> query(String sql, Object[] param) {
        RowMapper<MyUser> rowMapper = new BeanPropertyRowMapper<>(MyUser.class);
        return jdbcTemplate.query(sql, rowMapper, param);
    }
}
```

#### 5. 创建测试类

```java
public class TestSpringJDBC {
    public static void main(String[] args){
        ApplicationContext appContext = new ClassPathXmlApplicationContext("appContext.xml");
        TestDao td = (TestDao) appContext.getBean("testDao");
        String insertSql = "insert into user values(?, ?, ?)";
        //数组param的值与insertSql语句中的？一一对应
        Object[] param1 = {"1","cheng1", "男"};
        Object[] param2 = {"2","cheng2", "女"};
        Object[] param3 = {"3","cheng3", "男"};
        Object[] param4 = {"4","cheng4", "男"};
        //添加用户
        td.update(insertSql, param1);
        td.update(insertSql, param2);
        td.update(insertSql, param3);
        td.update(insertSql, param4);
        //查询用户
        String selectSql = "select * from user";
        List<MyUser> list = td.query(selectSql, null);
        for(MyUser user: list){
            System.out.println(user);
        }
    }
}
```

## 5.2 编程式事务管理

> 需要在代码中显式调用与事务处理相关的方法

### 1）基于底层API的编程式事务管理

涉及的核心接口：PlatformTransactionManager、TransactionDefinition、TransactionStatus。

#### 1. 配置事务管理器

**使用**PlatformTransactionManager接口的实现类**DataSourceTransactionManager为数据源添加事务管理器**：

```xml
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>	
</bean>
```

#### 2. 创建数据访问类

创建数据访问类，使用注解**将jdbcTemplate和txManager注入到该数据访问类中**，并**通过事务管理器来完成事务管理**：

1. **创建事务定义**
2. **开启事务**
3. **提交事务**
4. **如果出现异常，则事务回滚**

```java
@Repository("codeTransaction")
public class CodeTransaction {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Autowired
    private DataSourceTransactionManager txManager;
    public String test(){
        //默认事务定义
        TransactionDefinition tf = new DefaultTransactionDefinition();
        //开启事务
        TransactionStatus ts = txManager.getTransaction(tf);
        String message = "执行成功，没有事务回滚";
        try {
            //使用jdbc进行数据库操作
			...
            //提交事务
            txManager.commit(ts);
        } catch(Exception e){
            //出现异常，事务回滚
            txManager.rollback(ts);
            message = "主键重复，事务回滚";
            e.printStackTrace();
        }
        return message;
    }
}
```

#### 3. 创建测试类

```java
public class TestCodeTransaction {    public static void main(String[] args){        ApplicationContext appContext = new ClassPathXmlApplicationContext("appContext.xml");        CodeTransaction codeTransaction = (CodeTransaction) appContext.getBean("codeTransaction");        System.out.println(codeTransaction.test());    }}
```

### 2）基于TransactionTemplate的编程式事务管理

基于底层API事务管理将事务处理的代码散落在业务逻辑代码中，破坏了原有代码的条理性，并且每一个业务方法都包含了类似的启动事务、提交以及回滚事务的样板代码。可以**借助事务模板TransactionTemplate来隐藏任何调用事务处理API的代码**。

#### 1. 为事务管理器添加事务模板

事务模板位于org.springframework.transaction.support.TransactionTemplate，需要将事务管理器作为参数传递给事务模板。

```xml
<bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">	<property name="transactionManager" ref="txManager"/></bean>
```

#### 2. 创建数据访问类

创建数据访问类，使用注解**将jdbcTemplate和transactionTemplate注入到该数据访问类中**，并**通过transactionTemplate来完成事务管理**：

1. 调用**其execute(TransactionCallback<T> callback)方法**
2. **创建匿名内部类，实现其doInTransaction(TransactionStatus status)方法**，在该方法中实现业务逻辑代码（对数据库的操作）

```java
public class TransactionTemplateDao {    @Autowired     private JdbcTemplate jdbcTemplate;    @Autowired     private TransactionTemplate transactionTemplate;    String message = "";    public String test(){        //以匿名内部类的方式实现TransactionCallback接口，使用默认的事务提交和回滚规则，在业务代码中不需要显式调用任何事务处理的API        transactionTemplate.execute(new TransactionCallback<Object>() {            @Override            public Object doInTransaction(TransactionStatus status) {                try {                	//使用jdbcTemplate对数据库进行操作					...                    message = "执行成功，没有事务回滚";                } catch(Exception e){                    message = "主键重复，事务回滚";                    e.printStackTrace();                }                return message;            }        });        return message;    }}
```

#### 3. 创建测试类

与 5.2的1）部分中创建测试类相同。

## 5.3 声明式事务管理 

> Spring的**声明式事务管理是通过AOP技术实现的**，**本质是对方法前后进行拦截**，在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行结果提交或者回滚事务。

声明式事务管理不需要在业务逻辑代码中掺杂事务处理的代码，只需相关的**事务规则声明**便可将事务规则应用到业务逻辑中。声明式事务管理最细力度只能作用到方法级别，编程式事务管理可以作用到代码块级别。

### 1）基于XML方式的声明式事务管理

这种方式主要需要进行如下配置：

- 在配置文件中**配置事务规则**
- 编写AOP让Spring自动对目标对象生成代理

##### 1. 添加相关依赖（需要添加AOP依赖）

```xml
<dependency>    <groupId>org.springframework</groupId>    <artifactId>spring-aop</artifactId>    <version>5.0.8.RELEASE</version></dependency><dependency>    <groupId>org.springframework</groupId>    <artifactId>spring-aspects</artifactId>    <version>5.0.8.RELEASE</version></dependency>
```

#### 2. 创建Dao层

Dao层主要创建一个TestDao接口和TestDaoImpl实现类，其中接口中定义了两个数据操作方法，即save和delete。TestDaoImpl类的代码如下：

```java
@Repository("testDao")public class TestDaoImpl implements TestDao{    @Autowired    private JdbcTemplate jdbcTemplate;    @Override    public int save(String sql, Object[] param) {        return jdbcTemplate.update(sql, param);    }    @Override    public int delete(String sql, Object[] param) {        return jdbcTemplate.update(sql, param);    }}
```

#### 3. 创建Service层

Service层主要创建一个TestService接口和TestServiceImpl实现类，其中接口中定义了两个数据操作方法，即save和delete。TestServiceImpl类的代码如下：

```java
@Service("testService")public class TestServiceImpl implements TestService{    @Autowired    private TestDao testDao;    @Override    public int save(String sql, Object[] param) {        return testDao.save(sql, param);    }    @Override    public int delete(String sql, Object[] param) {        return testDao.delete(sql, param);    }}
```

#### 4. 创建Controller层

```java
@Controllerpublic class StatementController {    @Autowired    private TestService testService;    public String test(){        String message = "";        try {            String sql = "delect from user ";            String sql1 = "insert into user value(?,?,?) ";            Object[] param = {"1", "cheng1", "男"};            testService.delete(sql, null);            testService.save(sql, param);            testService.save(sql, param);        } catch(Exception e){            message = "主键重复，事务回滚";            e.printStackTrace();        }        return message;    }}
```

#### 5. 创建配置文件

在配置文件中，需要**编写通知声明事务**以及**编写AOP，让Spring自动对目标对象生成代理**。

```xml
<!-- 编写通知声明事务 --><tx:advice id="myAdvice" transaction-manager="txManager">    <tx:attributes>        <!-- 表示任意方法 -->        <tx:method name="*"/>    </tx:attributes></tx:advice><!-- 编写AOP，让Spring自动对目标对象生成代理，需要使用AspectJ表达式 --><aop:config>    <!-- 定义切入点 -->    <aop:pointcut id="txPointCut" expression="execution(* com.statement.service.*.*())"/>    <!-- 切面:将切入点与通知关联 -->    <aop:advisor advice-ref="myAdvice" pointcut-ref="txPointCut"/></aop:config>
```

#### 6. 创建测试类

```java
public class XMLTest {    public static void main(String[] args){        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("appContext.xml");        StatementController statementController = (StatementController) applicationContext.getBean("statementController");        System.out.println(statementController.test());    }}
```

### 2）基于@Transactional注解的声明式事务管理

**@Transactional注解**可以作用于接口、接口方法、类以及类的方法上。当**作用于类上时，该类的所有public方法都将具有该类型的事务属性**，同时也可以在方法级别使用该注解来覆盖类级别的定义。虽然@Transactional注解可以作用于接口、接口方法、类以及类的方法上，但是Spring小组建议不要在接口或者接口方法上使用该注解，因为它只有在使用基于接口的代理时才会生效。

#### 1. 创建配置文件

使用@Transactional注解进行声明式事务管理，需要**在配置文件中为事务管理器注册注解驱动器**：

```xml
<beans xmlns:tx="http://www.springframework.org/schema/tx"       xsi:schemaLocation="http://www.springframework.org/schema/tx            http://www.springframework.org/schema/tx/spring-tx.xsd">	<tx:annotation-driven transaction-manager="txManager"/></beans>
```

#### 2. 为Service层添加@Transactional注解

在Spring MVC中通常通过Service层进行事务管理，**为Service类添加@Transactional注解**后，就可以指定这个类需要受Spring的事务管理。

如果不想对某个异常进行事务处理，可以使用以下代码：

```java
@Transaction(rollbackFor=RuntimeException.class)	//不对RuntimeException回滚生效
```

注意：**@Transactional只能针对public属性范围内的方法添加**

# 9. 整合Mybatis到Spring（ MyBatis-Spring）

> http://mybatis.org/spring/zh/index.html

## 9.1 基本介绍

MyBatis-Spring 会帮助你将 MyBatis 代码无缝地整合到 Spring 中。它将允许 MyBatis 参与到 Spring 的事务管理之中，创建映射器 mapper 和 `SqlSession` 并注入到 bean 中，以及将 Mybatis 的异常转换为 Spring 的 `DataAccessException`。最终，可以做到应用代码不依赖于 MyBatis，Spring 或 MyBatis-Spring。

动机：spring 3.0在mybatis3.0开发之前就开发完了，mybatis社区召集爱好者编写将spring的集成作为mybatis的一个子项目。

知识基础：

mybatis-spring 2.0对应mybatis 3.5+， spring 5.0+， Java 8+



![image-20200830101645461](1. Spring入门.assets/image-20200830101645461.png)

## 9.2 入门

### 1）添加依赖

- mybatis-spring依赖仅仅是帮助spring整合mybatis的模块，此外还需要添加mybatis模块

```xml
<!-- 添加mybatis-spring帮助整合mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.1</version>
</dependency>
```

- mybatis
- spring-jdbc：Spring要操作数据库的时候，需要添加spring-jdbc依赖，其中有提供DataSource

```xml
<!-- 添加jdbc依赖，用于spring操作数据库 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.0.8.RELEASE</version>
</dependency>
```

- mysql-connector-java：配置DataSource时需要连接到mysql数据源的驱动

  ```xml
  <!-- 用于配置数据源 时提供连接到mysql的驱动 -->
  <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.37</version>
  </dependency>
  ```

  

### 2）基本使用

要和 Spring 一起使用 MyBatis，需要**在 Spring 应用上下文**中定义至少两样东西：

- 一个 `SqlSessionFactory` 
- 至少一个数据映射器类

#### I. SqlSessionFactory

在 MyBatis-Spring 中，可使用 `SqlSessionFactoryBean`来创建 `SqlSessionFactory`（Mybatis中使用SqlSessionFactoryBuilder的builder()方法来创建）。 在spring上下文环境配置文件中创建SqlSessionFactory时，需要设置几个属性（以下属性不需要再mybatis核心配置文件中配置）：

- **dataSource**：数据源需要在这个配置文件中配置
- **configLocation**：传入mybatis核心配置文件的路径

- **mapperLocations**：配置mapper.xml（可以选择还放到mybatis-config.xml中配置）

```xml
<!-- SqlSessionFactory -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <!-- 绑定Myabtis配置文件 -->
    <property name="configLocation" value="classpath:mybatis-config.xml" />
    <!-- 配置mapper.xml -->
    <property name="mapperLocations" value="classpath:com/spzhang/mapper/*.xml" />
</bean>
```

注意：`SqlSessionFactory` 需要一个 `DataSource`（数据源）。 这可以是任意的 `DataSource`，只需要和配置其它 Spring 数据库连接一样配置它就可以了。

```xml
<!-- DataSource：使用spring的数据源替换Mybatis的配置， c3p0 dbcp  druid -->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    <property name="url" value="jdbc:mysql://localhost:3306" />
    <property name="username" value="root" />
    <property name="password" value="love..1028" />
</bean>
```

#### II. 数据映射器类/Mapper接口

编写Mapper接口和对应的Mapper.xml文件：

- Mapper接口：

  ```java
  public interface UserMapper {
      public User selectUser(int id);
  }
  ```

- Mapper.xml文件（也可以使用注解的方式完成接口的实现，都需要在Mybatis核心配置文件或者sqlSessionFactory标签中配置）

  ```xml
  <mapper namespace="com.spzhang.mapper.UserMapper">
      <select id="selectUser" resultType="User">
          select * from user where id=#{id}
      </select>
  </mapper>
  ```

#### III. 获取Mapper对象

有两种获取mapper对象的方式：

- **SqlSessionTemplate**
- **SqlSessionDaoSupport**
- **MapperFactoryBean**

##### ① SqlSessionTemplate

mybatis-spring提供了SqlSessionTemplate类（类似于Mybatis中的Sqlsession）用来用数据库进行交互

- 可以在xml中配置一个bean，要设置sqlSessionFactory属性
- 该类没有set方法，只能通过构造器的方式设置sqlSessionFactory属性

```xml
<!-- SqlSessionTemplate就是使用Mybatis时的SqlSession -->
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
    <!-- 只能使用构造器方式注入SqlSessionFactory， 因为SqlSessionTemplate中没有set方法 -->
    <constructor-arg index="0" ref="sqlSessionFactory" />
</bean>
```

1. 创建MapperImpl，并通过构造方法注入的方式注入SqlSessionTemplate对象

   ```xml
   <bean id="userMapper1" class="com.spzhang.mapper.UserMapperImpl1">    <constructor-arg index="0" ref="sqlSession" /></bean>
   ```

2. 使用SqlSessionTemplate对象来实现接口方法

```java
public class UserMapperImpl1 implements UserMapper{    private SqlSessionTemplate sqlSession;    public UserMapperImpl1(SqlSessionTemplate sqlSession) {        this.sqlSession = sqlSession;    }    @Override    public User selectUser(int id) {        return sqlSession.getMapper(UserMapper.class).selectUser(id);    }}
```

##### ② SqlSessionDaoSupport

`SqlSessionDaoSupport` 是一个抽象的支持类，用来为你提供 `SqlSession`。调用 `getSqlSession()` 方法你会得到一个 `SqlSessionTemplate`，之后可以用于执行 SQL 方法，就像下面这样:

```java
public class UserMapperImpl2 extends SqlSessionDaoSupport implements UserMapper {    @Override    public User selectUser(int id) {        return getSqlSession().getMapper(UserMapper.class).selectUser(id);    }}
```

在这个类里面，通常更倾向于使用 `MapperFactoryBean`，因为它不需要额外的代码。但是，如果你需要在 DAO 中做其它非 MyBatis 的工作或需要一个非抽象的实现类，那么这个类就很有用了。

`SqlSessionDaoSupport` 需要通过属性设置一个 `sqlSessionFactory` 或 `SqlSessionTemplate`。如果两个属性都被设置了，那么 `SqlSessionFactory` 将被忽略。

```xml
<bean id="userMapper2" class="com.spzhang.mapper.UserMapperImpl2">    <property name="sqlSessionFactory" ref="sqlSessionFactory" /></bean>
```

##### ③ MapperFactoryBean

通过MapperFactoryBean创建实现接口的对象，需要通过set方法注入两个属性：

- mapperInterface：要创建的对象实现的接口
- sqlSessionFactory：创建的sqlSessionFactory

```xml
<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">  <property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper" />  <property name="sqlSessionFactory" ref="sqlSessionFactory" /></bean>
```

`MapperFactoryBean` 将会负责 `SqlSession` 的创建和关闭。如果使用了 Spring 的事务功能，那么当事务完成时，session 将会被提交或回滚。最终任何异常都会被转换成 Spring 的 `DataAccessException` 异常。

# 10. 声明式事务

## 10.1 回顾事务

- 一组业务要么都成功，要么都失败
- 确保完整性和一致性

事务ACID原则：

- 原子性
- 一致性
- 隔离性
  - 多个事务可能操作同一个资源。防止数据损坏
- 持久性

## 10.2 Spring事务

Spring支持两种事务：

- 编程式事务
- 声明式事务：AOP

### 1）配置声明式事务

#### I. 创建DataSourceTransactionManager对象

```xml
<!-- 配置声明式事务 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>
```

#### II. 结合AOP实现事务的织入

```xml
<!-- 结合AOP实现事务的织入 -->
<!-- 配置事务通知： -->
<tx:advice id="txAdvice" transaction-manager=" ">
    <!-- 给哪些方法配置事务 -->
    <!-- 配置事务的传播特性 -->
    <tx:attributes>
        <tx:method name="add*"/>
        <tx:method name="delete*" />
        <tx:method name="update*" />
        <!-- 指定所有方法都配置事务 -->
        <tx:method name="*" />
    </tx:attributes>
</tx:advice>

<!-- 配置事务切入 -->
<aop:config>
    <aop:pointcut id="txPointCut" expression="execution(* com.spzhang.mapper.*.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointCut" />
</aop:config>
```


# 0. Mapper文件标签

```xml
<select></select>
<insert></insert>
<update></update>
<delete></delete>
<resultMap></resultMap>
<sql></sql>
<cache></cache>
```

# 1. 简介

------

## 1.1 什么是Mybatis

- Mybatis是一款优秀的**持久层框架**，它支持自定义 SQL、存储过程以及高级映射
- MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。
- MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

**如何获取Mybatis？**

- Maven仓库

  ```xml
  <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.5</version>
  </dependency>
  ```

- Github：https://github.com/mybatis/mybatis-3

- 中文文档：https://mybatis.org/mybatis-3/zh/index.html

## 1.2 持久化

数据持久化

- 持久化就是将程序的数据在持久状态和瞬时状态转化的过程
- 内存：断电即失
- 数据库（Jdbc），io文件持久化

为什么需要持久化？

- 有一些对象，不能让它丢掉
- 内存太贵

## 1.3 持久层

**Dao层**、Service层、Controller层……

- 完成持久化工作的代码块
- 层界限十分明显

## 1.4 为什么需要Mybatis？

- 方便
- 传统的JDBC代码太复杂了，使用Mybatis简化、框架、自动话

# 2. 第一个Mybatis程序

思路：搭建环境-->导入Mybatis-->编写代码-->测试！

**需要注意的问题：**

1. 没有注册Mapper文件
2. 绑定接口错误
3. 方法名不对
4. 返回类型不对
5. Maven导出资源问题

## 2.1 搭建环境

搭建数据库：

```mysql
CREATE DATABASE `mybatis`;

USE `mybatis`;
`mybatis`

CREATE TABLE `user` (
	`id` INT(20) NOT NULL PRIMARY KEY,
	`name` VARCHAR(30) DEFAULT NULL,
	`pwd` VARCHAR(30) DEFAULT NULL
) ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `user` (`id`,`name`,`pwd`) VALUES
(1,'张三', '1234'),
(2,'李四', '1234'),
(3,'小红', '1234')
```

新建项目：

1. 先建一个普通的Maven项目

2. 删除src目录

3. 导入Maven依赖

   - mysql驱动
   - Mybatis依赖
   - junit依赖

   ```xml
    <!-- 导入依赖 -->
       <dependencies>
           <!-- mysql驱动 -->
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>5.1.45</version>
           </dependency>
           <!-- Mybatis -->
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>3.4.5</version>
           </dependency>
           <!-- junit -->
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
               <scope>test</scope>
           </dependency>
       </dependencies>
   ```

## 2.2 创建子模块

- 编写Mybatis核心配置文件

- 编写Mybatis工具类

  - 通过SqlSessionFactoryBuilder根据配置文件获取SqlSessionFactory对象
  - 通过SqlSessionFactory对象获取SqlSession

  ```java
  //创建SqlSessionFactory对象，并通过该对象返回SqlSession
  public class MybatisUtils {
      private static SqlSessionFactory sqlSessionFactory;
      static {
          String resource = "mybatis-config.xml";
          try {
              //获取配置文件
              InputStream inputStream = Resources.getResourceAsStream(resource);
              sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
      //获取SqlSession对象
      public static SqlSession getSqlSession() {
          //参数true表示开启事务自动提交
          return sqlSessionFactory.openSession(true);
      }
  }
  ```

## 2.3 编写代码

- pojo实体类

  ```java
  public class User {
      private String name;
      private String age;
      //get、set、toString()方法
  ```

- Dao/Mapper接口

  ```java
  public interface UserMapper {
      List<User> getUserList();
  }
  ```

- Dao/Mapper接口实现类：由UserDaoImpl变为**UserMapper.xml**（在UserDao接口下创建）

  - **namespace**：对应要绑定的Dao/Mapper接口
  - **\<select>标签**
    - **id**：与要实现的接口的方法名一致
    - **resultType**：要返回的对象类型

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.spzhang.dao.UserDao">
      <select id="getUserList" resultType="com.spzhang.pojo.User">
      select * from mybatis.user
    </select>
  </mapper>	
  ```

- **在Mybatis核心配置文件中注册Mapper.xml**

  - 否则会报错：org.apache.ibatis.binding.BindingException: Type interface com.spzhang.dao.UserDao is not known to the MapperRegistry.
  - 注意：resource路径必须使用斜杠

  ```xml
  <mappers>
      <mapper resource="com/spzhang/dao/UserMapper.xml" />
  </mappers>
  ```

这里需要注意，Maven约定大于配置，需要**在Maven中设置过滤规则（可以在父项目pom.xml中配置）**，否则会报错：Cause: org.apache.ibatis.builder.BuilderException: Error parsing SQL Mapper Configuration. Cause: java.io.IOException: Could not find resource。

```xml
<!-- 在build中配置resources，防止资源导出失败的问题 -->
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```

## 2.4 编写测试类

### 1）通过getMapper的方式获取Dao/Mapper对象

- 获取SqlSession对象
- 通过SqlSession对象获取Mapper对象（Dao层对象）
- 通过Mapper对象执行相应的数据库操作
- 关闭SqlSession连接

```java
public class UserDaoTest {    @Test    public void test() {        //第一步，获取SqlSession对象        SqlSession sqlSession = MybatisUtils.getSqlSession();        //方式一：getMapper        UserMapper userMapper = sqlSession.getMapper(UserDao.class);        List<User> userList = userMapper.getUserList();        for(User user: userList) {            System.out.println(user);        }        //关闭SqlSession        sqlSession.close();    }}
```

### 2）通过SqlSession对象执行Dao接口方法

```java
sqlSession.selectList("com.spzhang.dao.UserMapper.getUserList");
```



# 3. CRUD

1. 编写接口方法
2. Mapper配置文件中实现接口方法
3. 通过SqlSession获取Mapper对象
4. 执行方法
5. 通过SqlSession提交事务（增删改）
6. 关闭SqlSession

## 3.1 select

在Mapper配置文件中，可以通过<select></select>来实现Dao/Mapper接口中的select方法。该标签中有如下属性：

- **id**：id应该与要实现的接口中的方法名相同
- **parameterType**：要实现的方法中的参数类型
- **resultType**：查询结果的返回类型

\<select>\</select>标签中可以书写具体的查询语句

- 具体语句中可以通过#{}来获取参数的值

```xml
<select id="getUserById" parameterType="int" 				                         resultType="com.spzhang.pojo.User">
    select * from mybatis.user where id = #{id}
</select>
```

## 3.2 insert

- 对象中的属性可以通过#{}直接取出来

```xml
<!--对象中的属性可以直接取出来-->
<insert id="addUser" parameterType="com.spzhang.pojo.User">
    insert into mybatis.user (id, name, pwd) values (#{id}, #{name}, #{pwd})
</insert>
```

**注意：在增删改时需要提交事务！**

```java
@Test
public void addUser() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    userMapper.addUser(new User(4, "帅鹏", "12345"));
    //必须要提交事务
    sqlSession.commit();
    sqlSession.close();
}
```

## 3.3 update

```xml
<update id="updateUser" parameterType="com.spzhang.pojo.User">
    update mybatis.user set name=#{name}, pwd=#{pwd} =  where id=#{id};
</update>
```

## 3.4 delete

```xml
<delete id="deleteUser" parameterType="int">
    delete from mybatis.user where id=#{id}
</delete>
```

## 3.5 常见错误

- 标签匹配错误

- Mybatis核心配置文件中配置Mapper文件时，resource路径应该使用/

- xml中1字节的UTF-8序列的字节1无效

  - encoding="UTF-8"改为encoding="UTF8"

# 4. 万能Map

假设，实体类或者数据库中的表字段或者参数过多，应当考虑使用Map！

Map传递参数，直接在sql中取出key即可

对象传递参数，直接在sql中取出对象的属性即可

只有一个基本基本类型参数的情况下，可以直接在sql中取到

## 4.1 使用Map插入数据

Mapper：

```xml
<!--获取map中的key-->
<insert id="addUser2" parameterType="map">
    insert into mybatis.user (id, name, pwd) values (#{userId}, #{userName}, #{userPwd})
</insert>	
```

调用：

```java
UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
Map<String, Object> map = new HashMap<String, Object>();
map.put("userId", 5);
map.put("userName", "小白");
map.put("userPwd", "123");

userMapper.addUser2(map);
```

## 4.2 模糊查询

> 避免出现SQL注入

### 1）Java执行时，传递通配符%

```java
List<User> userList = mapUserMapper.getUserLike("%小%");
```

### 2）在sql拼接中使用通配符

```xml
<select id="getUserLike" resultType="com.spzhang.pojo.User">
    select * from mybatis.user where name like "%"#{value}"%"
</select>
```

# 5. 配置解析

## 5.1 属性（Properties）

可以通过properties属性来实现引用配置文件！

这些属性可以在外部进行配置，并可以进行动态替换。你既可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置。【db.properties，位于resource资源文件夹下】

### 1）编写外部配置文件

在resource资源文件夹下编写dp.properties文件

- 记得在pom.xml中添加扫描到该文件的依赖支持！

```properties
drive=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost::3306/mybatis
username=root
password=lvoe..1028
```

### 2）引入外部配置文件

Mybatis核心配置文件中通过\<properties />标签引入外部配置文件：

```xml
<!-- 引入外部配置文件 -->
<properties resource="dp.properties" />
```

也可以在核心配置文件内定义一些属性：

```xml
<!-- 引入外部配置文件 -->
<properties resource="dp.properties">
    <property name="username" value="root"/>
    <property name="password" value="123456"/>
</properties>
```

### 3）引用properties变量

在Mybatis核心配置文件引用外部配置文件中的变量：

```xml
<property name="driver" value="${driver}"/>
<property name="url" value="${url}"/>
<property name="username" value="${root}}"/>
<property name="password" value="${password}"/>
```

- 也可以用上述方式引用在核心配置文件内定义的properties变量

- 但是其中有与外部配置文件重合的变量，则引用时会引用外部配置文件
  - 首先读取在properties元素体内指定的属性
  - 最后再读取作为方法参数传递的属性，并覆盖之前读取过的同名属性

### 4）常见错误：

- 找不到外部配置文件：Cause: java.io.IOException: Could not find resource dp.properties
  - 在pox.xml中添加扫描到resources下的properties依赖即可

## 5.2 设置（settings）

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。 下表描述了设置中常用的各项设置的含义、默认值等。

| 设置名                   | 描述                                                         | 有效值                                                       | 默认值 |
| :----------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----- |
| mapUnderscoreToCamelCase | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。(数据库不支持区分大小写) | true \| false                                                | False  |
| logImpl                  | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置 |
| cacheEnabled             | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。     | true \| false                                                | true   |
| lazyLoadingEnabled       | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 | true \| false                                                | false  |

```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

## 5.3 类型别名（typeAliases）

- 类型别名可为 Java 类型设置一个缩写名字
- 仅用于 XML 配置，意在降低冗余的全限定类名书写

### 1）给实体类起别名

- 所起的别名可以在Mapper文件中直接使用

```xml
<!-- 可以给实体类起别名 -->
<typeAliases>
    <typeAlias type="com.spzhang.pojo.User" alias="User" />
</typeAliases>
```

### 2）指定包名

- 指定一个包名，**在没有使用注解的情况下**，该包中所有Bean都会使用**其首字母小写的非限定名作为它的别名** 
  - 引用时使用大写也可以，建议小写

```xml
<!-- 指定一个包名，在没有使用注解的情况下，该包中所有Bean都会使用其首字母小写的非限定名作为它的别名 -->
<typeAliases>
    <package name="com.spzhang.pojo"/>
</typeAliases>
```

- 如果**类上有注解，则别名为其注解值**：

```java
@Alias("UserT")
public class User{
    //User别名为UserT
}
```

- 实体类较少时，使用第一种方式。

- 实体类十分多时，建议使用第二种。

## 5.4 环境配置（environments）

- MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中，
- 现实情况下有多种理由需要这么做。例如，开发、测试和生产环境需要有不同的配置；或者想在具有相同 Schema 的多个生产数据库中使用相同的 SQL 映射。还有许多类似的使用场景。
- **不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**
- 所以，如果你想连接两个数据库，就需要创建两个 SqlSessionFactory 实例，每个数据库对应一个。而如果是三个数据库，就需要三个实例，依此类推，记起来很简单：
  - **每个数据库对应一个 SqlSessionFactory 实例**

为了指定创建哪种环境，只要将它作为可选的参数传递给 SqlSessionFactoryBuilder 即可。可以接受环境配置的两个方法签名是：

```java
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment);SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment, properties);
```

如果忽略了环境参数，那么将会加载默认环境，如下所示：

```java
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader);SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, properties);
```

environments 元素定义了如何配置环境。

```xml
<environments default="development">    <environment id="development">        <transactionManager type="JDBC">            <property name="..." value="..."/>        </transactionManager>        <dataSource type="POOLED">            <property name="driver" value="${driver}"/>            <property name="url" value="${url}"/>            <property name="username" value="${username}"/>            <property name="password" value="${password}"/>        </dataSource>    </environment></environments>
```

注意一些关键点:

- 默认使用的环境 ID（比如：default="development"）。
- 每个 environment 元素定义的环境 ID（比如：id="development"）。
- 事务管理器的配置（比如：type="JDBC"）。
- 数据源的配置（比如：type="POOLED"）。

默认环境和环境 ID 顾名思义。 环境可以随意命名，但务必保证默认的环境 ID 要匹配其中一个环境 ID。

### **1）事务管理器（transactionManager）**

在 MyBatis 中有两种类型的事务管理器（也就是 type="[JDBC|MANAGED]"）：

- JDBC – 这个配置直接使用了 JDBC 的提交和回滚设施，它依赖从数据源获得的连接来管理事务作用域。

- MANAGED – 这个配置几乎没做什么。它从不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接。然而一些容器并不希望连接被关闭，因此需要将 closeConnection 属性设置为 false 来阻止默认的关闭行为。例如:

  ```
  <transactionManager type="MANAGED">  <property name="closeConnection" value="false"/></transactionManager>
  ```

**提示** 如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器，因为 Spring 模块会使用自带的管理器来覆盖前面的配置。

这两种事务管理器类型都不需要设置任何属性。它们其实是类型别名，换句话说，你可以用 TransactionFactory 接口实现类的全限定名或类型别名代替它们。

### **2）数据源（dataSource）**

dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源。

- 大多数 MyBatis 应用程序会按示例中的例子来配置数据源。虽然数据源配置是可选的，但如果要启用延迟加载特性，就必须配置数据源。

有三种内建的数据源类型（也就是 type="[UNPOOLED|POOLED|JNDI]"）：

#### **I. UNPOOLED**

 这个数据源的实现会每次请求时打开和关闭连接。虽然有点慢，但对那些数据库连接可用性要求不高的简单应用程序来说，是一个很好的选择。 性能表现则依赖于使用的数据库，对某些数据库来说，使用连接池并不重要，这个配置就很适合这种情形。UNPOOLED 类型的数据源仅仅需要配置以下 5 种属性：

- `driver` – 这是 JDBC 驱动的 Java 类全限定名（并不是 JDBC 驱动中可能包含的数据源类）。
- `url` – 这是数据库的 JDBC URL 地址。
- `username` – 登录数据库的用户名。
- `password` – 登录数据库的密码。
- `defaultTransactionIsolationLevel` – 默认的连接事务隔离级别。
- `defaultNetworkTimeout` – 等待数据库操作完成的默认网络超时时间（单位：毫秒）。查看 `java.sql.Connection#setNetworkTimeout()` 的 API 文档以获取更多信息。

作为可选项，你也可以传递属性给数据库驱动。只需在属性名加上“driver.”前缀即可，例如：

- `driver.encoding=UTF8`

这将通过 DriverManager.getConnection(url, driverProperties) 方法传递值为 `UTF8` 的 `encoding` 属性给数据库驱动。

#### **II. POOLED**

这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这种处理方式很流行，能使并发 Web 应用快速响应请求。

除了上述提到 UNPOOLED 下的属性外，还有更多属性用来配置 POOLED 的数据源：

- `poolMaximumActiveConnections` – 在任意时间可存在的活动（正在使用）连接数量，默认值：10
- `poolMaximumIdleConnections` – 任意时间可能存在的空闲连接数。
- `poolMaximumCheckoutTime` – 在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒）
- `poolTimeToWait` – 这是一个底层设置，如果获取连接花费了相当长的时间，连接池会打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直失败且不打印日志），默认值：20000 毫秒（即 20 秒）。
- `poolMaximumLocalBadConnectionTolerance` – 这是一个关于坏连接容忍度的底层设置， 作用于每一个尝试从缓存池获取连接的线程。 如果这个线程获取到的是一个坏的连接，那么这个数据源允许这个线程尝试重新获取一个新的连接，但是这个重新尝试的次数不应该超过 `poolMaximumIdleConnections` 与 `poolMaximumLocalBadConnectionTolerance` 之和。 默认值：3（新增于 3.4.5）
- `poolPingQuery` – 发送到数据库的侦测查询，用来检验连接是否正常工作并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动出错时返回恰当的错误消息。
- `poolPingEnabled` – 是否启用侦测查询。若开启，需要设置 `poolPingQuery` 属性为一个可执行的 SQL 语句（最好是一个速度非常快的 SQL 语句），默认值：false。
- `poolPingConnectionsNotUsedFor` – 配置 poolPingQuery 的频率。可以被设置为和数据库连接超时时间一样，来避免不必要的侦测，默认值：0（即所有连接每一时刻都被侦测 — 当然仅当 poolPingEnabled 为 true 时适用）。

#### **III. JNDI** 

这个数据源实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的数据源引用。这种数据源配置只需要两个属性：

- `initial_context` – 这个属性用来在 InitialContext 中寻找上下文（即，initialContext.lookup(initial_context)）。这是个可选属性，如果忽略，那么将会直接从 InitialContext 中寻找 data_source 属性。
- `data_source` – 这是引用数据源实例位置的上下文路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。

和其他数据源配置类似，可以通过添加前缀“env.”直接把属性传递给 InitialContext。比如：

- `env.encoding=UTF8`

这就会在 InitialContext 实例化时往它的构造方法传递值为 `UTF8` 的 `encoding` 属性。

```xml
<environments default="development">    <environment id="development">        <transactionManager type="JDBC"/>        <dataSource type="POOLED">            <property name="driver" value="com.mysql.jdbc.Driver"/>            <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>            <property name="username" value="root"/>            <property name="password" value="love..1028"/>        </dataSource>    </environment></environments>
```

## 5.5 其他配置

- [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)
- [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory)
- **plugins（插件）**
  - mybatis-generator-core
  - mybatis-plus
  - 通用mapper

## 5.6 映射器（mappers）

MapperRegistry：注册绑定Mappers映射文件。

- 使用相对于类路径的资源引用

  - **资源路径使用 /**

  ```xml
  <!-- 使用相对于类路径的资源引用 --><mapper resource="com/spzhang/dao/UserMapper.xml" />
  ```

- 完全限定资源定位符（包括 `file:///` 形式的 URL）

  ```xml
  <!-- 使用完全限定资源定位符（URL） --><mappers>  <mapper url="file:///var/mappers/AuthorMapper.xml"/>  <mapper url="file:///var/mappers/BlogMapper.xml"/>  <mapper url="file:///var/mappers/PostMapper.xml"/></mappers>
  ```

- 类名和包名

  - **接口与和它的Mapper配置文件必须同名**
  - **接口和它的Mapper配置文件必须在同一个包下**

```xml
<mappers>    <!-- 通过class文件绑定注册 -->    <mapper class="com.spzhang.dao.UserMapper" /></mappers><!-- 将包内的映射器接口实现全部注册为映射器 --><mappers>  <package name="com.spzhang.dao"/></mappers>
```

这些配置会告诉 MyBatis 去哪里找映射文件

# 6. MyBatis开发入门

## 6.1 MyBatis简介

- **基于Java的持久层框架**（也是一种ORM（Object/Relational Mapping，对象映射）框架）
- 使用**XML或注解**来配置和原始映射
- 消除了Jdbc代码和参数的手工设置

要使用MyBatis需要在maven配置文件中导入相关的依赖：

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.11</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.5</version>
</dependency>
```

## 6.2 MyBatis的工作原理

MyBatis的工作原理如图所示：

<img src="0. Mapper文件标签.assets/image-20200724145825526.png" alt="image-20200724145825526" style="zoom:75%;" />

**MyBatis配置文件**：**mybatis-config.xml**，在这个配置文件中配置MyBatis的运行环境等信息，例如**数据库连接信息、SQL映射文件（可以加载多个）**。

**SQL映射文件**：该文件中**配置了操作数据库的SQL语句。**

## 6.3 MyBatis入门程序

> 目录结构：![image-20200725080132968](0. Mapper文件标签.assets/image-20200725080132968.png)

### 1）添加依赖

在maven中添加Spring核心包、MyBatis包以及MySQL数据库的驱动包。

### 2）创建日志文件

MyBatis默认使用log4j输出日志信息，如果需要查看控制台输出的SQL语句，则需要配置日志文件。

### 3）创建持久化类

创建持久化类，类中声明的属性与数据表的字段一致。

```java
/**
 * springtest数据库中user表的持久化类
 */
public class MyUser {
    private Integer uid;
    private String uname;
    private String usex;

    public Integer getUid() {
        return uid;
    }
    public void setUid(Integer uid) {
        this.uid = uid;
    }

    public String getUname() {
        return uname;
    }

    public void setUname(String uname) {
        this.uname = uname;
    }

    public String getUsex() {
        return usex;
    }

    public void setUsex(String usex) {
        this.usex = usex;
    }
    public String toString(){
        return "" + uid+uname+usex;
    }
}
```



### 4）创建映射文件

需要在resources文件下创建子目录com.mybatis.mapper，在该路径下创建映射文件UserMapper.xml。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.mapper.UserMapper">
    <!-- 根据uid查询一个用户信息 -->
    <select id="selectUserById" parameterType="Integer"
    resultType="com.mybatis.po.MyUser">
        select * from user where uid = #{uid}
    </select>
    <!-- 查询所有用户信息 -->
    <select id="selectAllUser" resultType="com.mybatis.po.MyUser">
        select * from user
    </select>
    <!-- 添加一个用户，#{uname}为com.mybatis.po.MyUser的属性值 -->
    <insert id="addUser" parameterType="com.mybatis.po.MyUser">
        insert into user (uid, uname, usex) values(#{uid}, #{uname}, #{usex})
    </insert>
    <!-- 修改一个用户 -->
    <update id="updateUser" parameterType="com.mybatis.po.MyUser">
        update user set uname = #{uname}, usex = #{usex} where uid = #{uid}
    </update>
    <!-- 删除一个用户 -->
    <delete id="deleteUser" parameterType="Integer">
        delete from user where uid = #{uid}
    </delete>
</mapper>
```



### 5）创建MyBatis的配置文件

在资源路径resources下创建MyBatis的配置文件mybatis-config.xml，在其中配置数据库环境和映射文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 配置环境 -->
    <environments default="development">
        <environment id="development">
            <!-- 使用JDBC的事务管理 -->
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <!-- MySQL数据库驱动 -->
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url"
                          value="jdbc:mysql://localhost:3306/springtest?serverTimezone=Asia/Shanghai" />
                <property name="username" value="root" />
                <property name="password" value="love..1028" />
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!-- 映射文件位置 -->
        <mapper resource="com/mybatis/mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

映射文件和配置文件存放路径如下图：

![image-20200725075543320](0. Mapper文件标签.assets/image-20200725075543320.png)

### 6）创建测试类

MyBatis框架的使用流程如下：

- 读取配置文件
- 构建SqlSessionFactory
- 创建SqlSession
- 执行映射文件中定义的SQL，并返回映射结果
- 提交事务
- 关闭SqlSession

```java
public class MyBatisTest {
    public static void main(String[] args){
        try{
            //读取配置文件mybatis-config.xml
            InputStream config = Resources.getResourceAsStream("mybatis-config.xml");
            //根据配置文件构建SqlSessionFactory
            SqlSessionFactory ssf = new SqlSessionFactoryBuilder().build(config);
            //通过SqlSessionFactory创建SqlSession
            SqlSession ss = ssf.openSession();
            //SqlSession执行映射文件中定义的SQL，并返回映射结果

            //查询一个用户
            MyUser myUser = ss.selectOne("com.mybatis.mapper.UserMapper.selectUserById", 1);
            System.out.println(myUser);
            //添加一个用户
            MyUser addmu = new MyUser();
            addmu.setUid(5);
            addmu.setUname("张");
            addmu.setUsex("男");
            ss.insert("com.mybatis.mapper.UserMapper.addUser", addmu);
            //修改一个用户
            MyUser updatemu = new MyUser();
            updatemu.setUid(5);
            updatemu.setUname("贺");
            updatemu.setUsex("女");
            ss.update("com.mybatis.mapper.UserMapper.updateUser", updatemu);
            //查询所有用户
            List<MyUser> listMu = ss.selectList("com.mybatis.mapper.UserMapper.selectAllUser");
            for(MyUser mu: listMu){
                System.out.println(mu);
            }
            //提交事务
            ss.commit();
            //关闭SqlSession
            ss.close();
        }catch(Exception e){

            e.printStackTrace();
        }
    }
}
```

## 6.4 MyBatis与Spring的整合

直接使用MyBatis框架的SqlSession访问数据库并不简便，本节将其进行整合，使得可以更加方便地访问数据库。

### 1）关键步骤

MyBatis和Spring整合主要包括以下几个关键步骤：

- 导入相关依赖
- 在Spring中配置MyBatis工厂
- 使用Spring管理MyBatis的数据操作

### 2）框架整合示例、

> 项目结构：![image-20200725084322095](0. Mapper文件标签.assets/image-20200725084322095.png)

#### 1. 创建应用并导入相关依赖包

- MyBatis所需依赖包
- Spring框架所需依赖包
- MyBatis和Spring整合的中间依赖包：mybatis-spring
- 数据库驱动依赖包
- 数据源所需的依赖包：commons-dbcp2

其中，MyBatis和Spring整合的中间依赖包以及数据源所需的依赖包导入如下：

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.1</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-dbcp2</artifactId>
    <version>2.5.0</version>
</dependency>
```

此外，需要添加spring-jdbc包来支持事务处理：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.0.8.RELEASE</version>
</dependency>
```



#### 2. 创建持久化类

该部分与6.3中的创建持久化类相同。

#### 3. 创建SQL映射文件和MyBatis核心配置文件

这两个文件创建路径与6.3中相同，其中UserMapper.xml需要将6.3配置文件中mapper元素的namespace属性改为指向要映射的接口（namespace="com.dao.UserDao"），mybatis-config.xml中只需要配置UserMapper.xml的路径。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <mappers>
        <!-- 映射文件位置 -->
        <mapper resource="com/mybatis/mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

#### 4.创建Spring的配置文件

在Spring配置文件中，需要完成的工作有：

- 指定要扫描的包，使注解生效
- 配置数据源：org.apache.commons.dbcp2.BasicDataSource
- 添加事务支持
- 开启事务注解
- 配置MyBatis工厂，同时指定数据源
- Mapper代理开发，使用Spring自动扫描MyBatis的接口并装配
  - 此处需注意，在传递sqlSessionFactoryBeanName时，使用value

```xml
<?xml version="1.0" encoding="UTF-8" ?><beans xmlns="http://www.springframework.org/schema/beans"       xmlns:context="http://www.springframework.org/schema/context"       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"       xmlns:tx="http://www.springframework.org/schema/tx"       xsi:schemaLocation="http://www.springframework.org/schema/beans            http://www.springframework.org/schema/beans/spring-beans.xsd            http://www.springframework.org/schema/context            http://www.springframework.org/schema/context/spring-context.xsd            http://www.springframework.org/schema/tx            http://www.springframework.org/schema/tx/spring-tx.xsd">    <!-- 指定要扫描的包 -->    <context:component-scan base-package="com.dao"/>    <context:component-scan base-package="com.controller"/>    <!-- 配置数据源 -->    <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">        <property name="driverName" value="com.mysql.jdbc.Driver"/>        <property name="url" value="jdbc:mysql://localhost:3306/springtest?serverTimezone=Asia/Shanghai"/>        <property name="username" value="root"/>        <property name="password" value="love..1028"/>        <!-- 最大连接数 -->        <property name="maxTotal" value="30"/>        <!-- 最大空闲连接数 -->        <property name="maxIdle" value="10"/>        <!-- 初始连接数 -->        <property name="initialSize" value="5"/>    </bean>    <!-- 添加事务支持 -->    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">        <property name="dataSource" ref="dataSource"/>    </bean>    <!-- 开启事务注解 -->    <tx:annotation-driven transaction-manager="txManager"/>    <!-- 配置MyBatis工厂，同时指定数据源，并于MyBatis完美整合 -->    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">        <property name="dataSource" ref="dataSource"/>        <!-- configLocation的属性值为MyBatis的核心配置文件 -->        <property name="configLocation" value="mybatis-config.xml"/>    </bean>    <!-- Mapper代理开发，使用Spring自动扫描MyBatis的接口并装配(Spring将指定包中所有被@Mapper注解标注的接口自动装配为MyBatis的映射接口) -->    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">        <!-- Mybatis-spring组建的扫描容器 -->        <property name="basePackage" value="com.dao"/>        <!-- 注意：这里使用的是value -->        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>    </bean></beans>
```

#### 5. 创建数据访问接口

接口使用@Mapper注解为Mapper，其中方法与SQL映射文件一致。

```java
@Repository("userDao")@Mapper/* 使用Spring自动扫描MyBatis的接口并装配（Spring将指定包中所有被@Mapper注解标注的接口自动装配为Mybatis的映射接口） */public interface UserDao {    /**     * 接口方法名对应SQL映射文件UserMapper.xml中的id     */    public MyUser selectUserById(Integer uid);    public List<MyUser> selectAllUser();    public int addUser(MyUser user);    public int updateUser(MyUser user);    public int deleteUser(Integer uid);}
```

#### 6. 创建控制层

创建控制类：UserController，在该类中调用数据访问接口中的方法便可以直接访问数据库：

```java
@Controllerpublic class UserController {    @Autowired    private UserDao userDao;    public void test(){        //查询一个用户        MyUser auser = userDao.selectUserById(1);        System.out.println(auser);        //查询所有用户        List<MyUser> list = userDao.selectAllUser();        for(MyUser myUser: list){            System.out.println(myUser);        }    }}
```

#### 7. 创建测试类

```java
public class TestController {    public static void main(String[] args){        ApplicationContext appContext = new ClassPathXmlApplicationContext("appContext.xml");        UserController userController = (UserController) appContext.getBean("userController");        userController.test();    }}
```

## 6.5 使用MyBatis Generator插件自动生成映射文件

# 7. 生命周期和作用域

练习：

1. 引入数据库配置文件
2. 实体类别名
3. mapper文件引入

不同作用域和生命周期类别是至关重要的，因为错误的使用会导致非常严重的并发问题

## 7.1 SqlSessionFactoryBuilder

- 用来创建SqlSessionFactory对象
- 这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了
- 因此 SqlSessionFactoryBuilder 实例的最佳作用域是**方法作用域**（也就是局部方法变量）
- 你可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但最好还是不要一直保留着它，以保证所有的 XML 解析资源可以被释放给更重要的事情。

## 7.2 SqlSessionFactory

- SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例
- 使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏习惯”
- 因此 SqlSessionFactory 的最佳作用域是**应用作用域**。 有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

## 7.3 SqlSession

- **SqlSession实例不是线程安全的** ，因此是不能被共享的，所以它的最佳的作用域是**请求或方法作用域**
- 如果你现在正在使用一种 Web 框架，考虑将 SqlSession 放在一个和 HTTP 请求相似的作用域中。
- 每次收到的HTTP请求，就可以打开一个SqlSession，返回一个响应，就关闭它
- 关闭操作放到finally代码块中

## 7.4 总结

依赖注入框架可以创建线程安全的、基于事务的 SqlSession 和映射器，并将它们直接注入到你的 bean 中，因此可以直接忽略它们的生命周期。 如果对如何通过依赖注入框架使用 MyBatis 感兴趣，可以研究一下 MyBatis-Spring 或 MyBatis-Guice 两个子项目。

# 6. 映射器

## 6.1 MyBatis配置文件

MyBatis的核心配置文件配置了很多影响MyBatis行为的信息，在与Spring框架整合后，这些信息将配置到Spring的配置文件中。主要包括以下信息：

- 配置数据源

  ```xml
      <!-- 配置数据源 -->
      <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
          <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
          <property name="url" value="jdbc:mysql://localhost:3306/springtest?serverTimezone=Asia/Shanghai"/>
          <property name="username" value="root"/>
          <property name="password" value="love..1028"/>
          <!-- 最大连接数 -->
          <property name="maxTotal" value="30"/>
          <!-- 最大空闲连接数 -->
          <property name="maxIdle" value="10"/>
          <!-- 初始连接数 -->
          <property name="initialSize" value="5"/>
      </bean>
  ```

  

- 配置MyBatis工厂，同时指定数据源

  ```xml
  <!-- 配置MyBatis工厂，同时指定数据源，并于MyBatis完美整合 -->
  <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
      <property name="dataSource" ref="dataSource"/>
      <!-- configLocation的属性值为MyBatis的核心配置文件 -->
      <property name="configLocation" value="mybatis-config.xml"/>
  </bean>
  ```

  

- Mapper代理开发

  ```xml
  <!-- Mapper代理开发，使用Spring自动扫描MyBatis的接口并装配(Spring将指定包中所有被@Mapper注解标注的接口自动装配为MyBatis的映射接口) -->
  <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
      <!-- Mybatis-spring组建的扫描容器 -->
      <property name="basePackage" value="com.dao"/>
      <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
  </bean>
  ```

  MyBatis的核心配置文件中的元素配置顺序不能颠倒，一旦颠倒，在MyBatis启动阶段将发生异常。其模板代码如下：

  ![image-20200803104054077](0. Mapper文件标签.assets/image-20200803104054077.png)

## 7.2 映射器概述

映射器由一个接口加上XML文件（SQL映射文件）组成。MyBatis的映射器也可以使用注解完成，只是使用注解无法满足复杂SQL的需求，其可读性也较差，还丢失了XML上下文相互引用的功能。所以推荐使用XML文件开发映射器。

SQL映射文件的常用配置元素如下：

- select
- insert
- update
- delete
- sql
- resultMap

## 7.3 <select\>元素

在SQL映射文件中，<select\>元素用于映射SQL的select语句。其主要属性有：

- id：它和Mapper的命名空间合起来使用，是唯一标识符，供MyBatis调用。对应接口中方法应该与id保持相同。
- parameterType：传入SQL语句的参数类型的全限定名或别名，是一个可选属性，MyBatis能推断出具体传入语句的参数
- resultType：SQL语句执行后返回的类型。如果是集合类型，返回的是集合元素的类型。

### 1）使用Map接口传递多个参数

1. 传递给映射器的是一个Map对象，使用它在SQL文件中设置对应的参数，对应的SQL文件的代码如下：

   ```xml
       <select id="selectAllUser" parameterType="map" resultType="com.mybatis.po.MyUser">
           select * from user
           where uname like concat('%', #{u_name}, '%')
           and usex = #{u_sex}
       </select>
   ```

   其中，u_name、u_sex为构建map时传入的键值对的键，#{u_name}、#{u_sex}为其对应键值对的值。parameterType设置为map。

2. 修改接口对应的方法声明，为其参数列表添加一个Map参数：

   ```java
       public List<MyUser> selectAllUser(Map map);
   ```

3. 使用时，创建map对象，然后再将其传递给对应的方法：

   ```java
   Map<String, String> map = new HashMap<>();
   map.put("u_name", "");
   map.put("u_sex", "男");
   List<MyUser> list = userDao.selectAllUser(map);
   for(MyUser myUser: list){
       System.out.println(myUser);
   }
   ```

### 2）使用Java Bean传递多个参数

> 参数较多，建议选择此方式

 1. 创建一个POJO类，专门用来传递参数：

    ```java
    package com.mybatis.po;
    
    public class SelectUserParam {
        private String u_name;
        private String u_sex;
    	//getter和setter函数
    }
    ```

    2. 修改映射文件中的对应元素的parameterType，并修改对应接口的方法参数：

    ```xml
        <select id="selectAllUser" parameterType="com.mybatis.po.SelectUserParam" resultType="com.mybatis.po.MyUser">
            select * from user
            where uname like concat('%', #{u_name}, '%')
            and usex = #{u_sex}
        </select>
    ```

    3. 使用时，创建POJO对象，并将其传递个方法：

    ```java
    SelectUserParam userParam = new SelectUserParam();
    userParam.setU_name("");
    userParam.setU_sex("男");
    List<MyUser> list = userDao.selectAllUser(userParam);
    for(MyUser myUser: list){
        System.out.println(myUser);	
    }
    ```

### 7.4 <insert\>元素

# 8. ResultMap结果集映射

## 8.1 字段冲突问题

如果 数据库中user表各字段为：id、name、pwd，而实体类中各属性名为：id、name、password，则对于类型处理器在讲数据库中检索结果映射到实体类型时会出现某些字段缺失的问题。

## 8.2 解决方法：

### 1）方法一：修改mysql语句

```mysql
select id, name, pwd as password
```

### 2）方法二：使用结果集映射\<resultMap>

ResultMap，即结果集映射（将结果集中各字段映射到实体类型中各字段）可以解决冲突问题。

- 只需要映射不同的字段即可

该标签中常用属性有：

- **id**：唯一标识符
- **type**：要映射到的实体类型

该标签的子元素有**\<result>**，其常用属性有：

- **column**：数据库中表的字段名
- **property**：要映射的实体类中的属性（**如果该字段为一个对象，该如何处理？**）

```xml
<resultMap id="UserMap" type="User">
	<result column="pwd" property="password" />
</resultMap>
<select id="getUserById" resultMap="UserMap" parameterType="int">
    select * from mybatis.user where id=#{id}
</select>
```

## 8.3 高级结果映射

MyBatis 创建时的一个思想是：数据库不可能永远是你所想或所需的那个样子。 我们希望每个数据库都具备良好的第三范式或 BCNF 范式，可惜它们并不都是那样。 如果能有一种数据库映射模式，完美适配所有的应用程序，那就太好了，但可惜也没有。 而 ResultMap 就是 MyBatis 对这个问题的答案。

比如，我们如何映射下面这个语句？

![image-20200829085009947](0. Mapper文件标签.assets/image-20200829085009947.png)

其他子标签

# 9. 日志

## 9.1 日志工厂

如果一个数据库操作出现异常，日志就可以帮助排错。

在Mybatis中具体使用什么日志实现，由Mybatis核心配置文件中的setting标签设置。

| 设置名  | 描述                                                  | 有效值                                                       | 默认值 |
| :------ | :---------------------------------------------------- | :----------------------------------------------------------- | :----- |
| logImpl | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。 | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置 |

### 1）STDOUT_LOGGING（标准日志输出）

```xml
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

### 2）LOG4J

什么是Log4j

- Log4j是Apache的一个开放源代码项目，通过使用Log4j，我们可以控制日志信息输送的目的地是控制台、文件、GUI组件、甚至是套接口服务器、NT的事件记录器、UNIX Syslog守护进程等;我们

- 可以控制每一条日志的输出格式
- 通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程
- 可以通过一个配置文件来灵活地进行配置，而不需要修改应用的代码。

- LOG4J使用之前需要在Maven中添加依赖

使用：

1. 在Maven中添加Log4j依赖

   ```xml
   <dependency>
       <groupId>log4j</groupId>
       <artifactId>log4j</artifactId>
       <version>1.2.17</version>
   </dependency>
   ```

2. 在resource资源文件夹下创建log4j.properties文件

   ```properties
   #将等级为DEBUG的日志信息输出到console和file这两个目的地，console和file的定义在下面的代码
   log4j.rootLogger=DEBUG,console,file
   
   #控制台输出的相关设置
   log4j.appender.console = org.apache.log4j.ConsoleAppender
   log4j.appender.console.Target = System.out
   log4j.appender.console.Threshold=DEBUG
   log4j.appender.console.layout = org.apache.log4j.PatternLayout
   log4j.appender.console.layout.ConversionPattern=[%c]-%m%n
   
   #文件输出的相关设置
   log4j.appender.file = org.apache.log4j.RollingFileAppender
   log4j.appender.file.File=./log/kuang.log
   log4j.appender.file.MaxFileSize=10mb
   log4j.appender.file.Threshold=DEBUG
   log4j.appender.file.layout=org.apache.log4j.PatternLayout
   log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n
   
   #日志输出级别
   log4j.logger.org.mybatis=DEBUG
   log4j.logger.java.sql=DEBUG
   log4j.logger.java.sql.Statement=DEBUG
   log4j.logger.java.sql.ResultSet=DEBUG
   log4j.logger.java.sql.PreparedStatement=DEBUG
   ```

3. 配置log4j为日志实现

   ```xml
   <settings>
   	<setting name="logImpl" value="LOG4J" />
   </settings>
   ```

4. log4j的使用

# 10. 分页查询

**为什么要分页？***

- 为了减少数据处理量

## 10.1 limit分页

基本用法：

```mysql
select * from user limit startIndex, pageSize
# 相当于select * from user limit 0, pageSize
select * from user limit pageSize 
```

使用Mybatis实现分页，核心SQL：

1. 接口

   ```java
   public interface UserMapper {
       public List<User> getUserByLimit(Map<String, Object> map);
   }
   ```

2. Mapper.xml

   ```xml
   <resultMap id="UserMap" type="com.spzhang.pojo.User">
   	<result column="pwd" property="password" />
   </resultMap>
   <select id="getUserByLimit" resultMap="UserMap" parameterType="map">
   	select * from mybatis.user limit #{startIndex}, #{pageSize}
   </select>
   ```

3. 测试

   ```java
   @Test
   public void getUserByLimit() {
       SqlSession sqlSession = MybatisUtils.getSqlSession();
   
       UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
       Map<String, Object> map = new HashMap<String, Object>();
       map.put("startIndex", 1);
       map.put("pageSize", 3);
   
       List<User> userList = userMapper.getUserByLimit(map);
       for(User user: userList) {
           System.out.println(user);
       }
   
       sqlSession.close();
   }
   ```

## 10.2 RowBounds分页

不再使用SQL实现分页

1. 接口

   ```java
   public List<User> getUserByRowBouds();
   ```

2. Mapper.xml

   ```xml
   <select id="getUserByLimit" resultMap="UserMap">
       select * from mybatis.user
   </select>
   ```

3. 测试

   ```java
   @Test
   public void getUserByRowBounds() {
       SqlSession sqlSession = MybatisUtils.getSqlSession();
   
       RowBounds rowBounds = new RowBounds(1, 3);
   
       List<User> userList = 		   sqlSession.selectList("com.spzhang.dao.UserMapper.getUserByRowBounds", null, rowBounds);
       for (User user : userList) {
           System.out.println(user);
       }
       sqlSession.close();
   }
   ```

## 10.3 分页插件

**Page Helper**

- https://github.com/pagehelper/Mybatis-PageHelper

# 11. 使用注解开发

对于像 BlogMapper 这样的映射器类来说，还有另一种方法来完成语句映射。 它们映射的语句可以不用 XML 来配置，而可以使用 Java 注解来配置。

使用注解来映射简单语句会使代码显得更加简洁，但对于稍微复杂一点的语句，Java 注解不仅力不从心，还会让你本就复杂的 SQL 语句更加混乱不堪。 因此，如果你需要做一些很复杂的操作，最好用 XML 来映射语句。

选择何种方式来配置映射，以及认为是否应该要统一映射语句定义的形式，完全取决于你和你的团队。 换句话说，永远不要拘泥于一种方式，你可以很轻松的在基于注解和 XML 的语句映射方式间自由移植和切换。

1. 接口：

   ```java
   @Select("select * from user")
   public List<User> getUserList();
   
   //使用多个参数时，必须使用@Param注解
   @Select("select * from user where id=#{id}")
   public User getUserById(@Param("id") int id, @);
   
   @Insert("insert into user(id, name, pwd) values (#{id}, #{name}, #{password})")
   public int addUser(User user);
   @Update("update user set name=#{name}, pwd=#{password} where id=#{id}")
   public int updateUser(User user);
   @Delete("delete from user where id=#{uid}")
   public int deleteUser(@Param("uid") int id);
   ```

2. Mybatis核心文件配置绑定接口：

   ```xml
   <mappers>
       <mapper class="com.spzhang.dao.UserMapper" />
   </mappers>
   ```

本质：反射机制实现！

底层：动态代理！

### @Param()注解

- 基本类型的参数或者String类型，需要加上
- 引用类型不用加
- 如果只有一个基本类型的化，可以忽略（建议都加上）
- 在SQL中引用的就是@Param()中设定的属性名

### #{}和${}的区别

> https://www.cnblogs.com/liaowenhui/p/12217959.html

1）#{}是预编译处理，${}是字符串替换。

2）MyBatis在处理#{}时，会将SQL中的#{}替换为?号，使用PreparedStatement的set方法来赋值；MyBatis在处理 $ { } 时，就是把 ${ } 替换成变量的值。

3）使用 #{} 可以有效的防止SQL注入，提高系统安全性。

# 12. Mybatis执行流程剖析

底层源码分析！

# 13. Lombok

> Project Lombok is a java library that automatically plugs into your editor and build tools, spicing up your java.
> Never write another getter or equals method again, with one annotation your class has a fully featured builder, Automate your logging variables, and much more.

## 13.1 Lombok基本介绍

- Lombok项目是一种自动接通你的编辑器和构建工具的一个Java库
- 不用再一次写额外的getter或者equals方法

缺点：

- 不支持多种参数构造器的重载
- 可读性不强

## 13.2 Lombok准备

1. 在Idea中安装Lombok插件

2. 添加Maven依赖

   ```xml
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
       <version>1.18.12</version>
   </dependency>
   ```

## 13.3 常用注解

### 1）@Data

该注解注解在类上，可以自动生成无参构造、get、set、equals、canEqual、hashCode、toString这些方法。

- @AllArgsConstructor：生成包含所有属性的的构造函数
- @NoArgsConstructor：生成无参构造函数
- @RequiredArgsConstructor： 会生成一个包含常量，和标识了@NotNull的变量的构造方法。生成的构造方法是私有的private。

```java
//同时使用三个注解，则会生成一个无参构造函数、含有所有属性的构造函数
@Data
@AllArgsConstructor
@NoArgsConstructor
```

### 2）@Getter和@Setter

这两个注解用于属性，被其注解的属性会自动生成get或者set方法。

![image-20200828203549220](0. Mapper文件标签.assets/image-20200828203549220.png)

# 14. 多对一的处理

## 14.1 数据库表的创建

- 学生与老师多对1的关系

SQL语句：

```sql
USE `mybatis`;

CREATE TABLE `teacher` (
	`id` INT(10) NOT NULL,
	`name` VARCHAR(30) DEFAULT NULL,
	PRIMARY KEY (`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `teacher` (`id`, `name`) VALUES (1, '张老师');
 
CREATE TABLE `student` (
	`id` INT(10) NOT NULL,
	`name` VARCHAR(30) DEFAULT NULL,
	`tid` INT(10) DEFAULT NULL,
	PRIMARY KEY (`id`),
	KEY `fktid` (`tid`),
	CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `student` (`id`, `name`, `tid`) VALUES (1, '小红', 1);
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (2, '小白', 1);
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (3, '小明', 1);
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (4, '小黑', 1);
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (5, '小绿', 1);
```

## 14.2 测试环境搭建

1. 编写pojo类：Student、Teacher

   ```java
   @Data
   public class Teacher {
       private int id;
       private String name;
   }
   ```

   ```java
   @Data
   public class Student {
       private int id;
       private String name;
   
       //学生需要关联一个老师
       private Teacher teacher;
   }
   ```

2. 编写Mapper接口

   ```java
   public interface TeacherMapper {
       @Select("select * from teacher")
       public List<Teacher> getTeacherList();
   }
   ```

3. 编写Mapper.xml文件(测试环境中直接使用了注解)

4. 编写mybatis-config.xml文件，并注册Mapper.xml

   ```xml
   <mappers>
       <mapper class="com.spzhang.dao.TeacherMapper" />
   </mappers>
   ```

## 14.3 复杂查询

### 1）查询所有的学生信息以及对应的老师信息

复杂的属性单独处理：

- 对象：association
- 集合：collection

#### I. 按照查询嵌套处理

1. 编写Mapper接口

   ```java
   public List<Student> getStudentList();
   ```

2. 编写Mapper.xml文件实现

   1. 定义resultMap处理复杂属性

      - 这里**将column指定的列值作为参数传递给子查询**

      ```xml
      <resultMap id="StudentTeacher" type="Student">
          <!-- 当属性为复杂属性时，如果是对象则用association，如果是集合则使用collection -->
          <association property="teacher" column="tid" javaType="Teacher" select="getTeacher" />
      </resultMap>
      ```

   2. 定义查询语句查询该表

      ```xml
      <select id="getStudentList" resultMap="StudentTeacher">
          select * from student
      </select>
      ```

   3. 定义子查询，根据父查询获得的外键，查询另一张表的信息，并自动封装到对应的pojo对象中

      ```xml
      <select id="getTeacher" resultType="Teacher">    select * from teacher where id=#{id}</select>
      ```

3. 编写测试代码

   ```java
   @Testpublic void getStudenList(){    SqlSession sqlSession = MybatisUtils.getSqlSession();    StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);    List<Student> studentList = studentMapper.getStudentList();    for (Student student : studentList) {        System.out.println(student);    }    sqlSession.close();}
   ```

#### II. 按照结果嵌套处理

- 不需要借助子查询语句
- resultMap中借助association将结果映射到属性对象中。

```xml
<select id="getStudentList" resultMap="StudentTeacher">    select s.id sid, s.name sname, t.id tid, t.name tname    from student s, teacher t where s.tid = t.id</select><resultMap id="StudentTeacher" type="Student">    <result column="sid" property="id" />    <result column="sname" property="name" />    <association property="teacher" javaType="Teacher">        <result column="tid" property="id" />        <result column="tname" property="name" />    </association></resultMap>
```

Mysql多对1查询：

- 子查询
- 联合查询

# 15.一对多处理

## 15.1 修改pojo类

```java
@Data
public class Student {
    private int id;
    private String name;
    private int tid;
}
```

```java
@Data
public class Teacher {
    private int id;
    private String name;
    private List<Student> students;
}
```

## 15.2 编写Mapper接口

```java
public Teacher getTeacherById(@Param("tid") int id);
```

## 15.3 编写Mapper.xml实现接口

### 1）方法一：按照查询嵌套处理

```xml
<select id="getTeacherById" resultMap="TeacherStudent">
    select * from teacher where id=#{tid}
</select>
<resultMap id="TeacherStudent" type="Teacher">
    <result column="id" property="id" />
    <collection property="students" javaType="ArrayList" ofType="Student" column="id" select="getStudentById" />
</resultMap>
<select id="getStudentById" resultType="Student">
    select * from student where id = #{id}
</select>
```

### 2）方法二：按照结果映射查询

```xml
<select id="getTeacherById" resultMap="TeacherStudent">
    select s.id sid, s.name sname, s.tid stid, t.id tid, t.name tname
    from student s, teacher t where tid = #{tid} and s.tid = #{tid}
</select>
<resultMap id="TeacherStudent" type="Teacher">
    <result column="tid" property="id" />
    <result column="tname" property="name" />
    <!-- 复杂的属性需要单独处理 对象：association  集合collection
    javaType指定属性的类型
    集合中的泛型信息，需要使用ofType获取
    -->
    <collection property="students" ofType="Student">
        <result column="sid" property="id" />
        <result column="sname" property="name" />
        <result column="stid" property="tid" />
    </collection>
</resultMap>
```

## 15.4小结：

- 关联：association【多对一】
- 集合：collection【1对多】
- javaType & ofType
  - javaType用来指定实体类中的贤惠能干
  - ofType指定映射到到List或者集合中的pojo类型，泛型中的约束类型

# 16. 动态SQL

所谓的动态SQL，本质还是SQL语句，只是可以在SQL层面，去执行一个逻辑代码。

- 动态SQL其实就是在拼接SQL语句

## 16.1 基本介绍

**什么是动态SQL？根据不同条件生成不同的SQL语句**

动态 SQL 是 MyBatis 的强大特性之一。如果你使用过 JDBC 或其它类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。

使用动态 SQL 并非一件易事，但借助可用于任何 SQL 映射语句中的强大的动态 SQL 语言，MyBatis 显著地提升了这一特性的易用性。

如果你之前用过 JSTL 或任何基于类 XML 语言的文本处理器，你对动态 SQL 元素可能会感觉似曾相识。在 MyBatis 之前的版本中，需要花时间了解大量的元素。借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。

- if
- choose (when, otherwise)
- trim (where, set)
- foreach

## 16.2 搭建环境

1. SQL

   ```sql
   USE `mybatis`;
   
   CREATE TABLE `blog` (
   	`id` VARCHAR(50) NOT NULL COMMENT '博客id',
   	`title` VARCHAR(100) NOT NULL COMMENT '博客标题',
   	`author` VARCHAR(30) NOT NULL COMMENT '博客作者',
   	`create_time` DATETIME NOT NULL COMMENT '创建时间',
   	views INT(30) NOT NULL COMMENT '浏览量'
   )ENGINE=INNODB DEFAULT CHARSET=utf8
   ```

2. 编写配置文件

3. 编写pojo类

   ```java
   @Data
   public class Blog {
       private String id;
       private String title;
       private String author;
       private Date create_time;
       private int views;
       
   }
   ```

4. 编写Mapper接口和Mapper.xml文件

5. 在核心配置文件中注册Mapper.xml

属性名和字段名不一致：数据库中为create_time，属性名为createTime

- 开启驼峰命名映射

```xml
<settings>
    <!-- 开启标准的日志工厂实现 -->
    <setting name="logImple" value="STDOUT_LOGGING"/>
    <!-- 开启驼峰命名映射 -->
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

## 16.3 IF

根据传入的参数决定查询的类型。

### 1）编写Mapper接口及Mapper.xml

Mapper接口：

```java
public List<Blog> queryBlogIF(Map<String, Object> map);
```

Mapper.xml文件：

- test：判断要不要执行\<if>标签的条件

```xml
<select id="queryBlogIF" parameterType="map" resultType="Blog">
    select * from blog where 1=1
    <if test="title != null">
        and title=#{title}
    </if>
    <if test="author != null">
        and author=#{author}
    </if>
</select>
```

### 2）测试

```java
@Test
public void queryBlogIF() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    BlogMapper blogMapper = sqlSession.getMapper(BlogMapper.class);
    Map<String, Object> map = new HashMap<String, Object>();
    List<Blog> blogs = blogMapper.queryBlogIF(map);
    map.put("title", "速学Mybatis");
    map.put("author", "spzhang");
    for (Blog blog : blogs) {
        System.out.println(blogs);
    }
    sqlSession.close();
}
```

## 16.4 choose、when、otherwise

> 有时候，我们不想使用所有的条件，而只是想从多个条件中选择一个使用。针对这种情况，MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句，只会从前到后选择第一个符合条件的子元素中的语句执行

### 1）编写Mapper接口及Mapper.xml文件

Mapper接口：

```java
public List<Blog> queryBlogChoose(Map<String, Object> map);
```

Mapper.xml文件：

```xml
<!-- 动态SQL：choose、when、otherwise -->
<select id="queryBlogChoose" parameterType="map">
    select * from blog
    <choose>
        <when test="title != null">
            title=#{title}
        </when>
        <when test="author != null">
            author =#{author}
        </when>
        <otherwise>
            views = #{views}
        </otherwise>
    </choose>
</select>
```

## 16.5 where

where 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，where元素也会将它们去除。

```xml
<select id="findActiveBlogLike"     resultType="Blog">  SELECT * FROM BLOG  <where>    <if test="state != null">         state = #{state}    </if>    <if test="title != null">        AND title like #{title}    </if>    <if test="author != null and author.name != null">        AND author_name like #{author.name}    </if>  </where></select>
```

## 16.6 trim

如果where元素与你期望的不太一样，你也可以通过自定义 trim 元素来定制 *where* 元素的功能。比如，和 *where* 元素等价的自定义 trim 元素为：

```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">  ...</trim>
```

- prefix：指定要插入内容
- prefixOverrides：会忽略通过管道符分隔的文本序列（注意此例中的空格是必要的）
- 上述例子会移除所有 *prefixOverrides* 属性中指定的内容，并且自动插入WHERE

## 16.7 set

- set用于动态更新语句

- set 元素可以用于动态包含需要更新的列，忽略其它不更新的列

```xml
<update id="updateAuthorIfNecessary">  update Author    <set>      <if test="username != null">username=#{username},</if>      <if test="password != null">password=#{password},</if>      <if test="email != null">email=#{email},</if>      <if test="bio != null">bio=#{bio}</if>    </set>  where id=#{id}</update>
```

这个例子中，*set* 元素会动态地在行首插入 SET 关键字，并会删掉额外的逗号（这些逗号是在使用条件语句给列赋值时引入的）。

来看看与 *set* 元素等价的自定义 *trim* 元素吧：

```xml
<trim prefix="SET" suffixOverrides=",">  ...</trim>
```

注意，我们覆盖了后缀值设置，并且自定义了前缀值。

## 16.8 foreach

动态 SQL 的另一个常见使用场景是对集合进行遍历（尤其是在构建 IN 条件语句的时候）。使用\<foreach>根据传入的集合，自动生成遍历该集合的SQL语句。

foreach元素的属性主要有

- item
- index
- collection：指定要遍历的集合（必须被指定）
- open：开头内容
- close：结尾内容
- separator：分隔符

其中，collection属性必须被指定，并且在不同情况下，其指定值不同：

1. 如果传入的是单参数且参数类型是一个List的时候，collection属性值为list

   ```xml
   <!-- 会根据list中的值生成以(开头，)结尾，or分隔每个元素的语句	如，如果传入的list中有1，2，3，4四个元素，则会生成(item=1 or item=2 or item3 or item=4)--><select id="dynamicForeachTest" resultType="Blog">          select * from t_blog where id in          <foreach collection="list" index="index" item="item"                  open="(" separator="or" close=")">              item=#{item}          </foreach>  </select> 
   ```

   上述collection的值为list，对应的Mapper是这样的 ：

   ```java
   public List<Blog> dynamicForeachTest(List<Integer> ids);  
   ```

2. 如果传入的是单参数且参数类型是一个array数组的时候，collection的属性值为array

   ```xml
   <select id="dynamicForeach2Test" resultType="Blog">                  select * from t_blog where id in          <foreach collection="array" index="index" item="item"                  open="(" separator="," close=")">              #{item}          </foreach>  </select>  
   ```

   上述collection为array，对应的Mapper代码： 

   ```java
   public List<Blog> dynamicForeach2Test(int[] ids);  
   ```

3. 如果传入的参数是多个的时候，我们就需要把它们封装成一个Map了，**当然单参数也可以封装成map**，实际上如果你在传入参数的时候，在[breast](http://www.breast-chest.com/)里面也是会把它封装成一个Map的，map的key就是参数名，所以这个时候collection属性值就是传入的List或array对象在自己封装的map里面的key

   ```xml
   <select id="dynamicForeach3Test" resultType="Blog">          select * from t_blog where title like "%"#{title}"%" and id in          <foreach collection="ids" index="index" item="item"                  open="(" separator="," close=")">              #{item}          </foreach>  </select> 
   ```

   上述collection的值为ids，是传入的参数Map的key，对应的Mapper代码： 

   ```java
   public List<Blog> dynamicForeach3Test(Map<String, Object> params);  
   ```

如果传入的集合中的元素不是普通元素，则可以通过以下方式获取需要的具体的值：

```xml
<select id="dynamicForeachTest" resultType="Blog">          select * from t_blog where id in          <foreach collection="list" index="index" item="item"                  open="(" separator="or" close=")">              item=#{item.id}          </foreach>  </select> 
```



## 16.9 sql

对于一些重复使用的sql片段，可以将其抽取出来，通过\<sql>标签设置为sql语句块，配置其id，在其他元素中可以通过\<include>来引用该sql语句块。

1. 使用\<sql>标签抽取公共部分

   ```xml
   <sql id="if-author">    <if test="title != null">        and title=#{title}    </if>    <if test="author != null">        and author=#{author}    </if></sql>
   ```

2. 在需要使用的地方使用\<include>标签引用

```xml
<!-- 动态SQL：if --><select id="queryBlogIF" parameterType="map" resultType="Blog">    select * from blog where 1=1    <include refid="if-author"/></select>
```

注意：

- 最好基于单表来定义SQL片段
- 不要存在\<where>标签

# 17. 缓存 

## 17.1 简介

什么是缓存？

- 存储在内存中的临时数据
- 将用户经常查询的数据放在缓存（内存）中，用户去查询数据就不用从磁盘上（关系型数据库）查询，从缓存中查询，从而提高查询效率

为什么使用缓存？

- 减少与数据库交互的次数，减少系统开销，提高系统效率

什么样的数据能使用缓存？

- 经常查询且不经常改变的的数据

## 17.2 Mybatis缓存

Mybatis系统中默认定义了两级缓存：一级缓存和二级缓存

- 默认情况下，只有一级缓存开启（SqlSession级别的缓存，也成为本地缓存）
- 二级缓存需要手动开启和配置，是基于namespace



### 1）一级缓存

> 默认开启，只在一次SqlSession中有效，即从拿到SqlSession到关闭SQLSession之间。

一级缓存也叫本地缓存：SqlSession

- 与数据库同一次会话期间查询到的数据会放在本地缓存中
- 以后如果需要获取相同的数据，直接从缓存中拿，不会再去查询数据库

如下图，执行两次相同的查询操作，实际只查询一次：

![image-20200829145501027](0. Mapper文件标签.assets/image-20200829145501027.png)

缓存失效的情况：

- 增删改操作，可能会改变原来的数据，所以必定会清理缓存
- 手动清理缓存

## 17.3 二级缓存

默认情况下，只启用了本地的会话缓存，它仅仅对一个会话中的数据进行缓存。 要启用全局的二级缓存，只需要在你的 SQL 映射文件中添加一行：

```xml
<cache/>
```



基本上就是这样。这个简单语句的效果如下:

- 映射语句文件中的所有 select 语句的结果将会被缓存。
- 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
- 缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
- 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
- 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
- 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。

**提示** ：缓存只作用于 cache 标签所在的映射文件中的语句。如果你混合使用 Java API 和 XML 映射文件，在共用接口中的语句将不会被默认缓存。你需要使用 @CacheNamespaceRef 注解指定缓存作用域。

这些属性可以通过 cache 元素的属性来修改。比如：

```xml
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```

这个更高级的配置创建了一个 FIFO 缓存，每隔 60 秒刷新，最多可以存储结果对象或列表的 512 个引用，而且返回的对象被认为是只读的，因此对它们进行修改可能会在不同线程中的调用者产生冲突。

可用的清除策略有：

- `LRU` – 最近最少使用：移除最长时间不被使用的对象。
- `FIFO` – 先进先出：按对象进入缓存的顺序来移除它们。
- `SOFT` – 软引用：基于垃圾回收器状态和软引用规则移除对象。
- `WEAK` – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。

默认的清除策略是 LRU。

flushInterval（刷新间隔）属性可以被设置为任意的正整数，设置的值应该是一个以毫秒为单位的合理时间量。 默认情况是不设置，也就是没有刷新间隔，缓存仅仅会在调用语句时刷新。

size（引用数目）属性可以被设置为任意正整数，要注意欲缓存对象的大小和运行环境中可用的内存资源。默认值是 1024。

readOnly（只读）属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓存对象的相同实例。 因此这些对象不能被修改。这就提供了可观的性能提升。而可读写的缓存会（通过序列化）返回缓存对象的拷贝。 速度上会慢一些，但是更安全，因此默认值是 false。

**提示** 二级缓存是事务性的。这意味着，当 SqlSession 完成并提交时，或是完成并回滚，但没有执行 flushCache=true 的 insert/delete/update 语句时，缓存会获得更新。

## 17.4 使用自定义缓存

除了上述自定义缓存的方式，你也可以通过实现你自己的缓存，或为其他第三方缓存方案创建适配器，来完全覆盖缓存行为。

```xml
<cache type="com.domain.something.MyCustomCache"/>
```

这个示例展示了如何使用一个自定义的缓存实现。type 属性指定的类必须实现 org.apache.ibatis.cache.Cache 接口，且提供一个接受 String 参数作为 id 的构造器。 这个接口是 MyBatis 框架中许多复杂的接口之一，但是行为却非常简单。

```java
public interface Cache {
  String getId();
  int getSize();
  void putObject(Object key, Object value);
  Object getObject(Object key);
  boolean hasKey(Object key);
  Object removeObject(Object key);
  void clear();
}
```

为了对你的缓存进行配置，只需要简单地在你的缓存实现中添加公有的 JavaBean 属性，然后通过 cache 元素传递属性值，例如，下面的例子将在你的缓存实现上调用一个名为 `setCacheFile(String file)` 的方法：

```xml
<cache type="com.domain.something.MyCustomCache">
  <property name="cacheFile" value="/tmp/my-custom-cache.tmp"/>
</cache>
```

你可以使用所有简单类型作为 JavaBean 属性的类型，MyBatis 会进行转换。 你也可以使用占位符（如 `${cache.file}`），以便替换成在[配置文件属性](https://mybatis.org/mybatis-3/zh/configuration.html#properties)中定义的值。

从版本 3.4.2 开始，MyBatis 已经支持在所有属性设置完毕之后，调用一个初始化方法。 如果想要使用这个特性，请在你的自定义缓存类里实现 `org.apache.ibatis.builder.InitializingObject` 接口。

```java
public interface InitializingObject {
  void initialize() throws Exception;
}
```

**提示** 上一节中对缓存的配置（如清除策略、可读或可读写等），不能应用于自定义缓存。

请注意，缓存的配置和缓存实例会被绑定到 SQL 映射文件的命名空间中。 因此，同一命名空间中的所有语句和缓存将通过命名空间绑定在一起。 每条语句可以自定义与缓存交互的方式，或将它们完全排除于缓存之外，这可以通过在每条语句上使用两个简单属性来达成。 默认情况下，语句会这样来配置：

```xml
<select ... flushCache="false" useCache="true"/>
<insert ... flushCache="true"/>
<update ... flushCache="true"/>
<delete ... flushCache="true"/>
```

鉴于这是默认行为，显然你永远不应该以这样的方式显式配置一条语句。但如果你想改变默认的行为，只需要设置 flushCache 和 useCache 属性。比如，某些情况下你可能希望特定 select 语句的结果排除于缓存之外，或希望一条 select 语句清空缓存。类似地，你可能希望某些 update 语句执行时不要刷新缓存。

#### cache-ref

回想一下上一节的内容，对某一命名空间的语句，只会使用该命名空间的缓存进行缓存或刷新。 但你可能会想要在多个命名空间中共享相同的缓存配置和实例。要实现这种需求，你可以使用 cache-ref 元素来引用另一个缓存。

```
<cache-ref namespace="com.someone.application.data.SomeMapper"/>
```






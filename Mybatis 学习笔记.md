# Mybatis 学习笔记

###### mybatis版本：3.5.7

java版本  jdk1.8

数据库 oracle

## 前言：

​	因为公司是传统公司，技术栈比较老，大多是用的Hibernate 和 JPA ，偶尔自己开发一个webservices或者适配器什么的才会用到Mybatis

现在重新学习记录一下，此笔记是目前学习几天的笔记，后续会根据学习进度持续更新



ps:第一次写笔记

再ps： 新买了一块显示器 很开心

<img src="C:\Users\迪迦奥特曼\Desktop\958d740b35e0c79ab10803a26e9f172.jpg" alt="958d740b35e0c79ab10803a26e9f172" style="zoom:25%;" />



##  Mybatis基础学习篇：

### 1.创建maven工程

​		项目结构  这是从Demo中随便截图的一个 Mybatis的大致项目结构如图所示，后续笔记中提到的文件名称可能与本图不一致，是因为进度不同Demo不同，但无伤大雅，本图只是展示Mybatis的项目结构，文件名称不同，但是结构是一样的



<img src="C:\Users\迪迦奥特曼\AppData\Roaming\Typora\typora-user-images\image-20220313115252557.png" alt="image-20220313115252557" style="zoom: 80%;" />



1.idea中新建maven项目

​		<img src="C:\Users\迪迦奥特曼\AppData\Roaming\Typora\typora-user-images\image-20220313030550548.png" alt="image-20220313030550548" style="zoom:50%;" />

2.引入依赖

```java
 <dependencies>
    <!-- Mybatis核心 -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.7</version>
    </dependency>
    <!-- junit测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>

    <!--oracle-->
    <dependency>
        <groupId>com.oracle</groupId>
        <artifactId>ojdbc6</artifactId>
        <version>11.2.0.4.0</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>
```

3.引入Mybatis核心配置文件mybatis-config.xml

```java
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="jdbc.properties"/>
    <settings>
        <!--开启驼峰命名-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!--开启延时加载-->

<!--
        <setting name="lazyLoadingEnabled" value="true"/>
-->

    </settings>
    <typeAliases>
        <package name="com.jialuyang.mybatis.pojo"/>
    </typeAliases>
    <!--设置连接数据库的环境-->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <!--引入映射文件-->
    <mappers>
        <!--
		<mapper resource="mappers/UserMapper.xml"/>
-->
        <package name="com.jialuyang.mybatis.mapper"/>
    </mappers>
</configuration>
```

**environments** :这个标签是用来配置多个数据源的

​	**属性 default** ：默认使用的数据源id

**environment**：设置数据源信息

​	**属性 id** ：设置环境的的唯一标识，可以通过environments标签中的default设置某一个环境的id， 表示默认使用的环境 

**transactionManager**：设置事务的管理方式

​	**属性type**：type="JDBC|MANAGED" ，当type为jdbc的时候，当前环境的事务管理都必须手动处理

​	type为managed的时候，事务会被管理

**dataSource**：设置数据源信息

​	**属性 type**：设置数据源的类型

​					**取值**：type="POOLED"：使用的是数据库连接池，就是讲创建的连接进行缓存，下次使用的时候直接从缓存里面拿出来，

​																不需要重新创建

​								type="UNPOOLED"：不用连接池，每次使用连接都是创建一个新的

​								type="JNDI"：调用上下文中的数据源 

**dataSource的子标签**：

```java
<!--设置驱动类的全类名--> 
<property name="driver" value="${jdbc.driver}"/> 
<!--设置连接数据库的连接地址--> 
<property name="url" value="${jdbc.url}"/>
<!--设置连接数据库的用户名-->
<property name="username" value="${jdbc.username}"/>
<!--设置连接数据库的密码-->
<property name="password" value="${jdbc.password}"/>
```

value里面可以直接写值，如果用${}这种写法，需要引入properties文件，如我上面配置中的

```java
<properties resource="jdbc.properties"></properties>
```

项目中的文件目录为

![image-20220313033938517](C:\Users\迪迦奥特曼\AppData\Roaming\Typora\typora-user-images\image-20220313033938517.png)

properties文件内容为

```java
jdbc.driver=oracle.jdbc.OracleDriver   //这是oracle驱动
jdbc.url=jdbc:oracle:thin:@127.0.0.1:1521/TEST
jdbc.username=TEST
jdbc.password=TEST
```

**mappers**：用来引入mapper映射文件的标签

​	官网中给出的配置是

```java
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
```

​	引入的单个文件，如果你项目中有多个文件，就需要在mappers中写多个mapper标签，挨个引入，这样很麻烦，所以我在项目中以包为单位引入

```java
   <mappers>
        <package name="com.jialuyang.mybatis.mapper"/>
    </mappers>
```

​	需要注意的是如果是以包为单位引入，你创建的mapper接口和mapper映射文件的包路径，必须相同

![image-20220313034641068](C:\Users\迪迦奥特曼\AppData\Roaming\Typora\typora-user-images\image-20220313034641068.png)

上面的配置文件里面 还有两个标签typeAliases和settings 里面的东西后面写Demo的时候用到再说。

最后因为方便查看，引入了log4j.xml

```java
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
	<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">
	    <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender">
	        <param name="Encoding" value="UTF-8" />
	        <layout class="org.apache.log4j.PatternLayout">
				<param name="ConversionPattern" value="%-5p %d{MM-dd HH:mm:ss,SSS} %m (%F:%L) \n" />
	        </layout>
	    </appender>
	    <logger name="java.sql">
	        <level value="debug" />
	    </logger>
	    <logger name="org.apache.ibatis">
	        <level value="info" />
	    </logger>
	    <root>
	        <level value="debug" />
	        <appender-ref ref="STDOUT" />
	    </root>
	</log4j:configuration>
```

log4j这个东西虽有爆出了很多的漏洞，但是因为本地写Demo用，所以就无所谓了。

### 2.创建Mapper接口

#### 	2.1添加库表

​			首先先在数据库新建用户表,注意 我的数据库是oracle，mysql不要直接复制粘贴

```sql
create table T_USER
(
  id       INTEGER not null,   //用户id
  username VARCHAR2(20),	   //用户名
  password VARCHAR2(20),       //密码
  age      INTEGER,            //年龄
  sex      CHAR(1),            //性别
  email    VARCHAR2(20)        //邮箱
)
```



#### 	2.2创建数据库对应实体类

```java
public class User {

    private Integer id;

    private String username;

    private String password;

    private Integer age;

    private String sex;

    private String email;

    public User() {
    }

    public User(Integer id, String username, String password, Integer age, String sex, String email) {
        this.id = id;
        this.username = username;
        this.password = password;
        this.age = age;
        this.sex = sex;
        this.email = email;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", age=" + age +
                ", sex='" + sex + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
}
```

实体类和数据库表的对应关系

| java           | 数据库                    |
| -------------- | ------------------------- |
| 类             | 数据库的表                |
| 类的属性       | 表的字段也就是表的列      |
| 生成的java对象 | 表里面的一行数据/一行记录 |

#### 2.3创建mapper接口

> Mybatis中的mapper接口相当于javaweb的dao层，dao层需要写一个接口，然后写一个实现类，mapper和dao的区别是mapper仅仅是一个接口，不需要写实现类，需要运行的sql在mapper的映射文件中去编写

```java
public interface UserMapper {

    /**
     *	
     * 表--实体类--mapper接口--映射文件
     */

    /**
     * 创建一个添加用户信息的方法
     */
    int insertUser();

}

```



#### 2.4 创建MyBatis的映射文件

> 映射文件的命名规则：mapper接口的名称.xml
>
> 例如：表t_user,映射的实体类对象是User，mapper接口的名称是UserMapper.java ,映射文件的的名称为UserMapper.xml
>
> Mybatis映射文件用于编写sql，访问一级操作表中的数据
>
> Mybatis映射文件存放的位置在src/main/resources目录下
>
> **MyBatis可以面向接口操作数据，但是要保持两个一致**
>
> ​		**1.mapper接口中的mapper标签里面的属性 namespace（命名空间）填写的是你对应的mapper接口的全类名也就是包名.类名，映射文件通过namespace与mapper做绑定**
>
> ​	**2.映射文件中sql标签的id属性，与mapper接口中所对应的方法名一致**
>
> ​		**例如我上面创建mapper接口名称是UserMapper ，他所对应的映射文件UserMapper.xml文件中，namespace的值就是UserMapper接口的全类名（包名.类名） sql标签也就是下面的<insert>标签的id属性值，就是mapper接口中，它对应方法的方法名称**
>
> **总结来说就是 namespace 绑定接口  sql标签的id属性绑定方法 **
>
> 

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.jialuyang.mybatis.mapper.UserMapper">

    <!--int insertUser();-->
    <insert id="insertUser">
        insert into t_user values(1,'admin','123456',23,'a','12345@qq.com')
    </insert>

</mapper>
```

#### 2.5测试功能

```java
@Test
public void testMyBatis() throws IOException {
    //加载核心配置文件
    InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
    //获取SqlSessionFactoryBuilder
    SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
    //获取sqlSessionFactory
    SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
    //获取SqlSession
    //注：如果sqlSessionFactory.openSession(true)中不传参true，就得手动提交事务sqlSession.commit();
    //传参true以后，sqlsession对象操作的sql都会自动提交
    SqlSession sqlSession = sqlSessionFactory.openSession(true);
    //获取mapper接口对象
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    //测试功能
    int result = mapper.insertUser();
    //提交事务
    //sqlSession.commit();
    System.out.println("result:"+result);
}
```

> SqlSession:代表java程序和数据库之间的会话，学过javaweb以后应该知道还有一个会话，是httpSession，他是java程序和浏览器直接的会话
>
> SqlSessionFactory是生产SqlSession的工厂
>
> ​	这里Mybaits用到了工厂模式，如果创建一个对象，使用过程基本固定，那么我们可以把创建的对象的相关代码封装到一个工厂类中，以后都使用这个工厂类来生产我们需要的对象

### 3.实现MyBaits简单的增删查改

> MyBatis的映射文件中有多个sql标签对应sql操作中的增删查改
>
> ![image-20220313142658331](C:\Users\迪迦奥特曼\AppData\Roaming\Typora\typora-user-images\image-20220313142658331.png)

具体的实现方法与上述demo中大致一样

##### 	1>.Mapper接口中新增方法

```java
/**
 * 添加用户信息
 */
int insertUser();

/**
 * 修改用户信息
 */
void updateUser();

/**
 * 删除用户信息
 */
void deleteUser();

/**
 * 根据id查询用户信息
 */
User getUserById();


```

##### 	2>.映射文件中添加对应的方法的sql配置

```xml
    <update id="updateUser">
        update t_user set username = '张三' where id = 4
    </update>
	<delete id="deleteUser">
        delete from t_user where id = 5
    </delete>
    <select id="getUserById" resultType="com.jialuyang.mybatis.pojo.User">
        select * from t_user where id = 3
    </select>
```

​		updateUser();deleteUser(); 这两个方法也可以返回int类型或者Integer类型，当返回为int或者Integer类型的时候，返回值是数据库操作对应sql后受影响的行数，比如你修改了三行数据，返回就是3，删除了1行数据，返回就是1

##### 	3>.select标签中的返回值设置

​		**注意：select标签中 ，是必须设置返回值的，也就是实体类和数据库表的映射关系，属性是resultType或者resultMap，这两个只能二选一**

​		**resultType：自动映射，当属性名称和表中的字段名一致的时候 可以用，我上述的Demo中，表和实体类对象一致，所以用了resultType，resultType的填写有两种，一种是全类名，就是包名.类名，如我上述Demo，还有一种是用别名的方式**
​		**resultType别名填写方式：首先需要了解一个标签typeAliases，这是全局配置文件中的一个，我上述的配置中也同样配置了此标签**

```xml
    <typeAliases>
        <package name="com.jialuyang.mybatis.pojo"/>
    </typeAliases>
```

​		**引用官网中的解释**

> ### 类型别名（typeAliases）
>
> 类型别名可为 Java 类型设置一个缩写名字。 它仅用于 XML 配置，意在降低冗余的全限定类名书写。例如：
>
> ```
> <typeAliases>
>   <typeAlias alias="Author" type="domain.blog.Author"/>
>   <typeAlias alias="Blog" type="domain.blog.Blog"/>
>   <typeAlias alias="Comment" type="domain.blog.Comment"/>
>   <typeAlias alias="Post" type="domain.blog.Post"/>
>   <typeAlias alias="Section" type="domain.blog.Section"/>
>   <typeAlias alias="Tag" type="domain.blog.Tag"/>
> </typeAliases>
> ```
>
> 当这样配置时，`Blog` 可以用在任何使用 `domain.blog.Blog` 的地方。
>
> 也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，比如：
>
> ```
> <typeAliases>
>   <package name="domain.blog"/>
> </typeAliases>
> ```
>
> 每一个在包 `domain.blog` 中的 Java Bean，在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。 比如 `domain.blog.Author` 的别名为 `author`；若有注解，则别名为其注解值。见下面的例子：
>
> ```
> @Alias("author")
> public class Author {
>     ...
> }
> ```
>
> 下面是一些为常见的 Java 类型内建的类型别名。它们都是不区分大小写的，注意，为了应对原始类型的命名重复，采取了特殊的命名风格。
>
> | 别名       | 映射的类型 |
> | :--------- | :--------- |
> | _byte      | byte       |
> | _long      | long       |
> | _short     | short      |
> | _int       | int        |
> | _integer   | int        |
> | _double    | double     |
> | _float     | float      |
> | _boolean   | boolean    |
> | string     | String     |
> | byte       | Byte       |
> | long       | Long       |
> | short      | Short      |
> | int        | Integer    |
> | integer    | Integer    |
> | double     | Double     |
> | float      | Float      |
> | boolean    | Boolean    |
> | date       | Date       |
> | decimal    | BigDecimal |
> | bigdecimal | BigDecimal |
> | object     | Object     |
> | map        | Map        |
> | hashmap    | HashMap    |
> | list       | List       |
> | arraylist  | ArrayList  |
> | collection | Collection |
> | iterator   | Iterator   |

​			

**我这里用到的就是指定包起别名，当你设置好typeAliases以后，你的sql映射就可以写为**

```xml
   <select id="getUserById" resultType="User/user"> <!--不区分大小写-->
        select * from t_user where id = 3
    </select>
```

**但是我不喜欢这么写，因为可读性太差了，没有写全类名看着直接了当，当然后续的Demo也用了这种写法，因为实体类少，写着省劲**				

**resultMap：自定义映射，用于一对多或多对一或字段名和属性名不一致的情况，resultMap的具体用法，后续的笔记里面会写出来**

### 4.MyBatis获取值的两种方式

**MyBatis获取参数值的两种方式#{}，${} **

**${}的本是就是字符串拼接，#{}的本质就是占位符赋值**

${}使用字符串拼接的方式拼接sql，若为字符串类型或日期类型的字段进行赋值时，需要手动加单引

号；但是#{}使用占位符赋值的方式拼接sql，此时为字符串类型或日期类型的字段进行赋值时，可以自

动添加单引号

#### 1.单个字面量类型的参数

> 若mapper接口中的方法参数为单个的字面量类型
>
> 此时可以使用${}和#{}以任意的名称获取参数的值，注意${}需要手动加单引号

```java
    /**
     * 根据用户名查询用户信息
     */
    User getUserByUsername(String username);
```

```xml
    <!--User getUserByUsername(String username);-->
    <select id="getUserByUsername" resultType="User">
        <!--select * from t_user where username = #{username}-->
        select * from t_user where username = '${username}'
    </select>
```

#### 2.多个字面量类型的参数

> 若mapper接口中的方法参数为多个时
>
> 此时MyBatis会自动将这些参数放在一个map集合中，以arg0,arg1...为键，以参数为值；以
>
> param1,param2...为键，以参数为值；因此只需要通过${}和#{}访问map集合的键就可以获取相对应的
>
> 值，注意${}需要手动加单引号

```java
    /**
     * 验证登录
     */
    User checkLogin(String username, String password);
```

```xml
<!--User checkLogin(String username, String password);-->
<select id="checkLogin" resultType="User">
    <!--select * from t_user where username = #{arg0} and password = #{arg1}-->
    select * from t_user where username = '${param1}' and password = '${param2}'
</select>
```

#### 3.map集合类型的参数

> 若mapper接口中的方法需要的参数为多个时，此时可以手动创建map集合，将这些数据放在map中
>
> 只需要通过${}和#{}访问map集合的键就可以获取相对应的值，注意${}需要手动加单引号

```java
    /**
     * 验证登录（参数为map）
     */
    User checkLoginByMap(Map<String, Object> map);
```

```xml
    <!--User checkLoginByMap(Map<String, Object> map);-->
    <select id="checkLoginByMap" resultType="User">
        select * from t_user where username = #{username} and password = #{password}
    </select>
```

```java
    @Test
    public void testCheckLoginByMap(){
        SqlSession sqlSession = SqlSessionUtils.getSqlSession();
        ParameterMapper mapper = sqlSession.getMapper(ParameterMapper.class);
        Map<String, Object> map = new HashMap<>();
        map.put("username", "admin");
        map.put("password", "123456");
        User user = mapper.checkLoginByMap(map);
        System.out.println(user);
    }
```

#### 4.实体类类型的参数

>若mapper接口中的方法参数为实体类对象时
>
>此时可以使用${}和#{}，通过访问实体类对象中的属性名获取属性值，注意${}需要手动加单引号

```java
    /**
     * 添加用户信息
     */
    int insertUser(User user);
```

```xml
    <!--int insertUser(User user);-->
    <insert id="insertUser">
        insert into t_user values(null,#{username},#{password},#{age},#{sex},#{email})
    </insert>
```

#### 5.使用@Param注解

>可以通过@Param注解标识mapper接口中的方法参数
>
>此时，会将这些参数放在map集合中，以@Param注解的value属性值为键，以参数为值；以
>
>param1,param2...为键，以参数为值；只需要通过${}和#{}访问map集合的键就可以获取相对应的值，
>
>注意${}需要手动加单引号

```java
 /**
     * 验证登录（使用@Param）
     */
    User checkLoginByParam(@Param("username") String username, @Param("password") String password);
```

```xml
    <!--User checkLoginByParam(@Param("username") String username, @Param("password") String password);-->
    <select id="checkLoginByParam" resultType="User">
        select * from t_user where username = #{username} and password = #{password}
    </select>
```

### 5.MyBatis的各种查询功能

#### 1.查询一个实体类对象

```java
    /**
     * 根据id查询用户信息
     */
    List<User> getUserById(@Param("id") Integer id);
```

```xml
    <!--User getUserById(@Param("id") Integer id);-->
    <select id="getUserById" resultType="User">
        select * from t_user where id = #{id}
    </select>
```

#### 2.查询一个List集合

```java
    /**
     * 查询所有的用户信息
     */
    List<User> getAllUser();
```

```xml
    <!--List<User> getAllUser();-->
    <select id="getAllUser" resultType="User">
        select * from t_user
    </select>
```

#### 3.查询单个数据

```java
    /**
     * 查询用户信息的总记录数
     */
    Integer getCount();
```

```xml
    <!--Integer getCount();-->
    <select id="getCount" resultType="_int">
        select count(*) from t_user
    </select>
```

#### 4.查询一条数据为map集合

```java
    /**
     * 根据id查询用户信息为一个map集合
     */
    Map<String, Object> getUserByIdToMap(@Param("id") Integer id);
```

```xml
    <!--Map<String, Object> getUserByIdToMap(@Param("id") Integer id);-->
    <select id="getUserByIdToMap" resultType="map">
        select * from t_user where id = #{id}
    </select>
```

```
<!--结果：{password=123456, sex=男, id=1, age=23, username=admin}-->
```

#### 5.查询多条数据为map集合

##### 方式一：

> 将表中的数据以map集合的方式查询，一条数据对应一个map，若返回多个数据，就会产生多个map集合，此时可以将这些map放在一个List集合里面

```
    /**
     * 查询所有用户信息为map集合
     */
    List<Map<String, Object>> getAllUserToMap();
```

```xml
    <select id="getAllUserToMap" resultType="map">
        select * from t_user
    </select>
```

##### 方式二：

> 使用@Mapkey注解
>
> 将表中的数据以map集合的方式查询，一条数据对应一个map；若有多条数据，就会产生多个map集合，并 且最终要以一个map的方式返回数据，此时需要通过@MapKey注解设置map集合的键，值是每条数据所对应的 map集合 

```java
    /**
     * 查询所有用户信息为map集合
     * 此时将id作为返回map的key
     */
    //List<Map<String, Object>> getAllUserToMap();
    @MapKey("id")
    Map<String, Object> getAllUserToMap();
```

```xml
    <!--Map<String, Object> getAllUserToMap();-->
    <select id="getAllUserToMap" resultType="map">
        select * from t_user
    </select>

```

```结果： 
```

<!-- 

{ 

1={password=123456, sex=男, id=1, age=23, username=admin}, 

2={password=123456, sex=男, id=2, age=23, username=张三}, 

3={password=123456, sex=男, id=3, age=23, username=张三} 

}

### 6.特殊sql的执行

#### 1.模糊查询

```java
    /**
     * 根据用户名模糊查询用户信息
     */
    List<User> getUserByLike(@Param("username") String username);
```

```xml
<!--我用的数据库是oracle 所以语法也是oracle的  -->
<select id="getUserByLike" resultType="user">
     <!--   select  * from t_user where username like '%${username}%'-->
       <!-- select  * from t_user where username like concat(concat('%',#{username}),'%')-->
       select  * from t_user where username like '%' ||#{username} || '%'
    </select>
```

```xml
<!-- mysql语法-->  
<!--List<User> getUserByLike(@Param("username") String username);-->
    <select id="getUserByLike" resultType="User">
        <!--select * from t_user where username like '%${username}%'-->
        <!--select * from t_user where username like concat('%',#{username},'%')-->
        select * from t_user where username like "%"#{username}"%"
    </select>
```

#### 2.批量删除

```java
    /**
     * 批量删除
     */
    int deleteMore(@Param("ids") String ids);
```

```xml
    <!--int deleteMore(@Param("ids") String ids);-->
    <delete id="deleteMore">
        delete from t_user where id in (${ids})
    </delete>

```

```java
    @Test
    public void testDeleteMore(){
        SqlSession sqlSession = SqlSessionUtils.getSqlSession();
        SQLMapper mapper = sqlSession.getMapper(SQLMapper.class);
        int result = mapper.deleteMore("1,2,3");
        System.out.println(result);
    }
```

#### 3.动态设置表名

```java
    /**
     * 查询指定表中的数据
     */
    List<User> getUserByTableName(@Param("tableName") String tableName);
```

```xml
    <!--List<User> getUserByTableName(@Param("tableName") String tableName);-->
    <select id="getUserByTableName" resultType="User">
        select * from ${tableName}
    </select>
```

#### 4.获取自增的主键

> **下面是mysql的写法，oracle中没有自增主键，oralce用的序列**

```java
    /**
     * 添加用户信息
     */
    void insertUser(User user);
```

```xml
    <!--
        void insertUser(User user);
        useGeneratedKeys:设置当前标签中的sql使用了自增的主键
        keyProperty:将自增的主键的值赋值给传输到映射文件中参数的某个属性
    -->
    <insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
        insert into t_user values(null,#{username},#{password},#{age},#{sex},#{email})
    </insert>
```

> oralce序列写法

```java
<insert id="addEmp" >
		<!-- 
		keyProperty:查出的主键值封装给javaBean的哪个属性
		order="BEFORE":当前sql在插入sql之前运行
			   AFTER：当前sql在插入sql之后运行
		resultType:查出的数据的返回值类型
		
		BEFORE运行顺序：
			先运行selectKey查询id的sql；查出id值封装给javaBean的id属性
			在运行插入的sql；就可以取出id属性对应的值
		AFTER运行顺序：
			先运行插入的sql（从序列中取出新值作为id）；
			再运行selectKey查询id的sql；
		 -->
		<selectKey keyProperty="id" order="BEFORE" resultType="Integer">
			<!-- 编写查询主键的sql语句 -->
			<!-- BEFORE-->
			select EMPLOYEES_SEQ.nextval from dual 
			<!-- AFTER：
			 select EMPLOYEES_SEQ.currval from dual -->
		</selectKey>
		
		<!-- 插入时的主键是从序列中拿到的 -->
		<!-- BEFORE:-->
		insert into employees(EMPLOYEE_ID,LAST_NAME,EMAIL) 
		values(#{id},#{lastName},#{email<!-- ,jdbcType=NULL -->}) 
		<!-- AFTER：
		insert into employees(EMPLOYEE_ID,LAST_NAME,EMAIL) 
		values(employees_seq.nextval,#{lastName},#{email}) -->
	</insert>
```

### 7.自定义映射resultMap

> 新增两个表 t_emp 员工表   t_dept  部门表

```sql
create table T_DEPT  --部门表
(
  did       INTEGER not null, --部门id
  dept_name VARCHAR2(20)      --部门名称
)
```

```sql
create table T_EMP          --员工表
(
  eid      INTEGER not null,	--员工id
  emp_name VARCHAR2(20),		--员工姓名
  age      INTEGER,				--年龄
  sex      CHAR(1),				--性别
  email    VARCHAR2(20),		--邮箱
  did      INTEGER				--部门id
)
```

> 创建对应实体类

```java
public class Emp {

    private Integer eid;

    private String empName;

    private Integer age;

    private String sex;

    private String email;

    

    @Override
    public String toString() {
        return "Emp{" +
                "eid=" + eid +
                ", empName='" + empName + '\'' +
                ", age=" + age +
                ", sex='" + sex + '\'' +
                ", email='" + email + '\'' +
                '}';
    }


    public Integer getEid() {
        return eid;
    }

    public void setEid(Integer eid) {
        this.eid = eid;
    }

    public String getEmpName() {
        return empName;
    }

    public void setEmpName(String empName) {
        this.empName = empName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Emp() {
    }

    public Emp(Integer eid, String empName, Integer age, String sex, String email) {
        this.eid = eid;
        this.empName = empName;
        this.age = age;
        this.sex = sex;
        this.email = email;
    }
}
```

```java
public class Dept {

    private Integer did;

    private String deptName;

   

    @Override
    public String toString() {
        return "Dept{" +
                "did=" + did +
                ", deptName='" + deptName + '\'' +
                '}';
    }


    public Integer getDid() {
        return did;
    }

    public void setDid(Integer did) {
        this.did = did;
    }

    public String getDeptName() {
        return deptName;
    }

    public void setDeptName(String deptName) {
        this.deptName = deptName;
    }

    public Dept() {
    }

    public Dept(Integer did, String deptName) {
        this.did = did;
        this.deptName = deptName;
    }
}
```



#### 1.resultMap处理字段和属性的映射关系

> 若字段名和实体类中的属性名不一致，则可以通过resultMap设置自定义映射

```java
    /**
     * 查询所有的员工信息
     */
    List<Emp> getAllEmp();
```

```xml
    <!--
        resultMap：设置自定义映射关系
        id：唯一标识，不能重复
        type：设置映射关系中的实体类类型
        子标签：
        id：设置主键的映射关系
        result：设置普通字段的映射关系
        属性：
        property：设置映射关系中的属性名，必须是type属性所设置的实体类类型中的属性名
        column：设置映射关系中的字段名，必须是sql语句查询出的字段名
    -->
    <resultMap id="empResultMap" type="Emp">
        <id property="eid" column="eid"></id>
        <result property="empName" column="emp_name"></result>
        <result property="age" column="age"></result>
        <result property="sex" column="sex"></result>
        <result property="email" column="email"></result>
    </resultMap>
    <!--List<Emp> getAllEmp();-->
    <select id="getAllEmp" resultMap="empResultMap">
        select * from t_emp
    </select>
```

> 解决字段名和属性名称不一致的情况：
>
> a>查询的时候给字段起别名，保持和属性名一直
>
> ​	select emp_name as empName where t_emp
>
> b>设置全局配置，将_自动映射为驼峰
>
> ​	就是我全局配置中的 <setting name="mapUnderscoreToCamelCase" value="true"/>
>
> ​	字段名user_name，设置了mapUnderscoreToCamelCase，此时字段名就会转换为userName
>
> c>通过resultMap设置自定义的映射关系
>
> ```
> <resultMap id="empResultMap" type="Emp">
>  	 <id property="eid" column="eid"></id>
>      <result property="empName" column="emp_name"></result>
>      <result property="age" column="age"></result>
>      <result property="sex" column="sex"></result>
>      <result property="email" column="email"></result>
>  </resultMap>
> ```

#### 2.多对一映射处理

> 查询员工信息以及员工所对应的部门信息

##### 1.级联处理处理映射关系

```java
   /**
     * 查询员工以及员工所对应的部门信息
     */
    Emp getEmpAndDept(@Param("eid") Integer eid);
```

```xml
  <!--处理多对一映射关系方式一：级联属性赋值-->
    <resultMap id="empAndDeptResultMapOne" type="Emp">
        <id property="eid" column="eid"></id>
        <result property="empName" column="emp_name"></result>
        <result property="age" column="age"></result>
        <result property="sex" column="sex"></result>
        <result property="email" column="email"></result>
        <result property="dept.did" column="did"></result>
        <result property="dept.deptName" column="dept_name"></result>
    </resultMap>
    <!--Emp getEmpAndDept(@Param("eid") Integer eid);-->
    <select id="getEmpAndDept" resultMap="empAndDeptResultMapOne">
        select * from t_emp left join t_dept on t_emp.did = t_dept .did where t_emp.eid = #{eid}
    </select>
```

##### 2.使用association处理映射关系

```java
  /**
     * 查询员工以及员工所对应的部门信息
     */
    Emp getEmpAndDept(@Param("eid") Integer eid);
```

```xml
    <!--处理多对一映射关系方式二：association-->
    <resultMap id="empAndDeptResultMapTwo" type="Emp">
        <id property="eid" column="eid"></id>
        <result property="empName" column="emp_name"></result>
        <result property="age" column="age"></result>
        <result property="sex" column="sex"></result>
        <result property="email" column="email"></result>
        <!--
            association:处理多对一的映射关系
            property:需要处理多对的映射关系的属性名
            javaType:该属性的类型
        -->
        <association property="dept" javaType="Dept">
            <id property="did" column="did"></id>
            <result property="deptName" column="dept_name"></result>
        </association>
    </resultMap>
    <!--Emp getEmpAndDept(@Param("eid") Integer eid);-->
    <select id="getEmpAndDept" resultMap="empAndDeptResultMapTwo">
        select * from t_emp left join t_dept on t_emp.did = t_dept .did where t_emp.eid = #{eid}
    </select>
```

##### 4.分布查询

```java
    /**
     * 通过分步查询查询员工以及员工所对应的部门信息
     * 分步查询第一步：查询员工信息
     */
    Emp getEmpAndDeptByStepOne(@Param("eid") Integer eid);

    /**
     * 通过分步查询查询员工以及员工所对应的部门信息
     * 分步查询第二步：通过did查询员工所对应的部门
     */
    Dept getEmpAndDeptByStepTwo(@Param("did") Integer did);
```

```xml
    <resultMap id="empAndDeptByStepResultMap" type="Emp">
        <id property="eid" column="eid"></id>
        <result property="empName" column="emp_name"></result>
        <result property="age" column="age"></result>
        <result property="sex" column="sex"></result>
        <result property="email" column="email"></result>
        <!--
            select:设置分步查询的sql的唯一标识（namespace.SQLId或mapper接口的全类名.方法名）
            column:设置分布查询的条件
        -->
        <association property="dept"
                     select="com.atguigu.mybatis.mapper.DeptMapper.getEmpAndDeptByStepTwo"
                     column="did"
                 ></association>
    </resultMap>
    <!--Emp getEmpAndDeptByStepOne(@Param("eid") Integer eid);-->
    <select id="getEmpAndDeptByStepOne" resultMap="empAndDeptByStepResultMap">
        select * from t_emp where eid = #{eid}
    </select>

    <!--Dept getEmpAndDeptByStepTwo(@Param("did") Integer did);-->
    <select id="getEmpAndDeptByStepTwo" resultType="Dept">
        select * from t_dept where did = #{did}
    </select>
```

> 延迟加载：
>
> ​	分步查询的优点：可以实现延迟加载，但是必须在核心配置文件中设置全局配置信息：
>
> lazyLoadingEnabled：延迟加载的全局开关。当开启时，所有关联对象都会延迟加载
>
> aggressiveLazyLoading：当开启时，任何方法的调用都会加载该对象的所有属性。 否则，每个属性会按需加载

> 在官网中

| **设置名**            | 描述                                                         | 有效值      | 默认值                                       |
| --------------------- | ------------------------------------------------------------ | ----------- | -------------------------------------------- |
| lazyLoadingEnabled    | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 | true /false | false                                        |
| aggressiveLazyLoading | 开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载（参考 `lazyLoadTriggerMethods`)。 | true /false | false （在 3.4.1 及之前的版本中默认为 true） |

> 在全局配置中加入
>
> ```
> <settings>
> 	  <setting name="lazyLoadingEnabled" value="true"/>
> 	  <setting name="aggressiveLazyLoading" value="false"/>
> </settings>
> ```

> 也可以用fetchType属性来配置是否延迟加载

```xml
 <resultMap id="empAndDeptByStepResultMap" type="Emp">
        <id property="eid" column="eid"></id>
        <result property="empName" column="emp_name"></result>
        <result property="age" column="age"></result>
        <result property="sex" column="sex"></result>
        <result property="email" column="email"></result>
        <!--
            select:设置分步查询的sql的唯一标识（namespace.SQLId或mapper接口的全类名.方法名）
            column:设置分布查询的条件
            fetchType:当开启了全局的延迟加载之后，可通过此属性手动控制延迟加载的效果
            fetchType="lazy|eager":lazy表示延迟加载，eager表示立即加载
        -->
        <association property="dept"
                     select="com.atguigu.mybatis.mapper.DeptMapper.getEmpAndDeptByStepTwo"
                     column="did"
                     fetchType="eager"></association>
    </resultMap>
    <!--Emp getEmpAndDeptByStepOne(@Param("eid") Integer eid);-->
    <select id="getEmpAndDeptByStepOne" resultMap="empAndDeptByStepResultMap">
        select * from t_emp where eid = #{eid}
    </select>
```

#### 3.一对多映射处理

##### 	1.collection

> 在实体类中新增private List<Emp> emps属性

```java
public class Dept {

    private Integer did;

    private String deptName;

    private List<Emp> emps;

    @Override
    public String toString() {
        return "Dept{" +
                "did=" + did +
                ", deptName='" + deptName + '\'' +
                ", emps=" + emps +
                '}';
    }

    public List<Emp> getEmps() {
        return emps;
    }

    public void setEmps(List<Emp> emps) {
        this.emps = emps;
    }

    public Integer getDid() {
        return did;
    }

    public void setDid(Integer did) {
        this.did = did;
    }

    public String getDeptName() {
        return deptName;
    }

    public void setDeptName(String deptName) {
        this.deptName = deptName;
    }

    public Dept() {
    }

    public Dept(Integer did, String deptName) {
        this.did = did;
        this.deptName = deptName;
    }
}
```

```java
    /**
     * 获取部门以及部门中所有的员工信息
     */
    Dept getDeptAndEmp(@Param("did") Integer did);
```

```xml
    <resultMap id="deptAndEmpResultMap" type="Dept">
        <id property="did" column="did"></id>
        <result property="deptName" column="dept_name"></result>
        <!--
            collection：处理一对多的映射关系
            ofType：表示该属性所对应的集合中存储数据的类型
        -->
        <collection property="emps" ofType="Emp">
            <id property="eid" column="eid"></id>
            <result property="empName" column="emp_name"></result>
            <result property="age" column="age"></result>
            <result property="sex" column="sex"></result>
            <result property="email" column="email"></result>
        </collection>
    </resultMap>
    <!--Dept getDeptAndEmp(@Param("did") Integer did);-->
    <select id="getDeptAndEmp" resultMap="deptAndEmpResultMap">
        select * from t_dept left join t_emp on t_dept.did = t_emp.did where t_dept.did = #{did}
    </select>
```

##### 2.分布查询

```java
    /**
     * 通过分步查询查询部门以及部门中所有的员工信息
     * 分步查询第一步：查询部门信息
     */
    Dept getDeptAndEmpByStepOne(@Param("did") Integer did);
    /**
     * 通过分步查询查询部门以及部门中所有的员工信息
     * 分步查询第二步：根据did查询员工信息
     */
    List<Emp> getDeptAndEmpByStepTwo(@Param("did") Integer did);
```

```xml
    <resultMap id="deptAndEmpByStepResultMap" type="Dept">
        <id property="did" column="did"></id>
        <result property="deptName" column="dept_name"></result>
        <collection property="emps"
                    select="com.atguigu.mybatis.mapper.EmpMapper.getDeptAndEmpByStepTwo"
                    column="did" fetchType="eager"></collection>
    </resultMap>
    <!--Dept getDeptAndEmpByStepOne(@Param("did") Integer did);-->
    <select id="getDeptAndEmpByStepOne" resultMap="deptAndEmpByStepResultMap">
        select * from t_dept where did = #{did}
    </select>

	    <!--List<Emp> getDeptAndEmpByStepTwo(@Param("did") Integer did);-->
    <select id="getDeptAndEmpByStepTwo" resultType="Emp">
        select * from t_emp where did = #{did}
    </select>
<!--延迟加载与association一致-->
```

> ​			 扩展：多列的值传递过去：
> ​			将多列的值封装map传递；
> ​			column="{key1=column1,key2=column2}"
>
> ```xml
> 	<collection property="emps"
>     select="com.atguigu.mybatis.mapper.EmpMapper.getDeptAndEmpByStepTwo"
>     column="{deptId=did}" fetchType="eager"></collection>
> 
>     <!--List<Emp> getDeptAndEmpByStepTwo(@Param("did") Integer did);-->
>     <select id="getDeptAndEmpByStepTwo" resultType="Emp">
>         select * from t_emp where did = #{deptId}
>     </select>
> ```

​	

### 8.动态SQL

#### 	1.if

> if：根据标签中test属性所对应的表达式决定标签中的内容是否需要拼接到SQL中

```java
   /**
     * 多条件查询
     */
    List<Emp> getEmpByCondition(Emp emp);
```

```xml
    <select id="getEmpByCondition" resultType="Emp">
        select * from t_emp where
       
            <if test="empName != null and empName != ''">
                emp_name = #{empName}
            </if>
            <if test="age != null and age != ''">
                and age = #{age}
            </if>
            <if test="sex != null and sex != ''">
                or sex = #{sex}
            </if>
            <if test="email != null and email != ''">
                and email = #{email}
            </if>
        
    </select>
```

> 这种情况当empname为null的时候 就会报错 因为会多一个and  可以拼接一个 1=1,或者用where标签

```xml
    <select id="getEmpByCondition" resultType="Emp">
        select * from t_emp where 1=1
       
            <if test="empName != null and empName != ''">
                and emp_name = #{empName}
            </if>
            <if test="age != null and age != ''">
                and age = #{age}
            </if>
            <if test="sex != null and sex != ''">
                or sex = #{sex}
            </if>
            <if test="email != null and email != ''">
                and email = #{email}
            </if>
        
    </select>
```

#### 2.where

> ```
> 当where标签中有内容时，会自动生成where关键字，并且将内容前多余的and或or去掉
> 当where标签中没有内容时，此时where标签没有任何效果
> 注意：where标签不能将其中内容后面多余的and或or去掉
> ```

```xml
    <select id="getEmpByCondition" resultType="Emp">
        select * from t_emp
        <where>
            <if test="empName != null and empName != ''">
                emp_name = #{empName}
            </if>
            <if test="age != null and age != ''">
                and age = #{age}
            </if>
            <if test="sex != null and sex != ''">
                or sex = #{sex}
            </if>
            <if test="email != null and email != ''">
                and email = #{email}
            </if>
        </where>
    </select>
```

#### 3.trim

> ```
> 若标签中有内容时：
> prefix：在trim标签中的内容的前面添加某些内容
> prefixOverrides：在trim标签中的内容的前面去掉某些内容
> suffix：在trim标签中的内容的后面添加某些内容
> suffixOverrides：在trim标签中的内容的后面去掉某些内容
> 若标签中没有内容时，trim标签也没有任何效果
> ```

```java
    /**
     * 多条件查询
     */
    List<Emp> getEmpByCondition(Emp emp);
```

```xml
    <!--List<Emp> getEmpByCondition(Emp emp);-->
    <select id="getEmpByCondition" resultType="Emp">
        select eid,emp_name,age,sex,email from t_emp
        <trim prefix="where" suffixOverrides="and|or">
            <if test="empName != null and empName != ''">
                emp_name = #{empName} and
            </if>
            <if test="age != null and age != ''">
                age = #{age} or
            </if>
            <if test="sex != null and sex != ''">
                sex = #{sex} and
            </if>
            <if test="email != null and email != ''">
                email = #{email}
            </if>
        </trim>
    </select>
```

#### 4.choose  when  otherwise

```
choose、when、otherwise，相当于switch case default
 when至少要有一个，otherwise最多只能有一个
```

```java
   /**
     * 测试choose、when、otherwise
     */
    List<Emp> getEmpByChoose(Emp emp);
```

```xml
    <!--List<Emp> getEmpByChoose(Emp emp);-->
    <select id="getEmpByChoose" resultType="Emp">
        select * from t_emp
        <where>
            <choose>
                <when test="empName != null and empName != ''">
                    emp_name = #{empName}
                </when>
                <when test="age != null and age != ''">
                    age = #{age}
                </when>
                <when test="sex != null and sex != ''">
                    sex = #{sex}
                </when>
                <when test="email != null and email != ''">
                    email = #{email}
                </when>
                <otherwise>
                    did = 1
                </otherwise>
            </choose>
        </where>
    </select>
```

#### 5.foreach

> ```
> collection:设置需要循环的数组或集合
> item:表示数组或集合中的每一个数据
> separator:循环体之间的分割符
> open:foreach标签所循环的所有内容的开始符
> close:foreach标签所循环的所有内容的结束符
> index:集合中元素迭代时的索引
> ```

```java
    /**
     * 通过数组实现批量删除
     */
    int deleteMoreByArray(@Param("eids") Integer[] eids);
```

```xml
    <delete id="deleteMoreByArray">
        <!--<include refid="sqlcolumns"></include>-->
        delete from t_emp where
        <foreach collection="eids" item="eid" separator="or">
            eid = #{eid}
        </foreach>

    </delete>
```

```java
    /**
     * 通过list集合实现批量添加的功能
     */
    Integer insertMoreByList(@Param("emps") List<Emp> emps);
```

```xml
    <insert id="insertMoreByList">
        <foreach collection="emps" item="emp" open="begin" close="end;">
            insert into t_emp(eid) values (#{emp.eid});
        </foreach>
    </insert>
```

> oralce的批量添加不同 mysql用mysql的批量添加语法即可

#### 6.SQL片段

> sql片段，可以记录一段公共sql片段，在使用的地方通过include标签进行引入
>
> ```
> 设置SQL片段：<sql id="empColumns">eid,emp_name,age,sex,email</sql>
> 引用SQL片段：<include refid="empColumns"></include>
> ```

```java
    /**
     * 通过数组实现批量删除
     */
    int deleteMoreByArray(@Param("eids") Integer[] eids);
```

```xml
    <sql id="sqlcolumns">delete from t_emp where</sql>
    <delete id="deleteMoreByArray">
        <include refid="sqlcolumns"></include>
        <foreach collection="eids" item="eid" separator="or">
            eid = #{eid}
        </foreach>

    </delete>
```


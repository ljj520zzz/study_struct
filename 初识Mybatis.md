---
title: Mybatis笔记
date: 2022/12/28
tags:
- ssm
- 框架
categories: SSM
cover: https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/13704/wallhaven-3lz8z3.png
---


## 初识Mybatis

### 框架的概述

- **生活中的框架**
  - 买房子（但是具体装修要自己来实现）
  - 笔记本电脑
- 程序中的框架【代码半成品】
  - Mybatis框架 ：持久化层框架【dao层】
  - SpringMVC框架：控制层框架【Servlet层】
  - Spring框架：全能

### Mybatis简介

- Mybatis是一个**半自动化**持久化层**ORM**框架
- ORM：Object Relational Mapping【对象  关系  映射】
  -  将Java中的**对象**与数据库中**表**建议**映射关系**，优势：操作Java中的对象，就可以影响数据库中表的数据

- Mybatis与Hibernate对比
  - Mybatis是一个半自动化【需要手写SQL】
  - Hibernate是全自动化【无需手写SQL】

- Mybatis与JDBC对比···
  - JDBC中的SQL与Java代码耦合度高
  - Mybatis将SQL与Java代码解耦
- Java POJO（Plain Old Java Objects，普通老式 Java 对象）
  - JavaBean  等同于  POJO

## Mybatis搭建

- ### 官网地址

  - 文档地址：https://mybatis.org/mybatis-3/
  - 源码地址：https://github.com/mybatis/mybatis-3

导入jar包

编写配置文件

使用核心类库

### 准备

- 建库建表建约束
- 准备maven工程

1. 导入jar包

   ```xml
   <!--导入MySQL的驱动包-->
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <version>5.1.37</version>
   </dependency>
   <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <version>8.0.26</version>
   </dependency>
   
   <!--导入MyBatis的jar包-->
   <dependency>
       <groupId>org.mybatis</groupId>
       <artifactId>mybatis</artifactId>
       <version>3.5.6</version>
   </dependency>
   <!--junit-->
   <dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
       <version>4.12</version>
       <scope>test</scope>
   </dependency>
   ```

2. 编写核心配置文件【mybatis-config.xml】

   - 位置：resources目录下

   - 名称：推荐使用mybatis-config.xml（官方推荐的）

   - 实例代码

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE configuration
             PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
             "https://mybatis.org/dtd/mybatis-3-config.dtd">
     <configuration>
         <environments default="development">
             <environment id="development">
                 <transactionManager type="JDBC"/>
     <!--           mysql8版本-->
                 <dataSource type="POOLED">
                     <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                     <property name="url" value="jdbc:mysql://localhost:3306/1221?serverTimezone=UTC"/>
                     <property name="username" value="root"/>
                     <property name="password" value="123456"/>
                 </dataSource>
             </environment>
         </environments>
     <!--    设置映射文件的路径-->
       <mappers>
             <mapper resource="mapper/EmployeeMapper.xml"/>
         </mappers>
     </configuration>
     ```
     

3. 书写相关接口及映射文件

   - 映射文件位置：resources/mapper

   - 映射文件名称：XXXMapper.xml

   - **映射文件作用：主要作用为Mapper接口书写Sql语句**
     - 映射文件名与接口名一致
     - 映射文件namespace与接口全类名一致
     - 映射文件SQL的Id与接口的方法名一致
     
   - 代码实例

     ```xaml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE mapper
             PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
             "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
     
     
     
     <mapper namespace="mybatis.mapper.EmployeeMapper">
         <select id="selectEmpById" resultType="mybatis.pojo.Employee">
             select id,last_name,email,salary from tbl_employee where id=#{empId}
         </select>
     </mapper>
     ```

     

4. 测试【sqlsession】

   - 先获取sqlSessionFactory对象
   - 再获取Sqlsession对象
   - 通过Sqlsession对象获取XxxMapper代理对象
   - 测试

### 添加Log4j日志框架

1. 导入jar包

   ```xml
   <!-- log4j -->
   <dependency>
       <groupId>log4j</groupId>
       <artifactId>log4j</artifactId>
       <version>1.2.17</version>
   </dependency>
   ```

2. 编写核心配置文件

   - 配置文件名：log4j.xml

   - 配置文件位置：resources

   - 实例

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
     
     <log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">
     
         <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender">
             <param name="Encoding" value="UTF-8" />
             <layout class="org.apache.log4j.PatternLayout">
                 <param name="ConversionPattern" value="%-5p %d{MM-dd HH:mm:ss,SSS} %m  (%F:%L) \n" />
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

## Mybatis核心配置详解

### 核心配置文件的概述

- MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息

### 核心配置文件根标签

- 没有实际的语意，主要作用：所有的子标签都需要设置在根标签内部

### 核心配置文件常用子标签

- ### properties子标签

  -  作用：定义或者引入外部属性文件

  - 代码实例

    ```properties
    #key = value
    db.driver = com.mysql.cj.jdbc.Driver
    db.url = jdbc:mysql://localhost:3306/1221?serverTimezone=UTC
    db.username = root
    db.password = 123456
    ```

    ```xml
    <properties resource="db.properties"></properties>
    ```

    ```xml
    <dataSource type="POOLED">
                    <property name="driver" value="${db.driver}"/>
                    <property name="url" value="${db.url}"/>
                    <property name="username" value="${db.username}"/>
                    <property name="password" value="${db.password}"/>
    </dataSource>
    ```

- ### settings(子标签）

  - 作用：这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。
  - mapUnderscoreToCamelCase：是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn（可以避免起别名），默认值：false
    - **注意：只能将字母相同的字段和属性相映射，位置要一致**

- ### 类型别名（typeAliases）

  - 作用：类型别名可为 Java 类型设置一个缩写名字。 它仅用于 XML 配置，意在降低冗余的全限定类名书写。

  - 例如：

    ```xml
    <mapper namespace="mybatis.mapper.EmployeeMapper">
        <select id="selectEmpById" resultType="mybatis.pojo.Employee"><!-- 这样就是全类名的书写 -->
            select id,last_name,email,salary from tbl_employee where id=#{empId}
        </select>
    </mapper>
    ```

  - 使用

    ```xml
    <!--  为指定类型定义别名  -->
    <typeAliases>
            <typeAlias alias="employee" type="mybatis.pojo.Employee"/>
    </typeAliases>
    ```

  - 也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，比如：

    ```xml
<!-- 为指定包下所有的类定义别名 -->    
    <typeAliases>
      <package name="mybatis.pojo"/>
    </typeAliases>
    ```

- ### 环境配置（environments）

  - 设置数据库的连接环境

  - 实例代码

    ```xml
    <!--  设置数据库的连接环境  -->
        <environments default="development">
            <environment id="development">
                <transactionManager type="JDBC"/>
    <!--           mysql8版本-->
                <dataSource type="POOLED">
                    <property name="driver" value="${db.driver}"/>
                    <property name="url" value="${db.url}"/>
                    <property name="username" value="${db.username}"/>
                    <property name="password" value="${db.password}"/>
                </dataSource>
            </environment>
        </environments>
    ```

- ### 映射器（mappers）

  - 作用：设置映射文件的路径

  - 实例

    ```xml
    <!--    设置映射文件的路径-->
    <mappers>
        <mapper resource="mapper/EmployeeMapper.xml"/>
    </mappers>
    ```

- 注意：在核心的配置文件中，每一个子标签都是有顺序的，顺序乱了就会报错，如下图。

  ![image-20221222125947293](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/13704/image-20221222125947293.png)

## Mybatis映射文件详解

- MyBatis 的真正强大在于它的语句映射，这是它的魔力所在。由于它的异常强大，映射器的 XML 文件就显得相对简单。如果拿它跟具有相同功能的 JDBC 代码进行对比，你会立即发现省掉了将近 95% 的代码。MyBatis 致力于减少使用成本，让用户能更专注于 SQL 代码。

### 映射文件的根标签

- mapper
  - namespace要求与接口的全类名一致

### 映射文件子标签【共9个】

主要掌握前8个

-  insert标签：定义添加SQL
- delete标签：定义删除SQL
- update标签：定义修改SQL
- select标签：定义查询SQL
- sql标签：定义可重用的SQL语句块
- cache标签：设置当前命名空间的缓存配置
- cache-ref标签:设置其他命名空间的缓存配置
- **resultMap**标签: 描述如何从数据库结果集中加载对象
  - resultType解决不了的问题，交给resultMap

### 映射文件中常用属性

- resultType：设置期望结果集返回类型【全类名或别名】
  - 注意：如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身的类型。 
  - resultType 和 resultMap 之间只能同时使用一个。

## 获取主键自增数据

- userGeneratedKeys:启用主键生成策略
- keyProperty:设置存储属性值

```xml
<insert id="insertEmp" keyProperty="id" useGeneratedKeys="true">
    insert into tbl_employee values(null,#{lastName},#{email},#{salary})
</insert>
```

## 获取数据库受影响行数

- 直接将接口的返回值设置成boolean或int即可
  - int:表示受影响的行数
  - boolean：表示对数据库的操作是否成功

## Mybatis中参数传递问题

### 单个普通参数

- 可以任意使用：参数数据类型，参数名称不用考虑

  ```xml
  <select id="selectEmpById" resultType="employee">
      select id,last_name,email,salary from tbl_employee where id=#{empId}<!-- 这里的empId的名称可以换成任意一个名称，可以不用和接口中的名称保持一致 -->
  </select>
  ```

### 多个普通参数

- Mybatis封装成Map的结构，key值为【param1，param2】或者【arg1，arg2】，不能直接设置为对应的参数名

  - 不然就会报下面的错误

    ![image-20221223110443414](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/13704/image-20221223110443414.png)

  - 参数的顺序也不能反

  - 代码实例

    ```xml
    <select id="findByAny"  resultType="employee">
        select * from tbl_employee where last_name=#{arg0} and salary=#{arg1}
    </select>
    ```

### 命名参数

- 语法：

  - @Param(value=“参数名”)
  - @Param（“参数名”）

- 位置：参数前面

- 注意：

  - 底层还是封装了Map结构
  - 命名参数还是支持【param1，param2】

- 示例代码

  ```java
  public List<Employee> findByName(@Param("lName") String lastname,@Param("salary") Double salary);
  ```

  ```xml
  <select id="findByName"  resultType="employee">
          select * from tbl_employee where last_name=#{lName} and salary=#{salary}
  </select>
  ```

- 源码分析

  - MapperMethod对象【命名参数的底层源码】

    ```java
    Object param = method.convertArgsToSqlCommandParam(args);
    ```

  - **底层封装了paramMap,它继承了HashMap。**

  - ParamNameResolver是命名参数实现的底层逻辑(130行)

    ```java
    final Map<String, Object> param = new ParamMap<>();
    int i = 0;
    for (Map.Entry<Integer, String> entry : names.entrySet()) {
        param.put(entry.getValue(), args[entry.getKey()]);
        // add generic param names (param1, param2, ...)
        final String genericParamName = GENERIC_NAME_PREFIX + (i + 1);
        // ensure not to overwrite parameter named with @Param
        if (!names.containsValue(genericParamName)) {
            param.put(genericParamName, args[entry.getKey()]);
        }
        i++;
    }
    return param;
    ```
  
    

### pojo参数

- Mybatis支持POJO【javaBean】 入参，参数key是POJO中属性.

- 实例代码

  ```xml
  <select id="findByEmployee"  resultType="employee">
          select * from tbl_employee where last_name=#{lastName} and salary=#{salary}
   </select>
  ```

### Map参数

- Mybatis支持直接Map入参，map的key=参数key

- 实例代码

  ```java
  public List<Employee> findByMap(Map<String,Object> map);
  ```

  ```xml
  <select id="findByMap" resultType="employee">
          select * from tbl_employee where last_name= #{lastName} and salary=#{salary}
  </select>
  ```

- 注意：

  - map传参不支持param1，param2这种形式

### Colletion|List|Array等参数

- 参数名：conlletcion,list,array

## Mybatis参数传递中 #与$区别

### 回顾JDBC

- DriverManager

- Connection

- Statement

  - 语法

    ```java
    Statement stmt = connect.createStatement();
    String sql= "SELECT * FROM cg_user WHERE userId=10086 AND name LIKE 'xiaoming'";
    ResultSet rs = stmt.executeUpdate(sql);
    ```

  - createStatement不会初始化，没有预处理，没次都是从0开始执行SQL,入参使用Sql【String】 拼接的方式

  - 单次执行性能最优

- PreparedStatement

  - 批量执行性能最优

  -  prepareStatement会先初始化SQL，先把这个SQL提交到数据库中进行预处理，多次使用可提高效率。入参使用占位符方式

  - 语法

    ```java
    PreparedStatement preparedStatement = connect.prepareStatement("SELECT * FROM cg_user WHERE userId= ? AND name LIKE ?");  
    preparedStatement .setInt(1, 10086 );  
    preparedStatement .setString(2, "xiaoming");  
    preparedStatement .executeUpdate();  
    ```

- ResultSet

### #与$区别

- 【#】 底层执行SQL语句的对象，使用**PreparedStatement**，预编译SQL,防止SQL注入，比较安全
- 【$】 底层执行SQL语句的对象使用**Statement**对象，为解决SQL注入安全隐患，相对不安全

### #与$使用场景

> 查询SQL：select col,col2 from table1 where col=? and col2=? group by ?,order by ? limit ?,?

- #使用场景，sql占位符位置均可使用#

- $使用场景，#解决不了的参数传递问题，均可以交给$处理【如：form动态化表名】

  - 代码实例

    ```xml
    <select id="findDynamicTable" resultType="employee">
            select * from ${table}
     </select>
    ```

    ```java
    public List<Employee> findDynamicTable(@Param("table") String table);
    ```

## Mybatis查询中返回值四种情况

### 查询单行数据返回单个对象

```xml
<select id="selectEmpById" resultType="employee">
    select id,last_name,email,salary from tbl_employee where id=#{empId}
</select>
```

### 查询多行数据返回对象的集合

> 但是在映射文件中resultType的值必须是你查询结果的对象，而不是集合对象

```xml
<select id="findAll" resultType="employee">
    select * from tbl_employee
</select>
```

```java
public List<Employee> findAll();
```

### 查询单行数据返回Map集合

- Map<Key,Value>

  - 数据库中的字段作为Map的key，查询结果作为Map的Value

- 代码实例

  - ```java
    // MAP作为返回值的场景
        public Map<String,Object> findMap(int empId);
    ```

  - ```xml
    <select id="findMap" resultType="map">
        select * from tbl_employee where id = #{empId}
    </select>
    ```

### 查询多行数据返回Map集合

- Map<Integer key,Employee value>

  - 对象的id作为key

  - 对象作为value

  - 实例代码

    ```java
    //    多行数据返回Map
        /*
            Map<Integer,Employee>
            员工的id作为map中的key
            员工对象作为map中的value
         */
        @MapKey("id")//这里做一个注释表示用id作为key
        public Map<Integer,Employee> findsMap();
    ```

    ```xaml
     <select id="findsMap" resultType="map">
            select * from tbl_employee
    </select>
    ```

## Mybatis中自动映射与自定义映射

> 自动映射【resultType】
>
> 自定义映射【resultMap】

### 自动映射与自定义映射

- 自动映射【resultType】：指的是自动将表中的字段与类中的属性进行关联映射
  - 自动映射解决不了两类问题
    - **多表连接查询，需要返回多张表的结果集**
    - ![image-20221224114054431](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/13704/image-20221224114054431.png)
    - 单表查询时，不支持驼峰式自动映射【又不想起别名】
- 自定义映射【resultMap】：自动映射解决不了的问题，交给自定义映射
- 注意：resultType和resultMap只能同时使用同一个

###  自定义映射-级联映射

- 代码示例

  ```xml
  <mapper namespace="mybatis.mapper.EmployeeMapper">
  <!--    自定义映射[员工与部门的关系]-->
      <resultMap id="empAndDeptResultMap" type="employee">
  <!--   id是定义主键与属性的关联关系     -->
          <id column="id" property="id"></id>
  <!--    result定义非主键与属性关联关系    -->
          <result column="last_name" property="lastName"></result>
          <result column="email" property="email"></result>
          <result column="salary" property="salary"></result>
  <!--    为员工中所属的部门，自定义关联关系    -->
          <result column="dept_id" property="dept.deptId"></result>
          <result column="dept_name" property="dept.deptName"></result>
      </resultMap>
      <select id="findEmpAndDeptById" resultMap="empAndDeptResultMap">
          select
          e.id,e.email,e.last_name,e.salary,
          d.dept_id,
          d.dept_name
          from
          tbl_employee e left join tbl_dept d on d.dept_id = e.dept_id
          where e.dept_id = #{empId};
      </select>
  </mapper>
  ```

###  自定义映射-association

- 特点：解决一对一映射关系【多对一】

- 示例代码

  ```xml
  <resultMap id="empAndDeptResultMapAssociation" type="employee">
          <!--   id是定义主键与属性的关联关系     -->
          <id column="id" property="id"></id>
          <!--    result定义非主键与属性关联关系    -->
          <result column="last_name" property="lastName"></result>
          <result column="email" property="email"></result>
          <result column="salary" property="salary"></result>
          <!--    为员工中所属的部门，自定义关联关系    -->
          <association property="dept" javaType="mybatis.pojo.Dept">
   <id column="dept_id" property="deptId"></id>
              <result column="dept_name" property="deptName"></result>
          </association>
      </resultMap>
      <select id="findEmpAndById" resultMap="empAndDeptResultMapAssociation">
          select
          e.id,e.email,e.last_name,e.salary,
          d.dept_id,
          d.dept_name
          from
          tbl_employee e left join tbl_dept d on d.dept_id = e.dept_id
          where e.id = #{empId};
      </select>
  ```

  

### **自定义映射-collection映射**

- ​	解决问题【一对多】

- 代码示例

  ```java
  //通过部门的id来获取部门信息,及相关员工信息
      public Dept findDeptAndEmpById(int deptId);
  ```

  ```xml
  <resultMap id="FindDeptAndEmp" type="Dept">
          <id property="deptId" column="dept_id"></id>
          <result property="deptName" column="dept_name"></result>
          <collection property="employees" ofType="mybatis.pojo.Employee">
              <id property="id" column="id"></id>
              <result property="lastName" column="last_name"></result>
              <result property="email" column="email"></result>
              <result property="salary" column="salary"></result>
          </collection>
      </resultMap>
      <select id="findDeptAndEmpById" resultMap="FindDeptAndEmp">
          select d.dept_id,d.dept_name,e.id,e.last_name,e.email,e.salary from tbl_dept d left join tbl_employee e on e.dept_id = d.dept_id where d.dept_id = #{deptId};
      </select>
  ```

  

### ResultMap相关标签及属性

- resultMap标签：自定义映射标签
  - id属性：定义唯一标识
  - type属性：设置映射类型
- resultMap子标签
  - id标签：定义主键字段与属性关联
  - result标签：定义非主键字段与属性关联关系
    - column属性：定义表中字段名称
    - property属性：定义类中的属性名称
  - association标签：定义一对一的关联关系
    - property：定义关联关系属性
    - javaType：定义关联关系属性的类型
    - select：设置分步查询SQL全路径
    - column：设置分步查询SQL中需要参数
  - collection标签：定义一对多的关联关系
    - property：
    - ofType:

### Mybatis中分步查询

> 需求：通过员工编号查询员工及员工对应的部门信息

- 为什么使用分布查询【分布查询优势】？

  - 将多表连接查询，改为【分布单表查询】，从而提高程序运行效率

- 步骤

  - 分步查询，通过员工的id先获取到员工的信息

  - 再通过员工的信息中的部门id获取到部门的信息

  - 先创建一个部门的(Mapper)接口，写下对应的方法

  - 在创建对应的Mapper.xml映射文件

    - 代码示例

      ```xml
      <?xml version="1.0" encoding="UTF-8" ?>
      <!DOCTYPE mapper
              PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
              "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
      <mapper namespace="mybatis.mapper.DeptMapper">
          <select id="findById" resultType="dept">
              select dept_id,dept_name from tbl_dept where dept_id= #{deptId}
          </select>
      </mapper>
      ```

  - 在EmployeeMpper中建立association自定义映射

    - 代码示例

      ```xml
      <resultMap id="AssociationStep" type="employee">
              <result property="id" column="id"></result>
              <result property="email" column="email"></result>
              <result property="salary" column="salary"></result>
              <association property="dept" select="mybatis.mapper.DeptMapper.findById" column="dept_id"></association>
          </resultMap>
      ```

    - 其中的select就是DeptMapper的全类名加上命名的方法

    - colum就是数据库的字段名

    - 代码示例

      ```xml
      <select id="findEmpAssociationStep" resultMap="AssociationStep">
              select
              id,
              last_name,
              email,
              salary,
              dept_id
              from
              tbl_employee
              where id = #{id}
          </select>
      ```

- **Collection**中的分步加载

  - 代码示例【通过部门id去查询相关的部门信息和员工信息】

    ```java
    public List<Employee> findEmploys(int deptId);
    ```

    ```java
    //通过部门的id来获取部门信息,及相关员工信息【分步查询】
     public Dept findDeptAndEmpByIdStep(int deptId);
    ```

    ```xml
    <resultMap id="DeptAndEmpByIdStep" type="dept">
            <id property="deptId" column="dept_id"></id>
            <result property="deptName" column="dept_name"></result>
            <collection property="employees" select="mybatis.mapper.EmployeeMapper.findEmploys" column="dept_id"></collection>
        </resultMap>
    <select id="findDeptAndEmpByIdStep" resultMap="DeptAndEmpByIdStep">
            select dept_id,dept_name from tbl_dept where dept_id = #{deptId}
    </select>
    ```

### Mybatis延迟加载【懒加载】

- 需要的时候就加载，不需要的时候就不加载

- 特点：可以提高程序的运行效率

- lazyLoadingEnabled

  - 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载

- aggressiveLazyLoading（设置加载的数据都是按需加载）

  - 开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载

- 语法

  - 全局设置

    ```xml
    <settings>
            <setting name="mapUnderscoreToCamelCase" value="true"/>
            <!-- 开启延迟加载 -->
            <setting name="lazyLoadingEnabled" value="true"/>
            <!--  设置加载的数据都是按需加载  在 3.4.1 及之前的版本中默认为 true，之后的版本可以省略   -->
            <setting name="aggressiveLazyLoading" value="false"/>
        </settings>
    ```

  - 局部的加载

    - fetchType
      - lazy:开启局部延迟加载
      - eager：关闭局部延迟加载

    ```xaml
    <association property="dept" select="mybatis.mapper.DeptMapper.findById" column="dept_id" fetchType="eager"></association>
    ```

    即使不设置全局懒加载，局部的设置了懒加载也可以生效

### 扩展：分布查询多列值的传递

- 如果分步查询时，需要传递给调用的查询中多个参数，则需要将多个参数封装成Map来进行传递，语法如下：**{k1=v1,k2=v2...}**

  - 代码演示

    ```xml
    <collection property="employees" select="mybatis.mapper.EmployeeMapper.findEmploys" column="{did=dept_id}"></collection>
    ```

    注意是中间相隔的是=不是：

## Mybatis动态SQL【重点】

### 动态SQL概述

- 动态SQL指的是：SQL语句可动态化
- Mybatis的动态SQL中支持OGNL表达式语言，OGNL对象导航图语言（Object Graph Navigation Language）

### 常用标签

- if标签：用于完成简单的判断

  代码示例

  ```xml
  <if test="id != null">
                  id = #{id}
              </if>
              <if test="lastName != null">
                  and last_name=#{lastName}
              </if>
              <if test="email != null">
                  and email=#{email}
              </if>
              <if test="salary != null">
                  and salary = #{salary}
              </if>
  ```

- where标签：用于解决where关键字及where后第一个and或or的问题

  代码示例

  ```xml
  <where>
              <if test="id != null">
                  id = #{id}
              </if>
              <if test="lastName != null">
                  and last_name=#{lastName}
              </if>
              <if test="email != null">
                  and email=#{email}
              </if>
              <if test="salary != null">
                  and salary = #{salary}
              </if>
          </where>
  ```

- **trim标签**： 可以在条件判断完的SQL语句前后添加或者去掉指定的字符

  prefix: 添加前缀

  prefixOverrides: 去掉前缀

  suffix: 添加后缀

  suffixOverrides: 去掉后缀

  ```xml
  <select id="findByDynamicTrim" resultType="employee">
          select
          id,
          last_name,
          email,
          salary
          from
          tbl_employee
          <trim prefix="where" suffixOverrides="and">
              <if test="id != null">
                  id = #{id} and
              </if>
              <if test="lastName != null">
                  and last_name=#{lastName} and
              </if>
              <if test="email != null">
                  and email=#{email} and
              </if>
              <if test="salary != null">
                   salary = #{salary}
              </if>
          </trim>
      </select>
  ```

- **set标签**：主要用于解决set关键字及多出一个【，】问题

  - 代码示例

    ```xml
    <update id="updateById">
            update
                tbl_employee
            <set>
                <if test="lastName != null">
                    last_name=#{lastName},
                </if>
                <if test="email != null">
                    email=#{email},
                </if>
                <if test="salary != null">
                    salary=#{salary}
                </if>
            </set>
            where
                id=#{id}
        </update>
    ```

- **choose标签**：类似java中if-else【switch-case】结构

  - 代码示例

    ```xml
    <select id="select" resultType="employee">
            select
                id,
                last_name,
                email,
                salary
            from
                tbl_employee
            <where>
                <choose>
                    <when test="id != null">
                        id =#{id}
                    </when>
                    <when test="lastName !=null">
                        last_name = #{lastName}
                    </when>
                    <otherwise>
                        1=1
                    </otherwise>
                </choose>
            </where>
        </select>
    ```

- **foreach标签**：类似java中for循环

  - **collection: 要迭代的集合**

  - **item: 当前从集合中迭代出的元素**

  - **separator: 元素与元素之间的分隔符**

  - open: 开始字符

  - close:结束字符

  - 代码示例

    ```xml
    <select id="selectByForeach" resultType="employee">
            select
                id,
                last_name,
                email,
                salary
            from
                tbl_employee
            <where>
                id in(
                <foreach collection="ids" item="id" separator=",">
                    #{id}
                </foreach>
                )
            </where>
        </select>
    ```

  - 批量添加

    ```xml
    <insert id="addEmployees">
            insert into
                tbl_employee (last_name,email,salary)
            values
                <foreach collection="employees" item="emp" separator=",">
                    (#{emp.lastName},#{emp.email},#{emp.salary})
                </foreach>
    </insert>
    ```

- sql标签：提取可重用SQL片段

  - 代码示例

  ```xml
  <sql id="select_from"><!-- id必须标注 -->
      select
          id,
          last_name,
          email,
          salary
      from
          tbl_employee
  </sql>
  
  <include refid="select_from"></include>
  ```

## Mybatis中的缓存机制

### 缓存概述

- 生活中缓存
  - 缓存一些音频、视频优势
    - 节约数据流量
    - 提高播放性能
- 程序中缓存【Mybatis缓存】
  - 使用缓存优势
    - 提高查询效率
    - 降低服务器压力

### Mybatis中的缓存概述

- 一级缓存

- 二级缓存

- 第三方缓存

  ![image-20221226124803371](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/13704/image-20221226124803371.png)

### Mybatis缓存机制-一级缓存

- 概述：一级缓存【本地缓存（Local Cache）或者SqlSession级别缓存】

- 特点
  - 一级缓存默认开启
  - 不能关闭
  - 可以清空

- 缓存原理
  
  - 第一次获取数据时，先从数据库中加载数据，将数据缓存至Mybatis一级缓存中【缓存底层实现原理Map，key：hashCode+查询的SqlId+编写的sql查询语句+参数】
  
- 以后再次获取数据时，先从一级缓存中获取，**如未获取到数据**，再从数据库中获取数据。

- **一级缓存五种失效情况**

  1) 不同的SqlSession对应不同的一级缓存

  2) 同一个SqlSession但是查询条件不同

  3) **同一个SqlSession两次查询期间执行了任何一次增删改操作**	

  - 清空一级缓存

  4) 同一个SqlSession两次查询期间手动清空了缓存

  - **sqlSession.clearCache()**

  5) 同一个SqlSession两次查询期间提交了事务

  - sqlSession.commit()

### Mybatis缓存机制-二级缓存

- 二级缓存【second level cache】概述

  - 二级缓存【全局作用域缓存】
  - SqlSessionFactory级别缓存

- 二级缓存特点

  - 二级缓存默认关闭，需要开启才能使用
  - 二级缓存需要提交sqlSession或者关闭SqlSession时，才会缓存

- 二级缓存使用的步骤：

  ① 全局配置文件中开启二级缓存<setting name="cacheEnabled" value="true"/>

  ② 需要使用二级缓存的**映射文件处**使用cache配置缓存<cache />

  ③ 注意：POJO需要实现Serializable接口

  ![image-20221227104738144](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/13704/image-20221227104738144.png)

  安装这个插件可以快捷生成序列化id

  ④ **关闭sqlSession或提交sqlSession时，将数据缓存到二级缓存**
  
- 二级缓存底层原理

  - 第一次获取数据时，先从数据库获取数据，将数据缓存至一级缓存；当提交或关闭SqlSession时，将数据缓存至二级缓存
  - 以后再吃获取数据时，先从一级缓存获取数据，如果一级缓存没有，才去二级缓存中获取数据，如二级缓存也没有指定的数据，需要去数据库获取数据

- 二级缓存相关属性

  - eviction=“FIFO”：缓存清除【回收】策略。
    - LRU – 最近最少使用的：移除最长时间不被使用的对象。
    - FIFO – 先进先出：按对象进入缓存的顺序来移除它们。
  - flushInterval：刷新间隔，单位毫秒
  - size：引用数目，正整数
  - readOnly：只读，true/false

- 二级缓存的失效情况

  - 在两次查询之间，执行增删改操作，会同时清空一级缓存和二级缓存
  - 注意：sqlSession.clearCache()：只是用来清除一级缓存。

###  Mybatis中缓存机制-第三方缓存

- 第三方缓存：EhCache

- EhCache 是一个纯Java的进程内缓存框架

- 使用步骤

  - 导入jar包

    ```xaml
    <!-- mybatis-ehcache -->
    <dependency>
        <groupId>org.mybatis.caches</groupId>
        <artifactId>mybatis-ehcache</artifactId>
        <version>1.0.3</version>
    </dependency>
    
    <!-- slf4j-log4j12 -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.6.2</version>
        <scope>test</scope>
    </dependency>
    ```

  - 编写配置文件【ehcache.xml】

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
        <!-- 磁盘保存路径 -->
        <diskStore path="E:\mybatis\ehcache" />
    
        <defaultCache
                maxElementsInMemory="512"
                maxElementsOnDisk="10000000"
                eternal="false"
                overflowToDisk="true"
                timeToIdleSeconds="120"
                timeToLiveSeconds="120"
                diskExpiryThreadIntervalSeconds="120"
                memoryStoreEvictionPolicy="LRU">
        </defaultCache>
    </ehcache>
    ```

  - 加载第三方缓存【映射文件】

    ```xml
    <cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>
    ```

  - 开始使用

- 注意事项

  - 第三方缓存，需要建立在二级缓存基础上【需要开启二级缓存，第三方缓存才能生效】
  - 如何让第三方缓存失效【将二级缓存设置失效即可】

## Mybatis逆向工程

### 逆向工程概述

- 正向工程：应用程序中代码影响数据库表中数据【java对象影响表】
- 逆向工程：数据库中表影响程序中代码【表影响java对象（POJO&&XXXMapper&&XXXMapper.xml）】

### MBG简个

- MyBatis Generator: 简称MBG
- 是一个专门为MyBatis框架使用者定制的代码生成器
- **可以快速的根据表生成对应的映射文件，接口，以及bean类。**
- 只可以生成单表CRUD【同时支持QBC风格CRUD】，但是表连接、存储过程等这些复杂sql的定义需要我们手工编写
- 官方文档地址
  http://www.mybatis.org/generator/

### MBG的基本使用

MBG基本使用

- 使用步骤

  导入jar包

  ```xml
  <!-- mybatis-generator-core -->
  <dependency>
      <groupId>org.mybatis.generator</groupid>
      <artifactId>mybatis-generator-core</artifactId>
      <version>1.3.6</version>
  </dependency>
  ```

  编写配置文件

  - 配置名称：mbg.xml
  - 配置位置：resources

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE generatorConfiguration
          PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
          "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
  
  <generatorConfiguration>
  <!--    id属性: 设置一个唯一标识-->
  <!--    targetRuntime属性值说明: T-->
  <!--    MyBatis3simple: 基本的增删改查MyBatis3: 带条件查询的增删改查【QBC风格】-->
  <!--    注意: 如果使用的是MySQL8，在jdbcConnection标签中还需要添加以下标签<property name="nullCatalogMeansCurrent” value=true”/>-->
      <context id="simple" targetRuntime="MyBatis3Simple">
          <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                          connectionURL="jdbc:mysql://localhost:3306/1221?serverTimezone=UTC"
                          userId="root"
                          password="123456">
              <property name="nullCatalogMeansCurrent" value="true"/>
          </jdbcConnection>
  
          <javaTypeResolver >
              <property name="forceBigDecimals" value="false" />
          </javaTypeResolver>
  <!--        设置javabean生成策略-->
          <javaModelGenerator targetPackage="mybatis.pojo" targetProject="src/main/java"/>
  <!--设置SQL映射文件的生成策略-->
          <sqlMapGenerator targetPackage="mapper"  targetProject="src/main/resources"/>
  <!--设置Mapper接口的生成策略-->
          <javaClientGenerator type="XMLMAPPER" targetPackage="mybatis.mapper"  targetProject="src/main/java"/>
  <!--逆向分析的表-->
          <table  tableName="tbl_employee" domainObjectName="Employee"/>
          <table  tableName="tbl_dept" domainObjectName="Department"/>
      </context>
  </generatorConfiguration> 
  ```

  运行程序【代码生成器】

  ```java
  List<String> warnings = new ArrayList<String>();
  boolean overwrite = true;
  File configFile = new File("generatorConfig.xml");
  ConfigurationParser cp = new ConfigurationParser(warnings);
  Configuration config = cp.parseConfiguration(configFile);
  DefaultShellCallback callback = new DefaultShellCallback(overwrite);
  MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
  myBatisGenerator.generate(null);
  ```

  代码示例

  ```java
  //【QBC风格代码】
  public void test03() throws IOException {
          String resource = "mybatis-config.xml";
          InputStream inputStream = Resources.getResourceAsStream(resource);
          SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
          SqlSession sqlSession = sqlSessionFactory.openSession();
          EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);
  //        System.out.println(mapper.selectByPrimaryKey(1));
          //创建员工对应的条件对象
          EmployeeExample ee = new EmployeeExample();
          //创建条件对象
          EmployeeExample.Criteria criteria = ee.createCriteria();
          //为条件对象添加条件
          List<Integer> list = new ArrayList<>();
          list.add(1);
          list.add(4);
          list.add(5);
          criteria.andIdIn(list);
          criteria.andLastNameLike("li%");
          List<Employee> employees = mapper.selectByExample(ee);
          System.out.println(employees);
      
      
  //简易风格
          public void test02() throws IOException {
  //        String resource = "mybatis-config.xml";
  //        InputStream inputStream = Resources.getResourceAsStream(resource);
  //        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
  //        SqlSession sqlSession = sqlSessionFactory.openSession();
  //        DepartmentMapper mapper = sqlSession.getMapper(DepartmentMapper.class);
  //        List<Department> departments = mapper.selectAll();
  //        System.out.println(departments);
  
      }
  ```

## Mybatis分页插件-PageHelp【重点】

### 分页基本概念【为什么使用分页】

- 提高用户体验度
- 降低服务器压力

### 设计Page类【自己手动设计应该有的东西】

> 47/60 47:当前页码 60: 总页数
>
> SELECT*FROM tbl employee WHERE 1=1 LIMIT x,y -- x:开启下标 :每页显示数据数量

- pageNum：当前页码
- pages：总页数【计算：总页数=总数据数量/每页显示数据数量】
- total：总数据数量
- pageSize：每页显示数据数量
- List<T>:当前页显示数据集合

### PageHelper概述

- 概述：PageHelper是MyBatis中非常方便的第三方分页插件
- 1) 官方文档
  https://github.com/pagehelper/Mybatis-PageHelper/blob/master/README_zh.md

### PageHelper使用步骤

- 导入相关jar包

  - 

    ```xml
    <!-- pagehelper -->
            <dependency>
                <groupId>com.github.pagehelper</groupId>
                <artifactId>pagehelper</artifactId>
                <version>5.0.0</version>
            </dependency>
    ```

- 在mybatis-config.xml中配置分页插件

  ```xml
  <plugins>
  <!--配置pageHelper分页插件-->
  <plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin></plugins>
  ```

- 查询之前，使用PageHelper开启分页

  PageHelper.**startPage**(1,3);

  ```java
  System.out.println(page.getPageNum()+"/"+page.getPages());
  System.out.println("显示总数量" + page.getTotal());
  System.out.println("每页显示多少个 " + page.getPageSize());
  ```

  查询之后，最后将结果封装PageInfo中，使用PageInfo实现后续分页效果
  
  ```java
  String resource = "mybatis-config.xml";
  InputStream inputStream = Resources.getResourceAsStream(resource);
  SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
  SqlSession sqlSession = sqlSessionFactory.openSession();
  EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);
  EmployeeExample ee = new EmployeeExample();
  PageHelper.startPage(1, 2);
  List<Employee> employees = mapper.selectByExample(ee);
  PageInfo<Employee> pageInfo = new PageInfo<>(employees,5);
  System.out.println("当前分页="+pageInfo.getPageNum()+"/"+pageInfo.getPages());
  System.out.println("是否有下一页"+pageInfo.isHasNextPage());
  System.out.println("是否有上一页"+pageInfo.isHasPreviousPage());
  System.out.println("第一页="+pageInfo.getNavigateFirstPage());
  System.out.println("最后一页="+pageInfo.getNavigateLastPage());
  ```
  
  
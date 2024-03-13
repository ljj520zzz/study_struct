---
title: 从 JDBC 到 ORM（例：Mybatis）的演化过程
date: 2022-11-29 19:03:04
tags: [jdbc,orm,mysql]
categories: 数据库
---

# 从 JDBC 到 ORM（例：Mybatis）的演化过程

​	下面我将介绍Java操作Mysql数据的方式的演化过程，从最基本的JDBC到ORM框架的实现，每一次演化都是为了解决现有存在的问题。

## JDBC

这里需要加入Mysql驱动包或者依赖.

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.13</version>
</dependency>
```

```java
@Test
public void test1() throws SQLException {
    // 注册驱动
    Driver driver = new com.mysql.jdbc.Driver();
    // 设置配置
    Properties properties = new Properties();
    properties.setProperty("user","root");
    properties.setProperty("password","123456");
    String url = "jdbc:mysql://localhost:3306/orm";
    // 获取连接
    Connection connect = driver.connect(url, properties);
    // 执行SQL
    String sql = "select * from user";
    Statement statement = connect.createStatement();
    // 返回结果
    ResultSet resultSet = statement.executeQuery(sql);
    // 结束资源
    resultSet.close();
    statement.close();
    connect.close();
}
```

​	可以看到我们是通过`Statement`去执行的SQL语句，这里是存在SQL注入风险的，简单来说就是SQL是字符串拼接的，如果有恶意参数，会影响整个SQL的意思，所以后面我们使用`PreparedStatement `，也叫做预处理，执行SQL语句的参数用`(?)`来表示,使用set方法插入值。好处：1. 防止SQL注入 2. 减少编译次数，效率高 3. 不在使用SQL拼接，减少语法错误。

```java
@Test
public void test1() throws SQLException {
    // 注册驱动
    Driver driver = new com.mysql.jdbc.Driver();
    // 设置配置
    Properties properties = new Properties();
    properties.setProperty("user","root");
    properties.setProperty("password","123456");
    String url = "jdbc:mysql://localhost:3306/orm";
    // 获取连接
    Connection connect = driver.connect(url, properties);
    // 执行SQL
    String sql = "select * from user where username =?";
    PreparedStatement statement = connect.prepareStatement(sql);
    statement.setString(1,"steak");
    // 返回结果
    ResultSet resultSet = statement.executeQuery();
    // 结束资源
    resultSet.close();
    statement.close();
    connect.close();
}
```

## 封装JDBC工具类

​	上面的操作，其中资源配置都是固定操作，所以我们可以直接写一个工具类来每次获取连接，进行操作，释放资源。

```java
import java.io.FileInputStream;
import java.io.IOException;
import java.sql.*;
import java.util.Properties;

public class JDBCUtils {
    private static String username;
    private static String password;
    private static String url;
    private static String driver;

    static {
        try {
            Properties properties = new Properties();
            // 将配置写入配置文件，更加灵活
            properties.load(new FileInputStream("src\\mysql.properties"));
            username = properties.getProperty("user");
            password = properties.getProperty("password");
            url = properties.getProperty("url");
            driver = properties.getProperty("driver");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static Connection getConnection() {
        try {
            return DriverManager.getConnection(url, username, password);
        } catch (SQLException e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }
    }

    public static void close(ResultSet set, Statement statement, Connection connection) throws SQLException {
        if (set != null){
            set.close();
        }
        if (statement != null) {
            set.close();
        }
        if (connection != null) {
            connection.close();
        }
    }
}
```

## 使用连接池

​	在上面我们每次操作数据库时，都是获取连接，操作，断开连接。

​	传统的JDBC操作会频繁的请求和验证，占用很多系统资源，导致服务崩溃，如果程序出现问题未能正常关闭，将导致数据库内存泄露，组织共导致重启数据库。

​	而且不能控制连接的数量，如果连接过多，可能导致Mysql崩溃。所以我们引入了连接池，让连接复用。这里我使用德鲁伊连接池，只用引入先关依赖即可：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.8</version>
</dependency>
```

> 这里我直接把原先的获取连接方式，改为使用连接池获取。

```java
import com.alibaba.druid.pool.DruidDataSourceFactory;
import javax.sql.DataSource;
import java.io.FileInputStream;
import java.io.IOException;
import java.sql.*;
import java.util.Properties;

public class JDBCUtils {
    private static DataSource dataSource;
    
    static {
        try {
            Properties properties = new Properties();
            // 将配置写入配置文件
            properties.load(new FileInputStream("src\\mysql.properties"));
            dataSource = DruidDataSourceFactory.createDataSource(properties);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static Connection getConnection() {
        try {
            return dataSource.getConnection();
        } catch (SQLException e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }
    }

    public static void close(ResultSet set, Statement statement, Connection connection) throws SQLException {
        if (set != null){
            set.close();
        }
        if (statement != null) {
            set.close();
        }
        // 在数据库连接池技术中，close 不是真的断掉连接，而是把使用的 Connection 对象放回连接池
        if (connection != null) {
            connection.close();
        }
    }
}
```

## Apache—DBUtils

​	上面的操作存在一个问题，关闭connection之后，resultSet结果集无法使用，且resultSet不利于数据的管理，所以每次我们查询了数据，可以手动的添加到List，Map等容器使用。还存在一个问题，每次查询返回的都是ResultSet，能不能通过泛型和反射，直接将数据封装成对象或者到List中。这些操作就被`Apache—DBUtils`完成了，是Apache提供的开源JDBC工具类库，且内部对SQL执行进行了线程安全保证。

```xml
<dependency>
    <groupId>commons-dbutils</groupId>
    <artifactId>commons-dbutils</artifactId>
    <version>1.6</version>
</dependency>
```

```java
@Test
public void test1() throws SQLException {
    Connection connection = JDBCUtils.getConnection();
    QueryRunner queryRunner = new QueryRunner();
    String sql = "select * from user where id >= ?";
    List<User> list = queryRunner.query(connection, sql, new BeanListHandler<>(User.class));
    JDBCUtils.close(null, null, connection);
}
```



## DAO与增删改查通用方法-BasicDao

> Dao : data access object 数据访问对象

​	我们希望将增删改查一些公共的方法抽取出来，BasicDao是专门和数据库交互的，在BasicDao的基础上，实现一张表对应一个Dao，更好的完成功能，比如Customer表对应就是CustomerDao。

```java
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.BeanHandler;
import org.apache.commons.dbutils.handlers.BeanListHandler;
import org.apache.commons.dbutils.handlers.ScalarHandler;

import java.sql.Connection;
import java.sql.SQLException;
import java.util.List;

public class BasicDAO<T> {
    private QueryRunner qr = new QueryRunner();

    // 增删改操作
    public int update(String sql, Object... parameters) throws SQLException {
        Connection connection = null;
        try {
            connection = JDBCUtils.getConnection();
            int update = qr.update(connection, sql, parameters);
            return update;
        } catch (SQLException e) {
            throw new RuntimeException(e); //将编译异常->运行异常 ,抛出
        } finally {
            JDBCUtils.close(null, null, connection);
        }
    }

    // 查询并封装多个对象
    public List<T> queryMulti(String sql, Class<T> clazz, Object... parameters) throws SQLException {
        Connection connection = null;
        try {
            connection = JDBCUtils.getConnection();
            return qr.query(connection, sql, new BeanListHandler<T>(clazz), parameters);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        } finally {
            JDBCUtils.close(null, null, connection);
        }
    }

    // 查询并封装单个对象
    public T querySingle(String sql, Class<T> clazz, Object... parameters) throws SQLException {
        Connection connection = null;
        try {
            connection = JDBCUtils.getConnection();
            return qr.query(connection, sql, new BeanHandler<T>(clazz), parameters);
        } catch (SQLException e) {
            throw new RuntimeException(e);
            //将编译异常->运行异常 ,抛出
        } finally {
            JDBCUtils.close(null, null, connection);
        }
    }

    // 查询一个值
    public Object queryScalar(String sql, Object... parameters) throws SQLException {
        Connection connection = null;
        try {
            connection = JDBCUtils.getConnection();
            return qr.query(connection, sql, new ScalarHandler(), parameters);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        } finally {
            JDBCUtils.close(null, null, connection);
        }
    }
}
```

```java
class CustomerDao extends BasicDAO<CustomerDao> {
    /**
     * 封装自己的方法，并且由于是BasicDao的子类，可直接使用其定义的方法
     * 由于指定了泛型，也可以直接返回封装好的对象
     */
}
```

## ORM

> 是什么？

​	对象关系映射（Object Relational Mapping，简称ORM）模式是一种`为了解决面向对象与关系数据库存在的互不匹配的现象的技术`。简单的说，ORM是通过使用描述对象和数据库之间映射的元数据，将程序中的对象自动持久化到关系数据库中。

​	也就是说我们能不能不写死SQL，将一个模型类与数据库中一张表做映射关系。我们只面向对象操作，比如`save(new User)`表示保存User对象到数据库中，一般我们会把User类与数据库中User表进行匹配与映射。这样我们也不用**直接**写SQL了。

> ORM的优缺点 

优点：简单，直接，面向对象操作

缺点：会牺牲程序的执行效率和会固定思维模式。 （用多了，SQL可能都不会写了，而且提供的方法是有限的，但你的需求是无限的，我深有体会，这里感谢鱼皮对我的帮助）。

> 常见ORM框架

1. Mybatis 、Mybatis-plus（常用，直接看官方文档操作）
2. Hibernate

​	

​	上面就是Java对数据库操作的演化过程，各个阶段的代码只做了简单的演示，感兴趣可以再深入学习，我现在是常用Mybatis-plus，但由于傻瓜式操作，我更想了解一下底层，所以在想我们能不能自己写一个简单的ORM，不需要很完善，可以表达思想即可，这里我看了一篇文章写的很棒，下面的代码也是基于该作者的。

详情链接：https://cloud.tencent.com/developer/article/2057369?from=article.detail.1803564

​	这里我们先声明一点，不管是什么操作，Mysql都是只认SQL，既然现在我们是面向对象操作，不去自己写SQL，那ORM底层就一定会通过某些操作，将我们的对象操作转写为SQL语句，最后执行。底层实际上还是基础的部分，只不过我们套了个壳子，让使用更方便了而已，这里需要的基本知识：`注解，反射，JDBC基础`。

### 代码演示

> 我们既然要将实体类与数据库表做映射，那我们就应该在类上声明，它对应的哪个表。我们使用@Table 来声明

```java
@Inherited
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
// 加在类上，标识哪张表
public @interface Table {
    String value() default "";
}
```

```java
@Inherited
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
// 加在属性上，标识主键
public @interface PrimaryKey {
    String value() default "";
}
```

> 对应的实体类

```java
// lombok注解，简化代码
@Data
@AllArgsConstructor
@NoArgsConstructor
// 自定义注解，表示对应的user表
@Table("user")
public class User {
    @PrimaryKey
    private Integer id;
    private String username;
    private String gender;
    private String address;
}
```

> 底层还是需要获取连接，这里我们使用Druid连接池

```java
/**
 * 注册驱动
 */
public class MyDataSource {
    protected DataSource dataSource;
    {
        com.alibaba.druid.pool.DruidDataSource dataSource = new com.alibaba.druid.pool.DruidDataSource();
        dataSource.setUrl("jdbc:mysql://1.14.74.242:3306/orm");
        dataSource.setUsername("root");
        dataSource.setPassword("123456");
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        this.dataSource = dataSource;
    }
}
```

> 创建JdbcTemplate，利用DataSource与数据库直接交互，实现通用方法

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

/**
 * 根据资源获取连接，执行SQL，结果集
 */
public abstract class JdbcTemplate extends MyDataSource {

    protected static Connection connection;
    protected static PreparedStatement preparedStatement;
    protected ResultSet resultSet;

    //查询
    protected <T> List<T> executeQuery(String sql , RowMapper<T> rowMapper) throws Exception {
        preparedStatement = preparedStatement(sql);
        resultSet = preparedStatement.executeQuery();
        List<T> list = resultSet(resultSet, rowMapper);
        close();
        return list;
    }

    //更新
    protected int executeUpdate(String sql) throws SQLException {
        preparedStatement = preparedStatement(sql);
        int i = preparedStatement.executeUpdate();
        close();
        return i;
    }

    protected void execute(String sql){
        try {
            preparedStatement = preparedStatement(sql);
            preparedStatement.execute();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            //close();
        }

    }

    //获取链接
    private Connection getConnection() throws SQLException {
        if (connection == null){
            connection = this.dataSource.getConnection();
        }
        return connection;
    }

    //预执行sql
    private PreparedStatement preparedStatement(String sql) throws SQLException {
        connection = getConnection();
        if (preparedStatement == null){
            preparedStatement = connection.prepareStatement(sql);
        }
        return preparedStatement;
    }

    //结果集
    private <T> List<T> resultSet(ResultSet resultSet , RowMapper<T> rowMapper) throws Exception {
        List<T> list = new ArrayList<>();
        while (resultSet.next()){
            list.add(rowMapper.mapRow(resultSet));
        }
        return list;
    }

    //关闭
    private void close(){
        try {
            if (connection != null) {
                connection.close();
            }
            if (preparedStatement != null) {
                preparedStatement.close();
            }
            if (this.resultSet != null) {
                this.resultSet.close();
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

> 其中结果集的映射：RowMapper，这里写的接口，当使用的时候，使用匿名内部类操作

```java
public interface RowMapper<T> {
    T mapRow(ResultSet resultSet) throws Exception;
}
```

> 用户面向对象编程，不写SQL，所以我们底层需要反射获取对象的参数，然后解析拼接为SQL

这里我们将反射对象的通用操作封装

```java
public abstract class BaseSQLBuilder {
    protected String tableName; //表名
    protected String primaryKeyName; //主键名

    protected final String SELECT = "select ";
    protected final String FROM = " from ";
    protected final String WHERE = " where ";
    protected final String AND = " and ";
    protected final String IN = " IN ";
    protected final String UPDATE = " UPDATE ";
    protected final String SET = " SET ";
    protected final String VALUES = " VALUES ";
    protected final String OR = " OR ";
    protected final String DELETE = " DELETE ";
    protected final String INSERT = " INSERT ";
    protected final String INTO = " INTO ";

    protected StringBuilder sqlBuilder = new StringBuilder();

    protected void getTableName(Object obj){
        Table table = obj.getClass().getAnnotation(Table.class);
        tableName = table.value();
        if (Objects.equals(tableName, "")) {
            // 全类名拆分
            tableName = StringUtil.getLastStr(obj.getClass().getName());
        }
    }

    protected void getPrimaryKey(Field field){
        PrimaryKey primaryKey = field.getAnnotation(PrimaryKey.class);
        primaryKeyName = primaryKey.value();
        if (Objects.equals(primaryKeyName, "")) {
            primaryKeyName = StringUtil.getLastStr(field.getClass().getName());
        }
    }

    protected boolean hasPrimaryKey(Field field){
        PrimaryKey primaryKey = field.getAnnotation(PrimaryKey.class);
        return primaryKey != null;
    }

    protected String getField(String fieldStr){
        return "get" + fieldStr.substring(0, 1).toUpperCase() + fieldStr.substring(1);
    }

    protected Field[] getFields(Object obj){
        return obj.getClass().getDeclaredFields();
    }
}
```

这里我们只以查询为例

```java
public class QuerySQLBuilder extends BaseSQLBuilder {
    public String querySql(Object t) throws Exception {
        // 获取操作表名
        getTableName(t);
        // SQL拼接
        sqlBuilder.append(SELECT + "*" + FROM).append(tableName).append(WHERE + " 1=1 ");
        for (Field field : getFields(t)) {
            String fieldStr = StringUtil.getLastStr(field.toString());
            Object value = t.getClass().getMethod(getField(fieldStr)).invoke(t);
            if (!"".equals(value) && null != value) {
                sqlBuilder.append(AND).append(fieldStr).append("=").append("'").append(value).append("'");
            }
        }
        return sqlBuilder.toString();
    }
}
```

> 我们将QuerySQLBuilder。。。其他Builder封装到一个工厂类中，进行获取

```java
public class SQLBuilderInstanceFactory {
    static QuerySQLBuilder queryBuilder = null;
    static SaveSQLBuilder saveBuilder = null;
    static UpdateSQLBuilder updateBuilder = null;
    static DeleteSQLBuilder deleteSQLBuilder = null;

    public static QuerySQLBuilder getQueryBuilder(){
        if (queryBuilder == null) {
            queryBuilder = new QuerySQLBuilder();
        }
        return queryBuilder;
    }

    public static SaveSQLBuilder getSaveBuilder(){
        if (saveBuilder == null) {
            saveBuilder = new SaveSQLBuilder();
        }
        return saveBuilder;
    }

    public static UpdateSQLBuilder getUpdateBuilder(){
        if (updateBuilder == null) {
            updateBuilder = new UpdateSQLBuilder();
        }
        return updateBuilder;
    }

    public static DeleteSQLBuilder getDeleteSQLBuilder(){
        if (deleteSQLBuilder == null) {
            deleteSQLBuilder = new DeleteSQLBuilder();
        }
        return deleteSQLBuilder;
    }
}
```

> 定义查询接口

```java
public interface IQuery<T> {
    List<T> query(T t) throws Exception;
}
```

> 查询具体实现类

```java
public class Query<T> extends JdbcTemplate implements IQuery<T> {
    @Override
    public List<T> query(T t) throws Exception {
        // 解析出的SQL
        String sql = SQLBuilderInstanceFactory.getQueryBuilder().querySql(t);
        System.out.println("sql  "+sql);
        Field[] fields = t.getClass().getDeclaredFields();
        // 执行SQL
        return executeQuery(sql, new RowMapper<T>() {
            @Override
            public T mapRow(ResultSet resultSet) throws Exception {
                for (Field field : fields) {
                    String getField = StringUtil.getSetMethod(StringUtil.getLastStr(field.toString()));
                    Object object = resultSet.getObject(StringUtil.getLastStr(field.toString()), field.getType());
                    t.getClass().getMethod(getField,field.getType()).invoke(t,object);
                }
                return t;
            }
        });
    }
}
```

现在大体上的代码都已经实现了，我们测试一下

```java
public class Client {
    public static void main(String[] args) throws Exception {
        User user = new User();
        user.setAddress("china");
        // 查询操作
        Query<User> userQuery = new Query<>();
        List<User> userList = userQuery.query(user);
    }
}
```

可以看到这期间用户在使用的过程中，没有写一句SQL，但可以正常操作数据库了。剩下的细节就等着我们去完善。



## 总结

​	本文中，我大概总结了一下，Java操作Mysql的演化过程，从最基本的JDBC到自动化的ORM操作，可以看到操作是越来越简单，也越来越灵活了，但底层大致的原理我们还是应该了解的。

​	本文我参考了韩顺平老师的Mysql基础文章和小四的技术之旅文章。十分感谢！！！原作者的文章中的内容更加详细，具体可在网上查询。

​	这里想推一下我的博客，虽然现在还很简陋，里面会分享我的学习所得，也希望能帮助到大家，http://1.14.74.242
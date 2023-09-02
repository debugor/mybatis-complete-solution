要说起mybatis必须了解传统JDBC编程流程。
## 1.1 JDBC编程
### 1.1.1 简介
JDBC，即 Java Database Connectivity，即 Java 数据库连接。是一种用于执行 SQL 语句的 Java API，它是 Java中的数据库连接规范。这个 API 为 Java 开发人员操作数据库提供了一个标准，可以为多种关系数据库提供统一访问。
### 1.1.2 编程步骤
第一步：注册驱动（告诉Java程序，即将要连接那个品牌的数据库）
第二步：获取连接（表示JVM的进程和数据库进程之间的通道打开了，这属于进程之间的通信，重量级的，使用完后一定要关闭）
第三步：获取数据库操作对象（专门执行sql语句的对象）
第四步：执行SQL语句（DQL DML ...)
第五步：处理查询结果集（只有第四步执行select语句时，才会执行第五步）
第六步：释放资源（使用完资源后一定要关闭资源。Java和数据库属于进程间通信，开启后一定要关闭）

需求：有一张user表，请使用传统JDBC编程方式将所有数据映射到集合List<User>中。



```java
// Student.java
@Data
@AllArgsConstructor
public class Student {
    private Long id;
    private String name;
    private Integer age;
}


// JDBCTest.java
public class JDBCTest {

    public static void main(String[] args) {
        Connection conn = null;
        Statement statement = null;
        ResultSet resultSet = null;
        try {
            // 1.注册驱动
            registerDriver();
            // 2.获取连接
            conn = getConnection();
            // 3.获取数据库操作对象
            statement = getStatement(conn);
            // 4. 执行SQL语句
            resultSet = executeSQL(statement);
            // 5. 处理结果集
            final List<Student> studentList = handleResultSet(resultSet);
        } catch (SQLException | ClassNotFoundException ex) {
            ex.printStackTrace();
        } finally {
            // 6.释放资源
            release(conn, statement, resultSet);
        }
    }

    private static void release(Connection conn, Statement statement, ResultSet resultSet) {
        try {
            if (resultSet != null && !resultSet.isClosed()) {
                resultSet.close();
            }
        } catch (SQLException ex) {
            ex.printStackTrace();
        }
        try {
            if (statement != null && !statement.isClosed()) {
                statement.close();
            }
        } catch (SQLException ex) {
            ex.printStackTrace();
        }
        try {
            if (conn != null && !conn.isClosed()) {
                conn.close();
            }
        } catch (SQLException ex) {
            ex.printStackTrace();
        }
    }


    private static List<Student> handleResultSet(final ResultSet resultSet) throws SQLException {
        final List<Student> studentList = new ArrayList<Student>();
        while (resultSet.next()) {
            final Long id = resultSet.getLong("id");
            final String name = resultSet.getString("name");
            final Integer age = resultSet.getInt("age");
            studentList.add(new Student(id, name, age));
        }
        return studentList;
    }

    private static ResultSet executeSQL(final Statement statement) throws SQLException {
        return statement.executeQuery("select id, name ,age from student");
    }

    private static Statement getStatement(final Connection conn) throws SQLException {
        return conn.createStatement();
    }

    private static Connection getConnection() throws SQLException {
        final String url = "mysql.url";
        final String user = "mysql.user";
        final String password = "mysql.password";
        return DriverManager.getConnection(url, user, password);
    }

    private static void registerDriver() throws ClassNotFoundException {
        Class.forName("com.mysql.jdbc.Driver");
    }

}


```

### 1.1.3 缺点

1. 数据库连接创建、释放频繁造成系统资源浪费从而影响系统性能。
2. sql语句在代码中硬编译，造成代码不易维护，实际应用sql变化的可能较大，sql变动需要改变Java代码。
3. 查询操作时，需要手动将结果集中的数据手动封装到实体中；插入操作时，需要手动将实体的数据设置到sql语句的占位符位置。

### 1.1.4 如果解决

1. 使用数据库连接池技术（C3P0或者Druid）初始化连接资源；
2. 将sql语句抽取到xml配置文件中（解耦合）；
3. 使用反射、内省等底层技术，自动将实体与表进行属性与字段的自动映射（比较困难）。

## 1.2 mybatis
MyBatis 本是apache的一个开源项目iBatis，2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。2013年11月迁移到Github。
MyBatis是一个优秀的持久层框架，它对jdbc的操作数据库的过程进行封装，使开发者只需要关注SQL本身，而不需要花费精力去处理例如注册驱动、创建connection、创建statement、手动设置参数、结果集检索等jdbc繁杂的过程代码。
Mybatis通过xml或注解的方式将要执行的各种statement（statement、preparedStatemnt）配置起来，并通过java对象和statement中的sql进行映射生成最终执行的sql语句，最后由mybatis框架执行sql并将结果映射成java对象并返回。

### 1.2.1 依赖引入
```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.8</version> <!-- 你可以根据需要选择不同版本 -->
</dependency>
```
### 1.2.2 映射描述文件
StudentMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--名称空间-->
<mapper namespace="com.learner.mybatis.mapper.StudentMapper">
  <!--  查询语句  -->
  <select id="getById" resultType="com.learner.mybatis.entity.Student">
    select * from student where id = #{id}
  </select>
</mapper>

```
### 1.2.3 配置文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

  <!--数据源环境-->
  <environments default="developement">
    <environment id="developement">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis_learner"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
      </dataSource>
    </environment>
  </environments>
  <!--加载映射文件-->
  <mappers>
    <mapper resource="mapper/StudentMapper.xml"/>
  </mappers>
</configuration>
```

### 1.2.4 mybatis执行
```java
public class MybatisTest {

    public static void main(String[] args) {
        SqlSession sqlSession = null;
        try {
            // 获取sqlSession
            sqlSession = getSqlSession();
            // 通过namespace+id找到要执行的sql语句并执行sql语句
            // com.learner.mybatis.mapper.StudentMapper对应namespace, getById对应语句id
            final Student student= sqlSession.selectOne("com.learner.mybatis.mapper.StudentMapper.getById", 2 );
            System.out.println(JSON.toJSONString(student));
        } catch (Exception ex) {
            ex.printStackTrace();
        }finally {
            if(sqlSession != null){
                sqlSession.close();
            }
        }
    }

    private static SqlSession getSqlSession() throws IOException {
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader("mybatis_config.xml"));
        return sqlSessionFactory.openSession();
    }
}
```
这里有两个xml文件和一段代码，如果你刚学mybatis会觉得有点懵，这里画了一个简单的流程图，方便大家理解。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/1482831/1692029990965-e1a3e046-13a9-485f-9db8-c579cc4d6a07.png#averageHue=%23fbfbfb&clientId=uc916fe2d-40a6-4&from=paste&height=483&id=u3f8dbc8d&originHeight=966&originWidth=2100&originalType=binary&ratio=2&rotation=0&showTitle=false&size=197706&status=done&style=none&taskId=ub2b88694-461e-4317-a07a-741f977c023&title=&width=1050)
由于mybatis大部分信息都是通过配置文件描述的，所以我们先从mybatis_config.xml这个入口配置文件说起。
在这个文件中会描述mybatis运行时的配置信息，如数据库连接，属性，映射文件列表（在本例子中只有一个mapper/StudentMapper.xml）等。mybatis映射文件是最重要的文件，没有之一。这里我们只是用最简单的一个例子描述它的基础功能，

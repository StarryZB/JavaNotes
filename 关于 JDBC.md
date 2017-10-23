# 关于 JDBC

JDBC（Java Data Base Connectivity,java 数据库连接）是一种用于执行 SQL 语句的 Java API，可以为多种关系数据库提供统一访问，它由一组用 Java 语言编写的类和接口组成。JDBC 提供了一种基准，据此可以构建更高级的工具和接口，使数据库开发人员能够编写数据库应用程序。

## 常用接口

1. Driver接口

　　Driver接口由数据库厂家提供，作为开发人员，只需要使用 Driver 接口就可以了。在编程中要连接数据库，必须先装载特定厂商的数据库驱动程序，不同的数据库有不同的装载方法。

如：

　　装载 MySql 驱动：`Class.forName("com.mysql.jdbc.Driver");`

　　装载 Oracle 驱动：`Class.forName("oracle.jdbc.driver.OracleDriver");`

2. Connection接口

　　Connection 与特定数据库的连接（会话），在连接上下文中执行 sql 语句并返回结果。`DriverManager.getConnection(url, user, password);`方法建立在 JDBC URL中定义的数据库 Connection 连接上。

　　连接MySql数据库：`Connection conn = DriverManager.getConnection("jdbc:mysql://host:port/database", "user", "password");`

　　连接Oracle数据库：`Connection conn = DriverManager.getConnection("jdbc:oracle:thin:@host:port:database", "user", "password");`

　　连接SqlServer数据库：`Connection conn = DriverManager.getConnection("jdbc:microsoft:sqlserver://host:port; DatabaseName=database", "user", "password");`

　　常用方法：

- createStatement()：创建向数据库发送 sql 的 statement 对象。

- prepareStatement(sql) ：创建向数据库发送预编译 sql 的 PrepareSatement 对象。
- prepareCall(sql)：创建执行存储过程的 callableStatement 对象。
- setAutoCommit(boolean autoCommit)：设置事务是否自动提交。
- commit() ：在链接上提交事务。
- rollback() ：在此链接上回滚事务。

3. Statement 接口

　　用于执行静态 SQL 语句并返回它所生成结果的对象。

　　三种Statement类：

- Statement：由 createStatement 创建，用于发送简单的 SQL 语句（不带参数）。

- PreparedStatement ：继承自 Statement 接口，由 preparedStatement 创建，用于发送含有一个或多个参数的 SQL 语句。PreparedStatement 对象比 Statement 对象的效率更高，并且可以防止 SQL 注入，所以我们一般都使用 PreparedStatement。
- CallableStatement：继承自 PreparedStatement 接口，由方法 prepareCall 创建，用于调用存储过程。

　　常用Statement方法：

- execute(String sql):运行语句，返回是否有结果集

- executeQuery(String sql)：运行 select 语句，返回 ResultSet 结果集。
- executeUpdate(String sql)：运行 insert/update/delete 操作，返回更新的行数。
- addBatch(String sql) ：把多条 sql 语句放到一个批处理中。
- executeBatch()：向数据库发送一批 sql 语句执行。

4. ResultSet接口

ResultSet提供检索不同类型字段的方法，常用的有：

- getString(int index)、getString(String columnName)：获得在数据库里是 varchar、char 等类型的数据对象。
- getFloat(int index)、getFloat(String columnName)：获得在数据库里是 Float类型的数据对象。
- getDate(int index)、getDate(String columnName)：获得在数据库里是Date 类型的数据。
- getBoolean(int index)、getBoolean(String columnName)：获得在数据库里是 Boolean 类型的数据。
- getObject(int index)、getObject(String columnName)：获取在数据库里任意类型的数据。

　　ResultSet 还提供了对结果集进行滚动的方法：

- next()：移动到下一行
- Previous()：移动到前一行
- absolute(int row)：移动到指定行
- beforeFirst()：移动 resultSet 的最前面。
- afterLast() ：移动到 resultSet 的最后面。

使用后依次关闭对象及连接：ResultSet → Statement → Connection

## 使用 JDBC 的步骤

加载 JDBC 驱动程序 → 建立数据库连接 Connection → 创建执行 SQL 的语句Statement → 处理执行结果 ResultSet → 释放资源

1. 注册驱动 (只做一次)

方式一：`Class.forName(“com.MySQL.jdbc.Driver”);`
　　推荐这种方式，不会对具体的驱动类产生依赖。
方式二：`DriverManager.registerDriver(com.mysql.jdbc.Driver);`
　　会造成 DriverManager 中产生两个一样的驱动，并会对具体的驱动类产生依赖。

2. 建立连接

```java
　Connection conn = DriverManager.getConnection(url, user, password); 
```

　　URL 用于标识数据库的位置，通过 URL 地址告诉 JDBC 程序连接哪个数据库， URL 的写法为：

jdbc：mysql://localhost:3306/test ?参数名：参数值

其他参数如：useUnicode=true&characterEncoding=utf8

3. 创建执行 SQL 语句的 statement

```java
Statement st = conn.createStatement();
st.executeQuery(sql);
//存在sql注入的危险
//如果用户传入的id为“5 or 1=1”，那么将删除表中的所有记录
```

```java
String sql = “insert into user (name,pwd) values(?,?)”;  
PreparedStatement ps = conn.preparedStatement(sql);  
ps.setString(1, "col_value");  //占位符顺序从1开始
ps.setString(2, "123456"); //也可以使用setObject
ps.executeQuery(); 
//PreparedStatement 有效的防止sql注入(SQL语句在程序运行前已经进行了预编译,当运行时动态地把参数传给PreprareStatement时，即使参数里有敏感字符如 or '1=1'也数据库会作为一个参数一个字段的属性值来处理而不会作为一个SQL指令
```

4. 处理执行结果 ResultSet

```java
While(rs.next()){  
     rs.getString("name");  
     rs.getInt("id"); 
}
```

5. 释放资源

```java
//都要加try catch 以防前面关闭出错，后面的就不执行了
try {
 2     if (rs != null) {
 3         rs.close();
 4     }
 5 } catch (SQLException e) {
 6     e.printStackTrace();
 7 } finally {
 8     try {
 9         if (st != null) {
10             st.close();
11         }
12     } catch (SQLException e) {
13         e.printStackTrace();
14     } finally {
15         try {
16             if (conn != null) {
17                 conn.close();
18             }
19         } catch (SQLException e) {
20             e.printStackTrace();
21         }
22     }
23 }
```

示例：

```java
import java.sql.*;//第一步，导包
public class DBHelper {
    private static final String driver = "com.mysql.jdbc.Driver"; //数据库驱动
    private static final String url="jdbc:mysql://localhost:3306/cart";//连接数据库的URL地址
    private static final String username="root";//数据库的用户名
    private static final String password="123456";//数据库的密码
  
    public static Connection getConn() {
        Connection conn = null;
        try {
            Class.forName(driver);//第二步，注册驱动器
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
            System.out.println("找不到驱动类");
        }
        try {
            conn = DriverManager.getConnection(url,username,password);//第三步，打开连接
        } catch (SQLException e) {
            e.printStackTrace();
            System.out.println("连接失败");
        }
        return conn;
    }
    public static Statement createStmt(Connection conn) {
        Statement stmt = null;
        try {
            stmt = conn.createStatement();//第四步，生成Statement用于执行静态SQL语句并返回它所生成结果的对象
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return stmt;
    }
    public static ResultSet executeQuery(Statement stmt,String sql) {
        ResultSet rs = null;
        try {
            rs = stmt.executeQuery(sql);//运行select语句，返回ResultSet结果集
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return rs;
    }
    public static void closers(ResultSet rs) {
        if (rs != null) {
            try {
                rs.close();//释放ResultSet
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    public static void closestmt(Statement stmt) {
        if(stmt != null) {
            try {
                stmt.close();//释放Statement
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    public static void closeconn(Connection conn) {
        if(conn != null) {
            try {
                conn.close();//关闭Connection
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```


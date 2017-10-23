# 关于 JDBC

JDBC（Java Data Base Connectivity,java 数据库连接）是一种用于执行 SQL 语句的 Java API，可以为多种关系数据库提供统一访问，它由一组用 Java 语言编写的类和接口组成。JDBC 提供了一种基准，据此可以构建更高级的工具和接口，使数据库开发人员能够编写数据库应用程序。

##常用接口

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

##使用 JDBC 的步骤

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

　　![img](data:image/png;base64,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)

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

3. 处理执行结果 ResultSet

```java
While(rs.next()){  
     rs.getString("name");  
     rs.getInt("id"); 
}
```

3. 释放资源

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


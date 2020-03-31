# JDBC：

1. 111111概念：Java DataBase Connectivity  Java数据库连接，Java语言操作数据库

   - JDBC本质：其实是官方定义的一套操作所有关系型数据库的规则，即接口。各个数据库厂商去实现这套接口，提供数据库驱动jar包。我们可以使用这套接口（JDBC）编程，真正执行的代码是驱动jar包中的实现类。

2. 快速入门：

   - 步骤：

     1. 导入驱动jar包    mysql-connector-java-5.1.7-bin.jar
        1. 复制mysql-connector-java-5.1.7-bin.jar到项目的libs目录下
        2. 右键-->Add As  Library
     2. 注册驱动
     3. 获取数据库连接对象  Connection
     4. 定义sql
     5. 获取执行sql语句的对象   Statement
     6. 执行sql，接受返回结果
     7. 处理结果
     8. 释放资源

   - 代码实现：

     ```java
     package JDBC;
     
     /*  JDBC快速入门    */
     
     import java.sql.Connection;
     import java.sql.DriverManager;
     import java.sql.Statement;
     
     public class jdbcDemo01 {
         public static void main(String[] args) throws Exception {
     
             //1.导入驱动jar包
             //2.注册驱动
             Class.forName("com.mysql.jdbc.Driver");
             //3.获取数据库连接对象
      		Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3366/xinzhi", "root", "root");
             //4.定义sql语句
             String sql = "UPDATE tb_emp6 SET salary = 222 where id = 1";
             //5.获取在执行sql的对象 Statement
             Statement stmt = conn.createStatement();
             //6.执行sql
             int count = stmt.executeUpdate(sql);
             //7.执行结果
             System.out.println(count);
             //8.释放资源
             stmt.close();
             conn.close();
     
         }
     }
     
     ```

3. 详解各个对象：

   1. DriverManager：驱动管理对象

      - 功能：

        1. 注册驱动：告诉程序该使用哪一个数据库驱动jar

           ```java
           static void registerDriver(Driver driver):注册与给定的驱动程序 DriverManager
               
           //写代码使用：Class.forName("com.mysql.jdbc.Driver");
           
           //通过查看源码发现：在com.mysql.jdbc.Driver类中存在静态代码块
           static {
           	try{
                   java.sql.DriverManager.registerDriver(new Driver());
               } catch (SQLException E){
                   throw new RuntimeException("Can't register driver!");
               }
           }
           
           //注意mysql5之后的驱动jar包可以省略注册驱动的步骤。
           ```

           

        2. 获取数据库连接：

           - 方法：

             ```java
             static Connection geiConnection(String url,String user,String password)
             ```

           - 参数：

             - url：指定连接的路径

               - 语法：

                 ```java
                 jdbc:mysql://IP地址(域名):端口号/数据库名称
                 ```

               - 例子：

                 ```java
                 jdbc:mysql://localhost:3366/xinzhi
                 ```

               - 细节：如果连接的时本机mysql服务器，并且mysql服务默认端口是3306，则url可以简写为：

                 ```java
                 jdbc:mysql:///数据库名称
                 ```

                 

             - user：用户名

             - password：密码

   2. Connection：数据库连接对象

      1. 功能：

         1. 获取执行sql的对象

            ```java
            //第一种方法
            createStatement()
            //第二种方法
            PreparedStatement preparedStatement(String sql)
            ```

         2. 管理事务：

            1. 开启事务：

               ```java
               //调用该方法设置参数为false，即开启事务
               void setAutoCommit(boolean autoCommit)
               ```

               

            2. 提交事务：

               ```java
               commit()
               ```

               

            3. 回滚事务：

               ```java
               rollback()
               ```

               

   3. Statement：执行sql的对象

      1. 执行sql

         1. boolean execute(String sql)：可以执行任意的sql（仅了解，不常用）
         2. int executeUpdate(String sql)：执行DML（insert、update、delete）语句、DDL(create、alter、drop)语句
            - 返回值：影响的行数，可以通过这个影响的行数判断DML语句是否执行成功 返回值 > 0则执行成功，反之，则失败。
         3. ResultSet executeQuery(String sql)：执行DQL（select）语句

      2. 练习：

         1. t_user表 添加一条记录

            ```java
            package JDBC;
            
            import java.sql.Connection;
            import java.sql.DriverManager;
            import java.sql.SQLException;
            import java.sql.Statement;
            
            /*
            t_user表 添加一条记录 insert语句
            */
            public class Demo02 {
                public static void main(String[] args) {
                    Statement stmt = null;
                    Connection conn = null;
            
                    try {
                        //1.注册驱动
                        Class.forName("com.mysql.jdbc.Driver");
                        //2.定义sql
                        String sql = "insert into t_user values('null','马','123','山西',12.22)";
                        //3.获取Connection对象
                        conn = DriverManager.getConnection("jdbc:mysql://localhost:3366/xinzhi", "root", "root");
                        //4.获取执行sql的对象 Statement
                        stmt = conn.createStatement();
                        //5.执行sql
                        int count = stmt.executeUpdate(sql);//影响的行数
                        //6.处理结果
                        System.out.println(count);
                        if (count > 0){
                            System.out.println("添加成功！");
                        }else {
                            System.out.println("添加失败！");
                        }
                        //
                    } catch (ClassNotFoundException e) {
                        e.printStackTrace();
                    } catch (SQLException e) {
                        e.printStackTrace();
                    }finally {
                        //7.释放资源
                        //避免空指针异常
                        if (stmt != null){
                            try {
                                stmt.close();
                            } catch (SQLException e) {
                                e.printStackTrace();
                            }
                        }
            
                        if (conn != null){
                            try {
                                conn.close();
                            } catch (SQLException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                }
            }
            
            ```

            

         2. t_user表 修改记录：同第一条，只要改改语句即可

         3. t_user表 删除一条记录：同第一条，只要改改语句即可

   4. ResultSet：结果集对象

      - boolean next()：游标向下移动一行，判断当前行是否是最后一行末尾（是否有数据），如果是，则返回数据，如果不是，则返回false。

      - getXxx()：获取数据

        - Xxx：数据类型  如：int getInt()  String getString()
        - 参数：
          1. Int：代表列的编号。从1开始   如：getString(1)
          2. String：代表列的名称。  如：getDouble("price")

      - 注意：

        - 使用步骤：
          1. 游标向下移动一行
          2. 判断是否有数据
          3. 获取数据

      - 代码展示

        ```java
        package JDBC;
        
        import java.sql.*;
        
        public class Demo05 {
            public static void main(String[] args) {
                Statement stmt = null;
                Connection conn = null;
                ResultSet rs = null;
                try {
                    //1.注册驱动
                    Class.forName("com.mysql.jdbc.Driver");
                    //2.定义sql
                    String sql = "select * from t_user";
                    //3.获取Connection对象
                    conn = DriverManager.getConnection("jdbc:mysql://localhost:3366/xinzhi", "root", "root");
                    //4.获取执行sql的对象 Statement
                    stmt = conn.createStatement();
                    //5.执行sql
                    rs = stmt.executeQuery(sql);
                    //6.处理结果
                    //循环判断结果集是否是最后一行某尾
                    while (rs.next()){
                        //6.2 获取数据
                        String id = rs.getString("id");
                        String userName = rs.getString("userName");
                        String password = rs.getString("password");
                        String address = rs.getString("address");
                        double price = rs.getDouble(5);
                        System.out.println(id + "---" + userName + "---" + password + "---" + address + "---" + price);
                    }
                } catch (ClassNotFoundException e) {
                    e.printStackTrace();
                } catch (SQLException e) {
                    e.printStackTrace();
                }finally {
                    //7.释放资源
                    //避免空指针异常
                    if (rs != null){
                        try {
                            rs.close();
                        } catch (SQLException e) {
                            e.printStackTrace();
                        }
                    }
                    if (stmt != null){
                        try {
                            stmt.close();
                        } catch (SQLException e) {
                            e.printStackTrace();
                        }
                    }
        
                    if (conn != null){
                        try {
                            conn.close();
                        } catch (SQLException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
        
        ```

      - 练习：

        - 定义一个方法，查询emp表的数据将其封装为对象，然后装载集合，返回。
          - 定义emp类
          - 定义方法public List<emp>

   5. PreparedStatement：执行sql的对象
   
      1. SQL注入问题：在拼接sql时，有一些sql的特殊关键字参与字符串的拼接。会造成安全性问题。
   
         1. 输入用户随便，输入密码：a'  or  'a'  =  'a
         2. sql: select * from user where username = 'xxxxxxx'  and password = 'a'  or 'a'  = 'a'
   
      2. 解决sql注入问题：使用PreparedStatement对象来解决
   
      3. 预编译的SQL：参数使用？作为占位符
   
      4. 步骤：
   
         1. 导入驱动jar包    mysql-connector-java-5.1.7-bin.jar
   
         2. 注册驱动
   
         3. 获取数据库连接对象  Connection
   
         4. 定义sql
   
            - 注意：sql的参数使用？作为占位符。如：
   
              ```mysql
              select * from user where username = ?  and password = ?;
              ```
   
              
   
         5. 获取执行sql语句的对象   PreparedStatement  Connection.prepareStatement(String sql)
   
         6. 给？赋值
   
            - 方法：setXxx（参数1，参数2）
              - 参数1：？的位置编号，从1开始
              - 参数2：？的值
   
         7. 执行sql，接受返回结果
   
         8. 处理结果
   
         9. 释放资源
   
      5. 注意：后期都会使用PreparedStatement来完成增删改查的所有操作
   
         1. 可以防止SQL注入
         2. 效率更高
   
      6. 解决SQL注入后的代码
   
         ```java
         package JDBC;
         
         import JDBC.util.JDBCUtils;
         
         import java.sql.*;
         import java.util.Scanner;
         
         public class Demo08 {
         
             public static void main(String[] args) {
                 //1.键盘录入，接受用户名和密码
                 Scanner sc = new Scanner(System.in);
                 System.out.println("请输入用户名：");
                 String username = sc.nextLine();
                 System.out.println("请输入密码：");
                 String password = sc.nextLine();
                 //2.调用方法
                 boolean flag = new Demo08().login(username, password);
                 //3.判断结果，输出不同语句
                 if (flag){
                     //登录成功
                     System.out.println("登录成功！");
                 }else{
                     System.out.println("用户名或密码错误！");
                 }
         
             }
         
         
             //登录方法，使用PreparedStatement实现
             public boolean login(String username, String password) {
                 if (username == null || password == null){
                     return false;
                 }
                 Connection conn = null;
                 PreparedStatement pstmt = null;
                 ResultSet rs = null;
         
                 //连接数据判断是否登录成功
         
         
                 try {
                     //1.获取连接
                     conn = JDBCUtils.getConnection();
                     //2.定义sql
                     String sql = "select * from user where username =  ?  and password = ? ";
                     //3.获取执行sql的对象
                     pstmt = conn.prepareStatement(sql);
                     //给？赋值
                     pstmt.setString(1,username);
                     pstmt.setString(2,password);
                     //4.执行查询,不需要传递sql
                     rs = pstmt.executeQuery();
                     //5.判断，如果有下一行，则返回true
                     return rs.next();
         
                 } catch (SQLException e) {
                     e.printStackTrace();
                 }finally {
                     JDBCUtils.close(rs,pstmt,conn);
                 }
         
                 return false;
             }
         }
         
         ```
   
         

## 抽取JDBC工具类：JDBCUtils

- 目的：简化书写

- 分析：

  1. 注册驱动也抽取

  2. 抽取一个方法获取连接对象

     - 需求：不想传递参数（麻烦），还得保证工具类的通用性。

     - 解决：配置文件

       ```
       jdbc.properties文件
       
       //文件里写入数据，例如：
       url=jdbc:mysql://localhost:3366/db
       user=root
       password=root
       driver=com.mysql.jdbc.Driver
       ```

       

  3. 抽取一个方法

- 工具类代码

  ```java
  package JDBC.util;
  
  import java.io.FileReader;
  import java.io.IOException;
  import java.net.URL;
  import java.sql.*;
  import java.util.Properties;
  
  public class JDBCUtils {
  
      private static String url;
      private static String user;
      private static String password;
      private static String driver;
      //文件的读取，只需要读取一次即可拿到这些值。使用静态代码块
  
      static {
          //读取资源文件，获取值。
  
          //1. 创建Properties集合类
          Properties pro = new Properties();
  
          //获取src路径下的文件的方式-->ClassLoader 类加载器
          ClassLoader classLoader = JDBCUtils.class.getClassLoader();
          URL res = classLoader.getResource("jdbc.properties");
          String path = res.getPath();
          //2. 加载文件
          try {
              pro.load(new FileReader(path));
          } catch (IOException e) {
              e.printStackTrace();
          }
          //3. 获取数据，赋值
          url = pro.getProperty("url");
          user = pro.getProperty("user");
          password = pro.getProperty("password");
          driver = pro.getProperty("driver");
      }
  
      //获取连接
      public static Connection getConnection() throws SQLException {
  
          return DriverManager.getConnection(url,user,password);
      }
      //释放资源
      public static void close(Statement stmt, Connection conn) {
  
          if (stmt != null) {
              try {
                  stmt.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
  
          if (conn != null) {
              try {
                  conn.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
      }
      //重载形式的释放资源
      public static void close(ResultSet rs,Statement stmt, Connection conn) {
          if (rs != null){
              try {
                  rs.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
  
          if (stmt != null) {
              try {
                  stmt.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
  
          if (conn != null) {
              try {
                  conn.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
      }
  }
  
  ```

- 使用JDBC工具类后的代码演示，代码量明显减少

  ```java
  package JDBC;
  
  import JDBC.util.JDBCUtils;
  
  import java.sql.Connection;
  import java.sql.ResultSet;
  import java.sql.SQLException;
  import java.sql.Statement;
  
  public class Demo06 {
      public static void main(String[] args) {
          Statement stmt = null;
          Connection conn = null;
          ResultSet rs = null;
          try {
              conn = JDBCUtils.getConnection();
              String sql = "select * from t_user";
              stmt = conn.createStatement();
              rs = stmt.executeQuery(sql);
  
              while (rs.next()){
                  //6.2 获取数据
                  String id = rs.getString("id");
                  String userName = rs.getString("userName");
                  String password = rs.getString("password");
                  String address = rs.getString("address");
                  double price = rs.getDouble(5);
                  System.out.println(id + "---" + userName + "---" + password + "---" + address + "---" + price);
              }
          } catch (SQLException e) {
              e.printStackTrace();
          }finally {
              JDBCUtils.close(rs,stmt,conn);
          }
      }
  }
  
  ```

- 练习：

  - 需求：

    1. 通过键盘判断录入用户名和密码

    2. 判断用户是否登陆成功

        

  - 步骤：

    1. 创建数据库表，user表

       ```mysql
       -- 创建user表
       create table user(
       	id int primary key auto_increment,
       	username varchar(32),
       	password varchar(32)
       );
       
       -- 插入数据
       insert into user values
       (null,'zhangsan','123'),
       (null,'lisi','234');
       ```

    2. java代码展示

       ```java
       package JDBC;
       
       import JDBC.util.JDBCUtils;
       
       import java.sql.Connection;
       import java.sql.ResultSet;
       import java.sql.SQLException;
       import java.sql.Statement;
       import java.util.Scanner;
       
       /*
       * 练习：
       *   需求：
       *       1.通过键盘录入用户名和密码
       *       2.判断用户是否登陆成功
       * */
       public class Demo07 {
           public static void main(String[] args) {
               //1.键盘录入，接受用户名和密码
               Scanner sc = new Scanner(System.in);
               System.out.println("请输入用户名：");
               String username = sc.nextLine();
               System.out.println("请输入密码：");
               String password = sc.nextLine();
               //2.调用方法
               boolean flag = new Demo07().login(username, password);
               //3.判断结果，输出不同语句
               if (flag){
                   //登录成功
                   System.out.println("登录成功！");
               }else{
                   System.out.println("用户名或密码错误！");
               }
       
           }
       
           //登陆方法
           public boolean login(String username, String password) {
               if (username == null || password == null){
                   return false;
               }
               Connection conn = null;
               Statement stmt = null;
               ResultSet rs = null;
       
               //连接数据判断是否登录成功
       
       
               try {
                   //1.获取连接
                   conn = JDBCUtils.getConnection();
                   //2.定义sql
                   String sql = "select * from user where username = '"+ username +"' and password = '"+ password +"' ";
                   //3.获取执行sql的对象
                   stmt = conn.createStatement();
                   //4.执行查询
                   rs = stmt.executeQuery(sql);
                   //5.判断
                   return rs.next();
       
               } catch (SQLException e) {
                   e.printStackTrace();
               }finally {
                   JDBCUtils.close(rs,stmt,conn);
               }
       
               return false;
           }
       }
       
       ```




## JDBC控制事务：

1. ​	事务：一个包含多个步骤的业务操作。如果这个业务操作被事务管理，则这多个步骤要么同时成功，要么同时失败。

2. 操作：

   1. 开启事务
   2. 提交事务
   3. 回滚事务

3. 使用Connection对象来管理事务

   - 开启事务：setAutoCommit(boolean autoCommit) : 调用该方法设置参数为false，即开启事务。
     - 在执行sql之前开启事务
   - 提交事务：commit();
     - 当所有sql都执行完提交事务
   - 回滚事务：rollback();
     - 在catch中回滚事务

4. 代码展示效果

   ```java
   package JDBC;
   
   import JDBC.util.JDBCUtils;
   
   import java.sql.*;
   
   /**
    * 事务操作
    */
   
   public class Demo09 {
   
       public static void main(String[] args) {
           Connection conn = null;
           PreparedStatement pstmt1 = null;
           PreparedStatement pstmt2 = null;
   
           try {
               //1.获取连接
               conn = JDBCUtils.getConnection();
               //开启事务
               conn.setAutoCommit(false);
   
               //2.定义sql
               //2.1 张三 - 500
               String sql1 = "update account set balance = balance - ? where id = ?";
               //2.2 李四 + 500
               String sql2 = "update account set balance = balance + ? where id = ?";
               //3. 获取执行sql对象
               pstmt1 = conn.prepareStatement(sql1);
               pstmt2 = conn.prepareStatement(sql2);
               //4. 设置参数
               pstmt1.setDouble(1,500);
               pstmt1.setInt(2,1);
   
               pstmt2.setDouble(1,500);
               pstmt2.setInt(2,2);
   
               //5. 执行sql
               pstmt1.executeUpdate();
               pstmt2.executeUpdate();
               //提交事务 
               conn.commit();
           } catch (Exception e) {
               //事务回滚
               try {
                   if (conn != null){
                       conn.rollback();
                   }
               } catch (SQLException ex) {
                   ex.printStackTrace();
               }
               e.printStackTrace();
   
           }finally {
               JDBCUtils.close(pstmt1,conn);
               JDBCUtils.close(pstmt2,null);
           }
       }
   }
   
   ```



## 数据库连接池

- 概念：其实就是一个容器（集合），存放数据库连接的容器。
  - 当系统初始化好后，容器被创建，容器中会申请一些连接对象，当用户来访问数据库时，从容器中获取连接对象，用户访问完之后，会将连接对象归还给容器。
- 好处：
  1. 节约资源
  2. 用户访问高效
- 实现：
  1. 标准接口：DataSource      javax.sql包下的
     1. 方法：
        - 获取连接：getConnection()
        - 归还连接：如果连接对象Connection是从连接池中获取的，那么调用Connection.close()方法，则不会再关闭连接了。而是归还连接。
  2. 一般我们不去实现它，有数据库厂商来实现
     1. C3P0：数据库连接池技术
     2. Druid：数据库连接池实现技术，由阿里巴巴提供的
- C3P0：数据库连接池技术
  - 步骤：
    1. 导入jar包 （两个）c3p0-0.9.2.1.jar 和  mchange-commons-java-0.2.3.4.jar
    2. 定义配置文件：
       - 名称：c3p0.properties   或者  c3p0-config.xml
       - 路径：直接将文件放在src目录下即可。
    3. 创建核心对象  数据库连接池对象 ComboPooledDataSource
    4. 获取连接：getConnection


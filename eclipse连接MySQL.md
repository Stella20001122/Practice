# eclipse连接MySQL

[Eclipse连接MySQL数据库（傻瓜篇） - wendy_cai - 博客园 (cnblogs.com)](https://www.cnblogs.com/caiwenjing/p/8079227.html)

[MySQL数据库操作（Command Line Client 控制台）_mysql command line client-CSDN博客](https://blog.csdn.net/Miss_suiFeng/article/details/107575629)

###### --sxj练习--5.13

###### 项目：connectorMySQL

1. 在MySQL官网下载驱动（mysql-connector-java-8.0.26.jar）

2. Eclipse中新建一个java project，右键项目-->Build Path-->Add External Archives，选择1中下好的jar包，成功导入驱动

   ![](D:\godblessing\forwork\操作记录\figure\连接SQL-1.png)

3. 使用mysql command line client（命令行）创建新的数据库

CREATE DATABASE database_name; 创建一个新的数据库

DROP DATABASE database_name; 删除一个数据库

CREATE TABLE table_name (column1 datatype, column2 datatype, ...); 创建一个新的表格

DROP TABLE table_name; 删除一个表格

INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...); 插入一条新的数据

SELECT * FROM table_name; 显示所有数据

SELECT * FROM table_name WHERE condition; 显示符合条件的数据

示例：

```sql
CREATE DATABASE test;

use test;

CREATE TABLE user (name VARCHAR(20),password VARCHAR(20));

INSERT INTO user VALUES('huzhiheng','123456');

INSERT INTO user VALUES('Amy','12138');

select * from test;
```

4. 在eclipse中新建一个类TestConnect：

   ```java
   package test;
   import java.sql.*;
   public class TestConnect {
   	  public static void main(String args[]) {
   	    try {
   	      Class.forName("com.mysql.jdbc.Driver");     //加载MYSQL JDBC驱动程序   
   	      //Class.forName("org.gjt.mm.mysql.Driver");
   	     System.out.println("Success loading Mysql Driver!");
   	    }
   	    catch (Exception e) {
   	      System.out.print("Error loading Mysql Driver!");
   	      e.printStackTrace();
   	    }
   	    try {
   	      Connection connect = DriverManager.getConnection(
   	          "jdbc:mysql://localhost:3306/test","root","123456");
   	       //连接URL为   jdbc:mysql//服务器地址/数据库名  ，后面的2个参数分别是登陆用户名和密码
   
   	      System.out.println("Success connect Mysql server!");
   	      Statement stmt = connect.createStatement();
   	      ResultSet rs = stmt.executeQuery("select * from user");
   	                                          //user 为你表的名称
   	      while (rs.next()) {
   	      	System.out.println(rs.getString("name"));
   	      }
   	    }
   	    catch (Exception e) {
   	      System.out.print("get data error!");
   	      e.printStackTrace();
   	    }
   	  }
   	
   }
   
   ```

   5. 运行，显示下面的Success说明成功连接了

      ![](D:\godblessing\forwork\操作记录\figure\连接SQL-2.png)

   


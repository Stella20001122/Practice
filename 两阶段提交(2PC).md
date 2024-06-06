## 两阶段提交(2PC)

###### 练习：my2pc

1. 模拟处理跨数据库事务；这里会涉及两个MySQL数据库，模拟转账操作从一个账户到另一个账户，这两个账户分别存储在两个不同的数据库中。

   首先创建数据库和表，在mysql命令行中输入命令

   ```sql
   CREATE DATABASE testdb;
   USE testdb;
   CREATE TABLE accounts (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(255),
       balance DECIMAL(10, 2)
   );
   
   CREATE DATABASE testdb2;
   USE testdb2;
   CREATE TABLE accounts (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(255),
       balance DECIMAL(10, 2)
   );
   
   -- 插入一些初始数据
   INSERT INTO bank1.accounts (name, balance) VALUES ('Alice', 1000);
   INSERT INTO bank2.accounts (name, balance) VALUES ('Bob', 500);
   
   ```

   2. 新建java项目，右键项目-->Build Path-->Add External Archives，导入驱动（mysql-connector-java-8.0.26.jar）（在D盘）
   3. 新建一个接口Participant，定义所有参与两阶段提交协议的对象必须实现的方法。在最基本的形式中，这些方法包括 `prepare()`, `commit()`, 和 `rollback()`。

```java
package my2pcCode;

public interface Participant {
    /**
     * 准备阶段，确定是否可以提交事务。
     * @return 如果准备就绪并同意提交事务，返回true；否则返回false。
     */
    boolean prepare();

    /**
     * 提交阶段，提交事务的改变。
     */
    void commit();

    /**
     * 回滚阶段，回滚所有的改变。
     */
    void rollback();
}

```

4. 创建一个类DBParticipant，继承Participant接口，并实现里面的方法；

   用于管理与数据库的连接；

   其中借助了 java.sql.Connection--用于处理事务

```
package my2pcCode;

//import java.sql.Connection;
//import java.sql.DriverManager;
//import java.sql.PreparedStatement;
//import java.sql.SQLException;
import java.sql.*;

public class DBParticipant implements Participant {
    private final String url;
    private final String username;
    private final String password;
    private Connection connection;
    private PreparedStatement statement;

    public DBParticipant(String url, String username, String password, String sql) {
        this.url = url;
        this.username = username;
        this.password = password;
        try {
            connection = DriverManager.getConnection(url, username, password);
            connection.setAutoCommit(false);
            this.statement = connection.prepareStatement(sql);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public boolean prepare() {
        try {
        	//用于执行 INSERT、UPDATE 或 DELETE 语句以及 SQL DDL（数据定义语言）语句，
            //例如 CREATE TABLE 和 DROP TABLE。INSERT、UPDATE 或 DELETE 语句的效果是修改表中零行或多行中的一列或多列。
            //executeUpdate  的返回值是一个整数（int），指示受影响的行数（即更新计数）。 
            //对于 CREATE TABLE 或 DROP TABLE 等不操作行的语句，executeUpdate 的返回值总为零。 
            int result = statement.executeUpdate();
            return result == 1;
        } catch (SQLException e) {
            e.printStackTrace();
            return false;
        }
    }

    // JDBC程序中当一个Connection对象创建时，默认情况下是自动提交事务：
    // 每次执行一个SQL语句时，如果执行成功，就会向数据库自动提交，而不能回滚。
    // JDBC程序中为了让多个SQL语句作为一个整体执行，需要使用事务
    // 调用Connection的setAutoCommit(false)可以取消自动提交事务
    //在所有的SQL语句都成功执行后，调用Connection的commit()；方法提交事务
    //在其中某个操作失败或出现异常时，调用Connection的rollback)；方法回滚事务

    @Override
    public void commit() {
        try {
            connection.commit();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    @Override
    public void rollback() {
        try {
            connection.rollback();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}

```

5. 编写一个类TransactionManager，用于实现两阶段提交的逻辑

```
package my2pcCode;

import java.util.ArrayList;
import java.util.List;

public class TransactionManager {
    private final List<Participant> participants = new ArrayList<>();

    public void addParticipant(Participant participant) {
        participants.add(participant);
    }

    public void executeTransaction() {
        System.out.println("Transaction started.");

        // 第一阶段：准备阶段--只要有一个没准备好，就返回false，触发回滚
        boolean allPrepared = true;
        for (Participant participant : participants) {
            if (!participant.prepare()) {
                allPrepared = false;
                break;
            }
        }

        // 第二阶段：提交或回滚阶段
        if (allPrepared) {
            System.out.println("All participants are prepared. Committing transaction.");
            for (Participant participant : participants) {
                participant.commit();
            }
            System.out.println("Transaction committed.");
        } else {
            System.out.println("Some participants are not prepared. Rolling back transaction.");
            for (Participant participant : participants) {
                participant.rollback();
            }
            System.out.println("Transaction rolled back.");
        }
    }
}

```

6. 编写Main函数，用于测试一下

   注意url，用户名和密码改成正确的（数据库对应之前新建的）

```
package my2pcCode;

public class Main {
    public static void main(String[] args) {
        TransactionManager tm = new TransactionManager();

        // 银行数据库的JDBC URL可能需要根据实际情况调整
        String urlBank1 = "jdbc:mysql://localhost:3306/testdb";
        String urlBank2 = "jdbc:mysql://localhost:3306/testdb2";
        String user = "root";
        String password = "123456";

        // 定义SQL命令，代表事务操作
        String sqlBank1 = "UPDATE accounts SET balance = balance - 200 WHERE name = 'Alice'";
        String sqlBank2 = "UPDATE accounts SET balance = balance + 200 WHERE name = 'Bob'";

        // 添加数据库参与者
        tm.addParticipant(new DBParticipant(urlBank1, user, password, sqlBank1));
        tm.addParticipant(new DBParticipant(urlBank2, user, password, sqlBank2));

        // 执行事务
        tm.executeTransaction();
    }
}

```

7. 运行main函数之后，命令行输出

```
Transaction started.
All participants are prepared. Committing transaction.
Transaction committed.
```

表示成功执行；

8. 此时查看数据库的变化，发现跟原本的相比的确发生了预料中的变化

![](D:\godblessing\forwork\操作记录\figure\两阶段提交.png)

9. rollback如何测试？

可以在DBParticipant类里面添加一个布尔属性，控制成功或失败
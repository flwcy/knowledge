### 数据库连接池

用户每次请求都需要向数据库获得连接，而数据库创建连接通常需要消耗相对较大的资源，创建时间也较长。**数据库连接池负责分配,管理和释放数据库连接,它允许应用程序重复使用一个现有的数据库连接,而不是重新建立一个**。

实现连接池功能的步骤：

1. 在`DataSource`构造函数中批量创建与数据库的连接，并把创建的连接加入`LinkedList`对象中。
2. 实现`getConnection`方法，让`getConnection`方法每次调用时，从`LinkedList`中取一个`Connection`返回给用户。
3. 当用户使用完`Connection`，调用`Connection.close()`方法时，`Collection`对象应保证将自己返回到`LinkedList`中,而不要把`conn`还给数据库。**Collection保证将自己返回到LinkedList中是此处编程的难点**。

#### 数据库连接池雏形 

编写`SimpleDataSource.java`

```java
package com.flwcy.util;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.LinkedList;

/**
 * 简单的数据库连接池
 */
public class SimpleDataSource {

    /**
     * 因为需要频繁的读写，使用双向链表结构的LinkedList
     */
    private static LinkedList<Connection> pool;

    private static final String USER = "root";

    private static final String PASSWORD = "123456";

    private static final String URL = "jdbc:mysql://localhost:3306/db_jdbc";

    static {
        pool = new LinkedList<>();

        // 加载驱动
        try {
            Class.forName("com.mysql.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    public SimpleDataSource(){
        try {
            for (int i=0;i<5;i++){
                pool.addLast(this.createConnection());
            }
        } catch (SQLException e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    public Connection createConnection() throws SQLException {
        return DriverManager.getConnection(URL,USER,PASSWORD);
    }

    public Connection getConnection(){
        return this.pool.removeFirst();
    }

    public void free(Connection connection){
        pool.addLast(connection);
    }
}
```

修改之前的工具类

```java
package com.flwcy.util;

import java.sql.*;

/**
 * 静态内部类的单例DbUtils
 */
public class SelfDbUtils {

    private static SimpleDataSource dataSource = new SimpleDataSource();

    private SelfDbUtils(){}

    private static class SelfDbUtilsHolder{
        private static final SelfDbUtils INSTANCE = new SelfDbUtils();
    }

    public static SelfDbUtils getInstance(){
        return SelfDbUtilsHolder.INSTANCE;
    }

    public Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }

    public void close(Connection connection,Statement statement,ResultSet resultSet){
        try {
            if(resultSet != null)
                resultSet.close();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            try {
                if(statement != null)
                    statement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            } finally{
                if(connection != null)
                    dataSource.free(connection);
            }
        }
    }
}
```

开始进行单元测试

```java
    @Test
    public void createConnection(){
        try {
            for(int i=0;i<10;i++){
                Connection connection = SelfDbUtils.getInstance().getConnection();
                System.out.println(connection);
                SelfDbUtils.getInstance().close(connection,null,null);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```

执行结果为：

```
com.mysql.jdbc.JDBC4Connection@4ca8195f
com.mysql.jdbc.JDBC4Connection@65e579dc
com.mysql.jdbc.JDBC4Connection@61baa894
com.mysql.jdbc.JDBC4Connection@b065c63
com.mysql.jdbc.JDBC4Connection@768debd
com.mysql.jdbc.JDBC4Connection@4ca8195f
com.mysql.jdbc.JDBC4Connection@65e579dc
com.mysql.jdbc.JDBC4Connection@61baa894
com.mysql.jdbc.JDBC4Connection@b065c63
com.mysql.jdbc.JDBC4Connection@768debd
```

可以看出数据库连接被重复使用了，因为通过打印的语句可以看出，有相同`hashcode`的`Connection`

#### 数据库连接池优化

为了保证，每一个线程获取的数据库连接实例是不同的，我们需要对线程池进行加锁，保证多线程并发时获取的连接各不相干，修改其中的代码片段如下

```java
    public Connection getConnection() throws SQLException {
        synchronized (pool){
            if(pool.size() > 0)
                return this.pool.removeFirst();
            // 无可用连接，且已创建的连接数小于最大连接数
            if(currentCount < maxCount){
                currentCount++;
                return this.createConnection();
            }
        }

        throw new SQLException(" don't have connection now!");
    }
```

#### 数据库连接池之代理模式

在使用`JDBC`连接数据库的时候，会经常使用`connection.close()`这样的方法，去关闭数据库连接，如果使用者按照传统的方式关闭连接，那么我们的连接池就没有存在的意义了，因为每一次使用者都会给关闭掉，导致连接池的连接会是无效的或者越来越少，为了防止这样的事情发生，我们需要保留使用者的使用习惯，也就是说允许使用者通过`close`方法释放连接，这个时候我们应该如何做既能起到保留使用者的使用习惯，又能在进行关闭的时候不是真的关掉数据库连接，而是直接存放至数据库连接池中。

##### 静态代理

解决这一问题的办法就是使用代理模式，因为代理模式可以替代原有类的行为，所以我们要做的就是替换掉`connection`的`close`方法。

我们实现`Connection`这个接口，我去掉了很多方法，因为都类似，全贴上来占地方。

```java
/**
 * 静态代理，自定义连接
 */
public class CustomConnection implements Connection {

    /**
     * 真实的连接对象
     */
    private static Connection realConnection;

    /**
     * 数据库连接池
     */
    private static CustomDataSource dataSource;

    /**
     * 包访问权限，即在整个包内均可被访问
     * @param realConnection
     * @param dataSource
     */
    CustomConnection(Connection realConnection,CustomDataSource dataSource){
        this.realConnection = realConnection;
        this.dataSource = dataSource;
    }

    @Override
    public void close() throws SQLException {
        dataSource.getPool().addLast(this);
    }

    @Override
    public Statement createStatement() throws SQLException {
        return this.realConnection.createStatement();
    }
  
    // 省略其他方法
} 
```

修改数据库连接池的代码：

```java
    public Connection createConnection() throws SQLException {
        Connection realConneciton = DriverManager.getConnection(URL,USER,PASSWORD);
        CustomConnection connection = new CustomConnection(realConneciton,this);
        return connection;
    }
```

在上述代码中我们可以看出我们自定义了一个`Connection`的实现类，并且在连接池中存放的是我们自定义的`Connection`类，这样在进行关闭的时候我们可以将数据库连接存放至连接池中。

##### 动态代理

可以看出，静态代理的机制，在实现上很死板，并且我们需要重复写的东西实在太多了，从`JDK1.3`开始有了一个动态代理机制，我们可以利用该机制来实现我们刚才想要的功能。

每一个动态代理类都必须要实现`InvocationHandler`这个接口，并且需要通过`Proxy`的`newProxyInstance`方法来创建我们的代理对象：

```java
package com.flwcy.datasource;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.sql.Connection;

public class ConnectionHandler implements InvocationHandler {

    private Connection realConnection;

    private CustomDataSource dataSource;

    private Connection warpedConnection;

    ConnectionHandler(CustomDataSource dataSource){
        this.dataSource = dataSource;
    }

    public Connection bind(Connection realConnection) {
        this.realConnection = realConnection;
        this.warpedConnection = (Connection) Proxy.newProxyInstance(this.getClass().getClassLoader(),new Class[]{Connection.class},this);
        return  warpedConnection;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result = null;
        if(method.getName().equals("close")){
           this.dataSource.free(this.warpedConnection);
        } else {
            result = method.invoke(realConnection,args);
        }
        return result;
    }
}
```

修改`DataSource`中创建`Connection`方法

```java
    public Connection createConnection() throws SQLException {
        Connection realConnection = DriverManager.getConnection(URL,USER,PASSWORD);
        // CustomConnection connection = new CustomConnection(realConneciton,this);
        ConnectionHandler handler = new ConnectionHandler(this);
        return handler.bind(realConnection);
    }
```

可以发现代码简化了好多，并且实现了我们所需要的功能。

#### DBCP 数据库连接池的使用 

写到这里，我们的连接池已经有些接近真实的数据库连接池了，但是其功能还是很单一，健壮性远远不够，不能直接在实际项目中应用，但是通过前面几个小节的讲解和逐步改进，我们最起码了解了什么是数据库连接池，以及连接池或与应该具备怎样的功能，在开源界，连接池框架实在太多了，并且健壮性足够而且还有专门的优秀团队维护，比如 DBCP， C3P0，DbPool 等等。
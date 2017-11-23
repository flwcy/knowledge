# 在实际项目中使用JDBC

标签（空格分隔）： 分层 异常处理 JDBC

---



### 基本架构
在实际项目中，我们经常会通过查询获取数据,然后将数据用于其他的用途，并非简单的打印或者展示，其次，在实际的应用中，和数据库打交道的jdbc代码会很少出现在业务逻辑中，因为这样对代码的维护以及再扩展会带来极大的开销。
![架构](../img/jdbc/JDBC_Schema.jpg)
使用接口隔离，这样设计的好处就是通过数据访问层的接口完全隐藏了数据层的实现细节，让业务逻辑层不需要关系具体的实现细节.例如我们将数据访问的具体实现从JDBC换成hibernate,对业务逻辑层并没有影响.
#### DAO
DAO的全称是data access object,其中非常重要的一个概念就是Domain对象，也就是说一个最常用的POJO与数据库中的一个表相对应.首先我们需要设计一个domain对象，这里我们为之前的db_user表设计对应的domain对象.
```java
package com.rooike.entity;

import java.sql.Timestamp;

/**
 * Created by three on 2015/9/15.
 */
public class User {
    private Integer id;

    private String userName;

    private String password;

    private String email;

    private Timestamp birthday;

    public User(){}

    public User(Integer id, String userName, String password, String email, Timestamp birthday) {
        this.id = id;
        this.userName = userName;
        this.password = password;
        this.email = email;
        this.birthday = birthday;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Timestamp getBirthday() {
        return birthday;
    }

    public void setBirthday(Timestamp birthday) {
        this.birthday = birthday;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", userName='" + userName + '\'' +
                ", password='" + password + '\'' +
                ", email='" + email + '\'' +
                ", birthday=" + birthday +
                '}';
    }
}
```
domain对象已经编写好了，domain对象中包括了数据表中所有的字段，并提供了get与set方法.接下来我们定义一个DAO接口来供业务逻辑层调用
```java
package com.rooike.dao;

import java.util.List;

/**
 * Created by three on 2015/9/16.
 */
public interface UserDao<User> {

    /**
     * 新增用户
     * @param user
     * @return
     */
    int insert(User user);

    /**
     * 更新用户信息
     * @param user
     * @return
     */
    int update(User user);

    /**
     * 根据用户id删除用户
     * @param id
     * @return
     */
    int delete(Integer id);

    /**
     * 根据用户id查询对应的用户信息
     * @param id
     * @return
     */
    User selectById(Integer id);

    /**
     * 查询所有的用户
     * @return
     */
    List<User> selectAll();
}

```
 一个简单的DAO接口的实现
```java
package com.rooike.dao.impl;

import com.rooike.dao.UserDao;
import com.rooike.entity.User;
import com.rooike.util.JdbcUtils;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

/**
 * Created by three on 2015/9/16.
 */
public class PreparedUserDaoImpl implements UserDao<User> {

    private String url = "jdbc:mysql://localhost:3306/db_jdbc";

    private String userName = "root";

    private String password = "123456";

    @Override
    public int insert(User user) {
        int result = -1;
        if (user == null) return result;
        Connection connection = null;
        PreparedStatement preparedStatement = null;

        try {
            String sql = "insert into db_jdbc.db_user(user_name,password,email,birthday) values(?,?,?,?)";
            sql = String.format(sql, user.getUserName(), user.getPassword(), user.getEmail(), user.getBirthday());
            connection = JdbcUtils.getConn();

            preparedStatement = connection.prepareStatement(sql);

            preparedStatement.setString(1,user.getUserName());
            preparedStatement.setString(2,user.getPassword());
            preparedStatement.setString(3,user.getEmail());
            preparedStatement.setTimestamp(4, user.getBirthday());

            preparedStatement.executeUpdate();

        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.close(null, preparedStatement, connection);
        }

        return result;
    }

    @Override
    public int update(User user) {
        int result = -1;
        if (user == null) return result;
        User record = selectById(user.getId());

        String sql = "update db_jdbc.db_user set user_name=?,password=?,email=?,birthday=? where id = ?";
        String userName = (user.getUserName() == null) ? record.getUserName() : user.getUserName();
        String password = (user.getPassword() == null) ? record.getPassword() : user.getPassword();
        String email = (user.getEmail() == null) ? record.getEmail() : user.getEmail();
        Timestamp birthday = (user.getBirthday() == null) ? record.getBirthday() : user.getBirthday();

        Connection connection = null;
        PreparedStatement preparedStatement = null;

        try {
            connection = JdbcUtils.getConn();
            preparedStatement = connection.prepareStatement(sql);
            preparedStatement.setString(1,user.getUserName());
            preparedStatement.setString(2,user.getPassword());
            preparedStatement.setString(3,user.getEmail());
            preparedStatement.setTimestamp(4, user.getBirthday());
            preparedStatement.setInt(5,user.getId());
            result = preparedStatement.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.close(null, preparedStatement, connection);
        }

        return result;
    }

    @Override
    public int delete(Integer id) {
        Integer result = -1;
        if (id == null || id <= 0) return result;
        String sql = "delete from db_jdbc.db_user where id = %s";
        sql = String.format(sql, id);
        Connection connection = null;
        Statement statement = null;
        try {
            connection = JdbcUtils.getConn();
            statement = connection.createStatement();
            result = statement.executeUpdate(sql);
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.close(null, statement, connection);
        }
        return result;
    }

    @Override
    public User selectById(Integer id) {
        User record = null;
        if (id == null || id <= 0) return record;
        String sql = "select * from db_jdbc.db_user where id = %s";
        sql = String.format(sql, id);
        Connection connection = null;
        Statement statement = null;
        ResultSet resultSet = null;
        try {
            connection = JdbcUtils.getConn();
            statement = connection.createStatement();
            resultSet = statement.executeQuery(sql);
            while (resultSet.next()) {
                String userName = resultSet.getString(2);
                String password = resultSet.getString(3);
                String email = resultSet.getString(4);
                Timestamp birthday = resultSet.getTimestamp(5);

                record = new User();
                record.setId(id);
                record.setUserName(userName);
                record.setEmail(email);
                record.setBirthday(birthday);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.close(resultSet, statement, connection);
        }
        return record;
    }

    @Override
    public List<User> selectAll() {
        Connection connection = null;
        Statement statement = null;
        ResultSet resultSet = null;
        String sql = "SELECT * FROM db_jdbc.db_user";
        List<User> list = new ArrayList<User>();
        try {
            //2、获取数据库连接
            connection = JdbcUtils.getConn();
            //3、获取数据库发送sql语句的statement对象
            statement = connection.createStatement();

            //4、向数据库发送sql，获取数据库结果集
            resultSet = statement.executeQuery(sql);

            //5、从结果集获取数据
            while (resultSet.next()) {
                Integer id = resultSet.getInt("id");
                String userName = resultSet.getString("user_name");
                String password = resultSet.getString("password");
                String email = resultSet.getString("email");
                Timestamp birthday = resultSet.getTimestamp("birthday");
                User user = new User(id, userName, password, email, birthday);
                list.add(user);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.close(resultSet, statement, connection);
        }
        return list;
    }
}
```
最后我们编写junit代码对DAO接口中所有的方法进行逐个测试
```java
package com.rooike.dao.impl.test;

import com.mysql.jdbc.Driver;
import com.rooike.dao.UserDao;
import com.rooike.entity.User;
import org.junit.Test;

import java.sql.*;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.List;

/**
 * Created by three on 2015/9/16.
 */
public class UserDaoTest {

    private SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

    private UserDao<User> userDao;

    @Test
    public void selectAll() {
        List<User> list = userDao.selectAll();
        for (User user : list) {
            System.out.println(user);
        }
    }

    @Test
    public void selectById() {
        User user = userDao.selectById(1);
        System.out.println(user);
    }

    @Test
    public void update() {
        User user = userDao.selectById(1);

        try {
            user.setBirthday(new Timestamp(sdf.parse("1991-10-15 21:12:12").getTime()));
        } catch (ParseException e) {
            e.printStackTrace();
        }
        userDao.update(user);
        System.out.println(userDao.selectById(1));
    }

    @Test
    public void insert() {
        User user = new User();
        user.setUserName("wcy");
        user.setPassword("654321");
        user.setEmail("wcy@test.net");

        try {
            user.setBirthday(new Timestamp(sdf.parse("1991-12-21 22:00:01").getTime()));
        } catch (ParseException e) {
            e.printStackTrace();
        }
        userDao.insert(user);
    }

    @Test
    public void delete() {
        userDao.delete(3);
    }

    @Test
    public void test() throws SQLException {
        //1、注册驱动(Driver)
        DriverManager.registerDriver(new Driver());
        //2、建立数据库连接(Connection)
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db_jdbc","root","123456");
        //3、创建执行sql语句(一般是Statement及其子类)
        Statement statement = conn.createStatement();
        //4、执行sql语句获得结果集(ResultSet)
        ResultSet resultSet = statement.executeQuery("select * from db_user");
        //5、处理执行结果，在非查询语句中，该步骤可以省略
        while (resultSet.next()) {
            String result = String.format("id=%s,userName=%s,passowrd=%s,email=%s,birthday=%s", resultSet.getInt("id"),
                    resultSet.getString("user_name"),resultSet.getString("password"), resultSet.getString("email")
                    ,resultSet.getDate("birthday"));
            System.out.println(result);
        }
        //6、关闭连接，释放资源
        resultSet.close();
        statement.close();
        conn.close();
    }
}
```
我们对原有的项目进行这样更改之后，不难发现，**操作数据库的具体实现过程完全被隐藏了起来，如果我们要更改其中的细节，业务逻辑层的相关代码则无需进行更改**。如果需要增加新的接口，可以进行继承或者在原有的接口基础之上进行修改，应用照样是不需要进行替换。
### 异常处理
在之前的练习代码中，为了顺利通过编译，我们只是简单的打印了一下堆栈的信息，并没有通知上一层使用者或者展示层，来表示现在出现的错误，并且也没有想办法进行相应的容错，当然，如果sql语句出现错误，或者逐渐冲突，容错是非常难做的，所以一般情况下数据库执行sql语句的错误只需要通过异常的传递机制告知上一层使用者即可。
![异常](../img/jdbc/jdbc_exception.jpg)
如上图所示,异常发生转移了,这样的对异常进行简单的try...catch,不便于我们进行错误定位.
>Public User selectByUserName(String userName) throws SQLException;

另一种常见的做法是直接将异常抛出,此时，我们将异常直接抛出后，必须修改接口.并且业务层必须进行try/catch或者继续抛出异常.而且这样修改后，我们无法更换数据访问层的实现(比如我们抛出SQLException[编译时异常],但是Hibernate中并没有java.sql.SQLException)，此时我们必须修改接口，而且业务逻辑层必须进行异常处理(tr/catch或者继续向上抛出)
我们采用三层架构设计的一个重要目标是让层与层之间相互独立,能够灵活更改层的具体实现细节.因此我们需要采用另外一种方式,自定义异常:
```java
package com.rooike.dao;

/**
 * Created by three on 2016/6/2.
 */
public class DaoException extends RuntimeException {
    public DaoException() {
        super();
    }

    public DaoException(String message) {
        super(message);
    }

    public DaoException(String message, Throwable cause) {
        super(message, cause);
    }

    public DaoException(Throwable cause) {
        super(cause);
    }

    protected DaoException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}
```
这样修改后，我们可以达到不污染接口的目的,也能够达到将异常通知到上一层代码的目的,这样业务逻辑层也能够进行选择，如果业务逻辑层能够处理,就进行try/catch,不能处理该异常,可以继续向上抛出.
```
    @Override
    public int insert(User user) {
        int result = -1;
        if (user == null) return result;
        Connection connection = null;
        Statement statement = null;

        try {
            String sql = "insert into db_jdbc.db_user(user_name,password,email,birthday) values('%s','%s','%s','%s')";
            sql = String.format(sql, user.getUserName(), user.getPassword(), user.getEmail(), user.getBirthday());
            connection = JdbcUtils.getConn();

            statement = connection.createStatement();

            statement.executeUpdate(sql);

        } catch (Exception e) {
            throw new DaoException(e.getMessage(),e);
        } finally {
            JdbcUtils.close(null, statement, connection);
        }

        return result;
    }
```

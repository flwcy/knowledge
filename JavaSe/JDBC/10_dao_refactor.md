### DAO层代码重构

在实际项目中，针对不同的业务会有不同的`domain`对象。查看之前的`DAO`层代码：

```java
    public int update(User user) {
        int result = -1;
        if(user != null) {
            Connection connection = null;
            PreparedStatement statement = null;
            ResultSet resultSet = null;

            try {
                connection = SelfDbUtils.getInstance().getConnection();
                String sql = "update db_user set user_name = ?,password = ?,email = ?,birthday = ? where id = ?;";
                statement = connection.prepareStatement(sql);
                statement.setString(1,user.getUserName());
                statement.setString(2,user.getPassword());
                statement.setString(3,user.getEmail());
                statement.setDate(4, new Date(user.getBirthday().getTime()));
                statement.setInt(5,user.getId());
                result = statement.executeUpdate();
            } catch (SQLException e) {
                throw DaoException.wrap(e,ErrorCode.UPDATE_ERROR);
            } finally {
                SelfDbUtils.getInstance().close(connection,statement,resultSet);
            }
        }
        return result;
    }
```

我们发现代码中存在很多公共的代码（冗余代码），因此我们重构代码时可以考虑将不变的部分和变化的部分区分开来，可以抽离成一个超类，对于变化的部分可以通过参数传递或者子类来实现：

```java
package com.flwcy.dao.refactor;

import com.flwcy.enums.ErrorCode;
import com.flwcy.exception.DaoException;
import com.flwcy.util.SelfDbUtils;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public abstract class BaseDao {
    /**
     *
     * @param sql
     * @param flag true插入操作,false删除/更新操作
     * @param args
     * @return
     */
    public int update(String sql,boolean flag,Object... args) {
        int result = -1;
        Connection connection = null;
        PreparedStatement statement = null;
        ResultSet resultSet = null;
        try {
            connection = SelfDbUtils.getInstance().getConnection();
            statement = (flag) ? connection.prepareStatement(sql,Statement.RETURN_GENERATED_KEYS) : connection.prepareStatement(sql);
            if (args != null && args.length > 0) {
                for(int i = 0; i<args.length; i++){
                    statement.setObject(i+1, args[i]);
                }
            }
            result = statement.executeUpdate();
            if(flag) {
                resultSet = statement.getGeneratedKeys();
                while (resultSet.next()) {
                    result = resultSet.getInt(1);
                }
            }
        } catch (SQLException e) {
            throw DaoException.wrap(e,ErrorCode.UPDATE_ERROR);
        } finally {
            SelfDbUtils.getInstance().close(connection,statement,resultSet);
        }
        
        return result;
    }
}
```

修改`UserDaoImpl`：

```java
package com.flwcy.dao.refactor;

import com.flwcy.dao.UserDao;
import com.flwcy.entity.User;

import java.util.List;

public class UserDaoImpl extends BaseDao implements UserDao {
    @Override
    public List<User> selectAll() {
        return null;
    }

    @Override
    public User selectByName(String name) {
        return null;
    }

    @Override
    public int update(User user) {
        String sql = "update db_user set user_name = ?,password = ?,email = ?,birthday = ? where id = ?;";
        Object[] args = new Object[] {user.getUserName(),user.getPassword(),user.getEmail(),user.getBirthday(),user.getId()};
        return super.update(sql,false,args);
    }

    @Override
    public int insert(User user) {
        String sql = "insert into db_user(user_name,password,email,birthday) values(? , ?, ?, ?)";
        Object[] args = new Object[] {user.getUserName(),user.getPassword(),user.getEmail(),user.getBirthday()};
        return super.update(sql,true,args);
    }

    @Override
    public int delete(Integer id) {
        String sql = "delete from db_user where id = ?";
        return super.update(sql,false,id);
    }
}
```

#### 使用模板模式处理DAO中的查询方法

对于查询方法，我们需要返回具体的`domain`对象，而超类中并不知道需要返回哪种`domain`对象，因此我们将组装对象这个定义成抽象方法，交由子类去实现，这种设计在设计模式中称为模版模式。

> 在面向对象开发过程中，通常我们会遇到这样的一个问题：我们知道一个算法所需的关键步骤，并确定了这些步骤的执行顺序。但是某些步骤的具体实现是未知的，或者说某些步骤的实现与具体的环境相关。
>
> 在阎宏博士的《JAVA与模式》一书中开头是这样描述模板方法（Template Method）模式的：
>
> 　　**模板方法模式是类的行为模式。准备一个抽象类，将部分逻辑以具体方法以及具体构造函数的形式实现，然后声明一些抽象方法来迫使子类实现剩余的逻辑。不同的子类可以以不同的方式实现这些抽象方法，从而对剩余的逻辑有不同的实现。这就是模板方法模式的用意。**

修改超类代码如下：

```java
package com.flwcy.dao.refactor;

import com.flwcy.enums.ErrorCode;
import com.flwcy.exception.DaoException;
import com.flwcy.util.SelfDbUtils;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public abstract class BaseDao {
    /**
     *
     * @param sql
     * @param flag true插入操作,false删除/更新操作
     * @param args
     * @return
     */
    public int update(String sql,boolean flag,Object... args) {
        int result = -1;
        Connection connection = null;
        PreparedStatement statement = null;
        ResultSet resultSet = null;
        try {
            connection = SelfDbUtils.getInstance().getConnection();
            statement = (flag) ? connection.prepareStatement(sql,Statement.RETURN_GENERATED_KEYS) : connection.prepareStatement(sql);
            if (args != null && args.length > 0) {
                for(int i = 0; i<args.length; i++){
                    statement.setObject(i+1, args[i]);
                }
            }
            result = statement.executeUpdate();
            if(flag) {
                resultSet = statement.getGeneratedKeys();
                while (resultSet.next()) {
                    result = resultSet.getInt(1);
                }
            }
        } catch (SQLException e) {
            throw DaoException.wrap(e,ErrorCode.UPDATE_ERROR);
        } finally {
            SelfDbUtils.getInstance().close(connection,statement,resultSet);
        }

        return result;
    }

    /**
     *
     * @param sql
     * @param flag true查询所有,false查询单个结果
     * @param args
     * @return
     */
    public Object select(String sql,boolean flag,Object... args) {
        Object result = null;
        List<Object> list = null;
        Connection connection = null;
        PreparedStatement statement = null;
        ResultSet resultSet = null;

        try {
            if(flag)
                list = new ArrayList<>();
            connection = SelfDbUtils.getInstance().getConnection();
            statement = connection.prepareStatement(sql);
            if (args != null && args.length > 0) {
                for(int i = 0; i<args.length; i++){
                    statement.setObject(i+1, args[i]);
                }
            }
            resultSet = statement.executeQuery();
            while (resultSet.next()) {
                result = rowMapper(resultSet);
                if(flag)
                    list.add(result);
            }

        } catch (SQLException e) {
            throw DaoException.wrap(e, ErrorCode.SELECT_ERROR);
        } finally {
            SelfDbUtils.getInstance().close(connection,statement,resultSet);
        }
        return (flag) ? list : result;
    }

    protected abstract Object rowMapper(ResultSet resultSet) throws SQLException;
}
```

修改`UserDaoImpl`的查询方法：

```java
package com.flwcy.dao.refactor;

import com.flwcy.dao.UserDao;
import com.flwcy.entity.User;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

public class UserDaoImpl extends BaseDao implements UserDao {
    @Override
    public List<User> selectAll() {
        String sql = "select id,user_name,password,email,birthday from db_user";
        return (List<User>) super.select(sql,true,null);
    }

    @Override
    public User selectByName(String name) {
        String sql = "select id,user_name,password,email,birthday from db_user where user_name = ?";
        return (User) super.select(sql,false,name);
    }

    @Override
    public int update(User user) {
        String sql = "update db_user set user_name = ?,password = ?,email = ?,birthday = ? where id = ?;";
        Object[] args = new Object[] {user.getUserName(),user.getPassword(),user.getEmail(),user.getBirthday(),user.getId()};
        return super.update(sql,false,args);
    }

    @Override
    public int insert(User user) {
        String sql = "insert into db_user(user_name,password,email,birthday) values(? , ?, ?, ?)";
        Object[] args = new Object[] {user.getUserName(),user.getPassword(),user.getEmail(),user.getBirthday()};
        return super.update(sql,true,args);
    }

    @Override
    public int delete(Integer id) {
        String sql = "delete from db_user where id = ?";
        return super.update(sql,false,id);
    }

    @Override
    public User rowMapper(ResultSet resultSet) throws SQLException {
        User user = new User();
        user.setId(resultSet.getInt("id"));
        user.setUserName(resultSet.getString("user_name"));
        user.setPassword(resultSet.getString("password"));
        user.setEmail(resultSet.getString("email"));
        user.setBirthday(resultSet.getDate("birthday"));
        return user;
    }
}
```


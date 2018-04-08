### 使用Spring中的JdbcTemple

为了使`JDBC`更加易于使用, `Spring`在`JDBC API`上定义了一个抽象层, 以此建立一个`JDBC`存取框架。

作为 `Spring JDBC`框架的核心， `JDBC`模板的设计目的是为不同类型的`JDBC`操作提供模板方法。每个模板方法都能控制整个过程，并允许覆盖过程中的特定任务。通过这种方式, 可以在尽可能保留灵活性的情况下，将数据库存取的工作量降到最低。

```java
package com.flwcy.dao.spring;

import com.flwcy.dao.UserDao;
import com.flwcy.entity.User;
import com.flwcy.util.SelfDbUtils;
import org.springframework.dao.EmptyResultDataAccessException;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.BeanPropertySqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.jdbc.core.namedparam.SqlParameterSource;
import org.springframework.lang.Nullable;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class UserDaoImpl implements UserDao {

    //获取数据源(设置为static 是因为该jdbcTemplate多次被调用)
    static JdbcTemplate jdbcTemplate = new JdbcTemplate(SelfDbUtils.getDataSource());

    @Override
    public List<User> selectAll() {
        String sql = "select id,user_name,password,email,birthday from db_user";
        return jdbcTemplate.query(sql,new BeanPropertyRowMapper<User>(User.class));
    }

    @Override
    public User selectByName(String name) {
        String sql = "select id,user_name,password,email,birthday from db_user where user_name = ?";
        Object[] args = new Object[]{name};
        User user = null;
        try {
            user = jdbcTemplate.queryForObject(sql,args,new BeanPropertyRowMapper<User>(User.class));
        } catch (EmptyResultDataAccessException e) {
            return null;
        }
        return user;
    }

    @Override
    public int update(User user) {
        String sql = "update db_user set user_name = ?,password = ?,email = ?,birthday = ? where id = ?;";
        Object[] args = new Object[] {user.getUserName(),user.getPassword(),user.getEmail(),user.getBirthday(),user.getId()};
        return jdbcTemplate.update(sql,args);
    }

    @Override
    public int insert(User user) {
        NamedParameterJdbcTemplate namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(SelfDbUtils.getDataSource());
        String sql = "insert into db_user(user_name,password,email,birthday) values( :userName, :password, :email, :birthday)";

        SqlParameterSource sqlParameterSource = new BeanPropertySqlParameterSource(user);

        return namedParameterJdbcTemplate.update(sql,sqlParameterSource);
    }

    @Override
    public int delete(Integer id) {
        String sql = "delete from db_user where id = ?";
        int result = jdbcTemplate.update(sql,id);
        return result;
    }

    @Override
    public List<String> selectNamesByBirthday(Date[] dates) {
        StringBuilder stringBuilder = new StringBuilder();
        for(Date date : dates){
            stringBuilder.append("?,");
        }

        String sql = "SELECT user_name FROM db_user WHERE birthday IN(" + stringBuilder.deleteCharAt(stringBuilder.length() - 1) +")";

        return jdbcTemplate.query(sql, dates, new RowMapper<String>() {
            @Nullable
            @Override
            public String mapRow(ResultSet resultSet, int i) throws SQLException {
                return resultSet.getString("user_name");
            }
        });
    }
}
```

### JdbcTemplate具名参数的使用

在`SQL`语句中使用具名参数时, 可以在一个 Map 中提供参数值, 参数名为键。也可以使用`SqlParameterSource`参数。批量更新时可以提供`Map`或`SqlParameterSource`的数组。下面是使用具名参数的第一种方法:

```java
    @Override
    public int insert(User user) {
        NamedParameterJdbcTemplate namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(SelfDbUtils.getDataSource());
        String sql = "insert into db_user(user_name,password,email,birthday) values( :userName, :password, :email, :birthday)";

        SqlParameterSource sqlParameterSource = new BeanPropertySqlParameterSource(user);

        return namedParameterJdbcTemplate.update(sql,sqlParameterSource);
    }
```

下面是使用具名参数的第二种方式:

```java
    @Override
    public int insert(User user) {
        NamedParameterJdbcTemplate namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(SelfDbUtils.getDataSource());
        String sql = "insert into db_user(user_name,password,email,birthday) values( :userName, :password, :email, :birthday)";
        Map<String,Object> map = new HashMap<>();
        map.put("userName",user.getUserName());
        map.put("password",user.getPassword());
        map.put("email",user.getEmail());
        map.put("birthday",user.getBirthday());

        return namedParameterJdbcTemplate.update(sql,map);
    }
```

此外具名参数还有另外一个`update`方法，用于获取返回值：

```java
    @Override
    public int insert(User user) {
        NamedParameterJdbcTemplate namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(SelfDbUtils.getDataSource());
        String sql = "insert into db_user(user_name,password,email,birthday) values( :userName, :password, :email, :birthday)";

        KeyHolder generatedKeyHolder = new GeneratedKeyHolder();
        namedParameterJdbcTemplate.update(sql,new BeanPropertySqlParameterSource(user),generatedKeyHolder);

        Integer id = generatedKeyHolder.getKey().intValue();
        return id;
    }
```


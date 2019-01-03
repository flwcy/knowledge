### Spring入门之登录案例

> 本文为阅读《精通Spring 4.x  企业应用开发实战》的个人总结

#### 创建数据库

首先新建一个用户表`t_user`

```sql
DROP TABLE IF EXISTS `t_user`;
CREATE TABLE `t_user` (
  `user_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `user_name` varchar(20) COLLATE utf8_unicode_ci DEFAULT NULL unique COMMENT '用户名',
  `password` varchar(50) COLLATE utf8_unicode_ci DEFAULT NULL COMMENT '密码',
	`crodits` int(11) DEFAULT NULL COMMENT '积分',
	`last_ip` varchar(50) COLLATE utf8_unicode_ci DEFAULT NULL COMMENT '最后登录IP',
	`last_visit` timestamp NULL DEFAULT CURRENT_TIMESTAMP COMMENT '最后访问时间',
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

新建登录日志表`t_login_log`

```sql
DROP TABLE IF EXISTS `t_login_log`;
CREATE TABLE `t_login_log` (
  `login_log_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
	`user_id` int(11) DEFAULT NULL COMMENT '用户ID',
	`ip` varchar(20) COLLATE utf8_unicode_ci DEFAULT NULL COMMENT '登录ip',
	`login_datetime` timestamp NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `create_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP COMMENT '登录时间',
  PRIMARY KEY (`login_log_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

`mysql`引擎为`InnoDB`，支持事务。

#### 创建项目

项目为标准的`maven web`项目结构。创建对应的实体`bean`。

`User`

```java
package com.flwcy.sl.entity;

import lombok.Data;

import java.util.Date;

/**
 * @Description: 对应数据库表t_user的实体对象
 * @author: flwcy
 * @date: 2019/1/2 10:07
 */
@Data
public class User {

    /**
     * 用户id
     */
    private Integer userId;

    /**
     * 用户名
     */
    private String userName;

    /**
     * 用户密码
     */
    private String password;

    /**
     * 积分
     */
    private Integer crodits;

    /**
     * 最后登录的ip地址
     */
    private String lastIp;

    /**
     * 最后访问时间
     */
    private Date lastVisit;
}
```

`LoginLog`

```java
package com.flwcy.sl.entity;

import lombok.Data;
import java.util.Date;

/**
 * @Description: 对应数据库表t_login_log的实体对象
 * @author: flwcy
 * @date: 2019/1/2 10:10
 */
@Data
public class LoginLog {

    private Integer loginLogId;

    private Integer userId;

    private String ip;

    private Date loginDate;
}
```

使用了`lombok`来减少模版代码，因此需要添加依赖：

```xml
      <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.4</version>
        <scope>provided</scope>
      </dependency>
```
添加`spring`依赖：

```java
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.1.3.RELEASE</version>
    </dependency>
```

#### 数据访问层

##### 添加相关依赖

因此首先添加`mysql`依赖：

```xml
    <!--添加mysql driver-->
    <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.44</version>
    </dependency>
```

使用`DBPC`实现的数据源，添加`DBPC`依赖：

```xml
    <!-- https://mvnrepository.com/artifact/commons-dbcp/commons-dbcp -->
    <dependency>
      <groupId>commons-dbcp</groupId>
      <artifactId>commons-dbcp</artifactId>
      <version>1.4</version>
    </dependency>
```

使用`Spring-JDBC`框架

```xml
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.1.3.RELEASE</version>
    </dependency>
```

##### Dao代码编写

查看[官方文档](https://docs.spring.io/spring-framework/docs/5.1.3.RELEASE/spring-framework-reference/core.html#beans-introduction)，发现`@Repository`用于将数据访问层 (`DAO`层 ) 的类标识为`Spring Bean`。

> The `@Repository` annotation is a marker for any class that fulfills the role or stereotype of a repository (also known as Data Access Object or DAO). 

```java
package com.flwcy.sl.dao;

import com.flwcy.sl.entity.User;
import lombok.Data;
import lombok.Setter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Repository;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

/**
 * @Description: UserDao
 * @author: flwcy
 * @date: 2019/1/2 10:16
 */
@Repository
public class UserDao {

    private static final String SELECT_BY_USER_NAME_SQL = "select * from t_user where user_name = ?";

    private static final String UPDATE_USER_SQL = "update t_user set user_name=?,password=?,crodits=?,last_ip=?,last_visit=? where user_id=?;";

    private static final String INSERT_USER_SQL = "insert into t_user(user_name,`password`,crodits,last_ip,last_visit) values(?,?,?,?,?);";

    @Autowired
    private JdbcTemplate jdbcTemplate;

    /**
     * @Author: flwcy
     * @Description: 根据用户名查询
     * @Date: 2019/1/2 10:35
     * @Param:
     * @return:
     */
    public User selectByUserName(final String userName){
        List<User> users = jdbcTemplate.query(SELECT_BY_USER_NAME_SQL, new Object[]{userName}, new RowMapper<User>() {
            @Override
            public  User mapRow(ResultSet rs, int rows) throws SQLException {
                User u = new User();
                u.setUserId(rs.getInt("user_id"));
                u.setUserName(userName);
                u.setPassword(rs.getString("password"));
                u.setCrodits(rs.getInt("crodits"));
                return u;
            }
        });

        return users.size() != 0 ? users.get(0) : null;
    }

    public int insert(User user) {
        int result = 0;
        if(user != null) {
            result = jdbcTemplate.update(INSERT_USER_SQL,new Object[]{user.getUserName(),user.getPassword(),user.getCrodits(),user.getLastIp(),user.getLastVisit()});
        }
        return result;
    }

    public int update(User user) {
        int result = 0;
        if(user != null) {
            result = jdbcTemplate.update(UPDATE_USER_SQL,new Object[]{user.getUserName(),user.getPassword(),user.getCrodits(),user.getLastIp(),user.getLastVisit(),user.getUserId()});
        }

        return result;
    }
}
```

同时，为了让`Spring`能够扫描类路径中的类并识别出`@Repository`注解，需要在`XML`配置文件中启用`Bean`的自动扫描功能，这可以通过`<context:component-scan/>`实现。添加`DBPC`实现的数据源，并配置`Spring JDBC`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 扫描包，将标注Spring注解的类自动转换成bean，同时完成bean的注入 -->
    <context:component-scan base-package="com.flwcy.sl.dao"></context:component-scan>
    <!-- 定义DBPC实现的数据源 -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
          destroy-method="close"
          p:driverClassName="com.mysql.jdbc.Driver"
          p:url="jdbc:mysql://localhost:3306/db_jdbc"
          p:username="root"
          p:password="123456"/>
    <!-- 定义JDBC模版bean -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate" p:dataSource-ref="dataSource">
        <!-- collaborators and configuration for this bean go here -->
    </bean>
</beans>
```

通过`xml`配置将`dataSource`注入`jdbcTemplate`中，而`jdbcTemplate`这个`bean`将通过`Autowired`自动注入到`UserDao`中。同理编写`LoginLogDao`的代码：

```java
package com.flwcy.sl.dao;

import com.flwcy.sl.entity.LoginLog;
import lombok.Setter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

/**
 * @Description:
 * @author: flwcy
 * @date: 2019/1/2 13:34
 */
@Repository
public class LoginLogDao {
    private static final String INSERT_LOGIN_LOG_SQL = "insert into t_login_log(user_id,ip,login_datetime) values(?,?,?);";

    @Autowired
    private JdbcTemplate jdbcTemplate;

    /**
     * @Author: flwcy
     * @Description: 插入一条登录记录
     * @Date: 2019/1/2 13:37
     * @Param:
     * @return:
     */
    public int insertLoginLog(LoginLog loginLog) {
        int result = 0;
        if(loginLog != null) {
            result = jdbcTemplate.update(INSERT_LOGIN_LOG_SQL,new Object[] {loginLog.getUserId(),loginLog.getIp(),loginLog.getLoginDate()});
        }
        return result;
    }
}
```

##### 测试

添加依赖：

```xml
 <!-- https://mvnrepository.com/artifact/junit/junit -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13-beta-1</version>
      <scope>test</scope>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-test -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>5.1.3.RELEASE</version>
      <scope>test</scope>
    </dependency>
```

编写`UserDao`的测试代码：

```java
package com.flwcy.sl.dao;

import com.flwcy.sl.entity.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.Date;

/**
 * @Description:
 * @author: flwcy
 * @date: 2019/1/2 13:38
 */
@RunWith(SpringJUnit4ClassRunner.class)//让测试运行于spring测试环境
@ContextConfiguration(locations="classpath:spring-context.xml")//指定 Spring 配置文件所在的位置
public class UserDaoTest {

    @Autowired
    UserDao userDao;

    @Test
    public void selectByUserName(){
        User u = userDao.selectByUserName("flwcy1");
        System.out.println(u);
    }

    @Test
    public void insert(){
        User user = new User();
        user.setUserName("flwcy");
        user.setPassword("123456");
        user.setCrodits(0);
        user.setLastIp("127.0.0.1");
        user.setLastVisit(new Date());
        userDao.insert(user);
    }

    @Test
    public void update(){
        User user = new User();
        user.setUserName("flwcy");
        user.setPassword("123456");
        user.setCrodits(5);
        user.setUserId(1);
        user.setLastIp("127.0.0.1");
        user.setLastVisit(new Date());
        userDao.update(user);
    }
}
```

编写`LoginLogDao`的测试代码：

```java
package com.flwcy.sl.dao;

import com.flwcy.sl.entity.LoginLog;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.Date;

/**
 * @Description:
 * @author: flwcy
 * @date: 2019/1/2 13:39
 */
@RunWith(SpringJUnit4ClassRunner.class)//让测试运行于spring测试环境
@ContextConfiguration(locations="classpath:spring-context.xml")//指定 Spring 配置文件所在的位置
public class LoginLogDaoTest {

    @Autowired
    LoginLogDao loginLogDao;

    @Test
    public void insert(){
        LoginLog loginLog = new LoginLog();
        loginLog.setIp("127.0.0.1");
        loginLog.setUserId(1);
        loginLog.setLoginDate(new Date());

        loginLogDao.insertLoginLog(loginLog);
    }
}
```

#### 业务层

##### service层代码编写

```java
package com.flwcy.sl.service;

import com.flwcy.sl.dao.LoginLogDao;
import com.flwcy.sl.dao.UserDao;
import com.flwcy.sl.entity.LoginLog;
import com.flwcy.sl.entity.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * @Description:
 * @author: flwcy
 * @date: 2019/1/2 15:31
 */
@Service
public class UserService {

    @Autowired
    private UserDao userDao;

    @Autowired
    private LoginLogDao loginLogDao;

    public User selectByUserName(String userName){
        return userDao.selectByUserName(userName);
    }

    @Transactional(rollbackFor = Exception.class)
    public void loginSuccess(User user) {
        user.setCrodits(user.getCrodits() + 5);
        userDao.update(user);
        LoginLog loginLog = new LoginLog();
        loginLog.setUserId(user.getUserId());
        loginLog.setIp(user.getLastIp());
        loginLog.setLoginDate(user.getLastVisit());
        loginLogDao.insertLoginLog(loginLog);
    }

}
```

> 引用[spring的官方文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-stereotype-annotations)中的一段描述：
>
> 在Spring2.0之前的版本中，`@Repository`注解可以标记在任何的类上，用来表明该类是用来执行与数据库相关的操作（即dao对象），并支持自动处理数据库操作产生的异常
>
> 在Spring2.5版本中，引入了更多的Spring类注解：`@Component`,`@Service`,`@Controller`。`Component`是一个通用的Spring容器管理的单例bean组件。而`@Repository`, `@Service`, `@Controller`就是针对不同的使用场景所采取的特定功能化的注解组件。
>
> 因此，当你的一个类被`@Component`所注解，那么就意味着同样可以用`@Repository`, `@Service`, `@Controller`来替代它，同时这些注解会具备有更多的功能，而且功能各异。
>
> 最后，如果你不知道要在项目的业务层采用`@Service`还是`@Component`注解。那么，`@Service`是一个更好的选择。
>
> | 注解        | 含义                                         |
> | ----------- | -------------------------------------------- |
> | @Component  | 最普通的组件，可以被注入到spring容器进行管理 |
> | @Repository | 作用于持久层                                 |
> | @Service    | 作用于业务逻辑层                             |
> | @Controller | 作用于表现层（spring-mvc的注解）             |

##### 事务管理配置

在`service`类的方法上加上`@Transactional`，声明这个方法需要事务管理。首先引入相关依赖：

```xml
    <!-- https://mvnrepository.com/artifact/org.aspectj/aspectjrt -->
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjrt</artifactId>
      <version>1.9.2</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.9.2</version>
    </dependency>
```

引入`aop`及`tx`命名空间所对应的`schema`文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="...
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

```

扫描`service`包，将标注`Spring`注解的类自动转换成`bean`，同时完成`bean`的注入。

```xml
<context:component-scan base-package="com.flwcy.sl.service"></context:component-scan>
```

配置事务：

```xml
    <!-- 配置事务管理器 -->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager" p:dataSource-ref="dataSource">
    </bean>

    <!-- 定义哪些方法需要被事务管理器进行管理 -->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <tx:attributes>
            <tx:method name="*" />
        </tx:attributes>
    </tx:advice>

    <!-- 需要哪些方法是被监控,并且是有事务管理 -->
    <aop:config>
        <!--声明所有包含Service的类的所有方法使用事务-->
        <aop:pointcut id="service" expression="execution(* com.flwcy.sl..*Service*.*(..))" />
        <!-- 代表的意思: service包下的说有类以Service结尾的类下的所有方法,都为只读状态 -->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="service" />
    </aop:config>
```

##### 测试

`userService`测试代码如下：

```java
package com.flwcy.sl.service;

import com.flwcy.sl.entity.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.Date;

/**
 * @Description:
 * @author: flwcy
 * @date: 2019/1/2 17:54
 */
@RunWith(SpringJUnit4ClassRunner.class)//让测试运行于spring测试环境
@ContextConfiguration(locations="classpath:spring-context.xml")//指定 Spring 配置文件所在的位置
public class UserServiceTest {

    @Autowired
    UserService userService;

    @Test
    public void loginSuccess(){
        User user = new User();
        user.setUserName("flwcy");
        user.setPassword("123456");
        user.setCrodits(15);
        user.setUserId(1);
        user.setLastIp("127.0.0.1");
        user.setLastVisit(new Date());
        userService.loginSuccess(user);
    }
}
```

#### 展示层

##### 配置Spring MVC框架

首先需要对`web.xml`进行配置，以便`Web`容器在启动时能够自动启动Spring容器，参考[官方文档](https://docs.spring.io/spring-framework/docs/5.1.3.RELEASE/spring-framework-reference/core.html#context-create)进行配置：

```xml
    <!-- 从类路径下加载Spring配置文件 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:spring-context.xml</param-value>
    </context-param>
    <!-- 负责启动Spring容器的监听器，引用contextConfigLocation中配置的文件地址 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
```

需要引入相关依赖：

```xml
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-web -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>5.1.3.RELEASE</version>
    </dependency>
```

在`Web MVC`架构中，用户并不是直接访问所需资源，而是先由前端控制器（`Front Controller`）来判断请求要分派（`Dispatch`）给哪一个控制器（`Controller`）来处理，在`Spring`的`Web MVC`框架中，担任前端控制器角色的是`org.springframework.web.servlet.DispatcherServlet`，所以使用`Spring Web MVC`的第一步，就是在`web.xml`中定义`DispatcherServlet`，配置文件参考[官方文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#spring-web)：

```xml
    <!-- 这个Servlet是实现Spring mvc 的前端控制器，所有的Web请求都需要通过它来处理，进行匹配、转发、数据处理。 -->
    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!-- 处理url -->
    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>.html</url-pattern>
    </servlet-mapping>
```

引入依赖

```xml
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.1.3.RELEASE</version>
    </dependency>
```

##### Controller层代码编写




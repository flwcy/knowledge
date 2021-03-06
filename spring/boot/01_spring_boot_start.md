### Spring Boot入门篇

`Spring Boot`是由`Pivotal`团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。

#### 创建Spring Boot项目

首先我们使用idea创建一个`Spring boot`项目。`idea--->File--->New--->Project---> Spring Initializr`。初始创建的包结构如下：

```
com
 +- flwcy
     +- boot
         +- Application.java
```

`pom.xml`配置可参考[官方文档](https://docs.spring.io/spring-boot/docs/2.2.0.BUILD-SNAPSHOT/reference/html/getting-started.html#getting-started-first-application)，`Application.java`文件将声明`main`方法以及基本的`@SpringBootApplication`，如下所示：

```java
package com.flwcy.boot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```

#### Hello World

现在我们来编写一个`Hello World`的简单程序，代码如下：

```java
package com.flwcy.boot.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @RequestMapping(value = "/sayHello",method = RequestMethod.GET)
    public String sayHello(){
        return "Hello Spring Boot";
    }
}
```

启动`Application`的`main`方法，我们可以看到控制台输出的日志：

```
2019-01-15 22:32:06.030  INFO 7112 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
```

因此我们可以在浏览器中输入`http://localhost:8080/sayHello`来查看结果。

#### 配置文件

`Spring Boot`使用一个全局配置文件，一般是`application.properties`或者`application.yml`，推荐使用`YAML`格式。

```
login:
  user:
    userName: flwcy
    age: 18
    gender: male
```

那么我们如何在代码中获取配置文件中的值呢？

```java
    @Value("${login.user.userName}")
    private String userName;
```

假如我们要获取多个属性呢？是否需要将上述代码重复多遍呢，答案是否定的，参考[官方文档](https://docs.spring.io/spring-boot/docs/2.2.0.BUILD-SNAPSHOT/reference/html/spring-boot-features.html#boot-features-external-config-yaml)，我们通过如下方式来实现：

```java
package com.flwcy.boot.dto;

import lombok.Data;
import lombok.Getter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@ConfigurationProperties(prefix = "login.user")
@Data
@Component
public class User {

    private String userName;

    private Integer age;

    private String gender;
}
```

使用

```java
    @Autowired
    private User user;
```

这样我们就可以通过`user.getUserName()`来获取配置文件配置的用户名。

#### Controller的使用

`@Controller`用于将类标记为`Spring Mvc Controller`对象。`@RestController`是一个组合注解，等同于`@Controller`和`@RequestBody`。因此以下两个`controller`定义应该相同：

```java
@Controller
@ResponseBody
public class MyController { }

@RestController
public class MyRestController { }
```

##### 获取url中的参数

`@PathVariable`：获取`url`中的的数据。

```java
    @RequestMapping(value = "/getUserName/{userName}",method = RequestMethod.GET)
    public String getUserName(@PathVariable("userName") String userName){
        return "hi," + userName;
    }
```

在浏览器中输入`http://localhost:8080/getUserName/flwcy`。

`@RequestParam`：获取请求参数中的值。

```java
    @RequestMapping(value = "/getInfo",method = RequestMethod.GET)
    public String getInfo(@RequestParam(value = "userName",required = true,defaultValue = "") String userName){
        return "hi," + userName;
    }
```

在浏览器中输入`http://localhost:8080/getInfo?userName=flwcy`。

组合注解：`@GetMapping`，`@PostMapping`，`@PutMapping`等。`@GetMapping("/getInfo")`等同于`@RequestMapping(value = "/getInfo",method = RequestMethod.GET)`，减少了我们的代码量。

#### 数据库操作

假如有这样一张数据表

```sql
DROP TABLE IF EXISTS `db_user`;
CREATE TABLE `db_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_name` varchar(50) NOT NULL,
  `password` varchar(32) NOT NULL,
  `email` varchar(50) DEFAULT NULL,
  `birthday` date DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

`JPA`：定义了一系列对象持久化的标准。`Spring Data JPA`是`Spring`基于`Hibernate`开发的一个`JPA`框架。

首先需要在`pom.xml`中添加相关依赖：

```xml
		<!-- mysql依赖 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<!-- Spring data JPA -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
```

在配置文件中添加配置：

```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
  datasource:
    url: jdbc:mysql://localhost:3306/db_jdbc
    data-username: root
    data-password: 123456
    driver-class-name: com.mysql.jdbc.Driver
```

`spring.jpa.show-sql`：显示`sql`语句

`spring.jpa.hibernate.ddl-auto`：`update`更新数据库时不删除旧表，`create`每次加载`hibernate`时都会删除上一次的生成的表，然后根据你的实体类重新生成新表。

参考[官方文档](https://docs.spring.io/spring-boot/docs/2.2.0.BUILD-SNAPSHOT/reference/html/spring-boot-features.html#boot-features-entity-classes)，新建`User`类

```java
package com.flwcy.boot.entity;

import lombok.Data;
import org.springframework.boot.autoconfigure.domain.EntityScan;

import javax.persistence.Entity;
import javax.persistence.Id;
import java.util.Date;

@Entity(name="db_user")
@Data
public class User {

    @Id
    @GeneratedValue
    private Integer id;

    private String userName;

    private String password;
    
    private String email;

    private Date birthday;
}
```

对于数据库的增删改查操作，定义`UserRepository`接口，继承`JpaRepository`，此接口是`Spring-Data-Jpa`内部定义好的泛型接口，第一个参数实体类，第二个参数是`ID`。

```java
package com.flwcy.boot.repository;


import com.flwcy.boot.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User,Integer> {
}
```

#### RESTful API

我们对其设计`RESTful API`，具体如下：

|     请求路径     | 请求类型 |      功能描述      |
| :--------------: | :------: | :----------------: |
|    /listUsers    |   get    |  获取所有用户信息  |
|  /getUser/{id}   |   get    | 通过id查询一个用户 |
|     /addUser     |   post   |    创建一个用户    |
| /updateUser/{id} |   put    | 通过id修改用户信息 |
| /deleteUser/{id} |  delete  | 通过id删除一个用户 |

接下来编写对应`API`接口代码：

```java
package com.flwcy.boot.controller;

import com.flwcy.boot.entity.User;
import com.flwcy.boot.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.Date;
import java.util.List;

@RestController //处理http请求，返回json格式
@RequestMapping("/user")//配置url，让该类下的所有接口url都映射在/user下
public class UserController {

    @Autowired
    UserRepository userRepository;

    @GetMapping("/selectAll")
    public List<User> selectAll(){
        return userRepository.findAll();
    }

    @GetMapping("/getUser/{id}")
    public User getUser(@PathVariable("id") Integer id) {
        return userRepository.findById(id).get();
    }

    @PostMapping("/addUser")
    public User addUser(@RequestParam("userName") String userName,
                          @RequestParam("password") String password, @RequestParam("email") String email) {
        User u = new User();
        u.setUserName(userName);
        u.setPassword(password);
        u.setEmail(email);
        return userRepository.save(u);
    }

    @PutMapping("updateUser")
    public User updateUser(@RequestParam("id") Integer id, @RequestParam("userName") String userName,
                           @RequestParam("password") String password, @RequestParam("email") String email){
        User u = new User();
        u.setId(id);
        u.setUserName(userName);
        u.setPassword(password);
        u.setEmail(email);
        return userRepository.save(u);
    }

    @DeleteMapping("/deleteUser/{id}")
    public void deleteUser(@PathVariable("id")Integer id) {
        userRepository.deleteById(id);
    }
}
```

参考[官方文档](https://docs.spring.io/spring-data/jpa/docs/2.2.0.M1/reference/html/#jpa.query-methods.query-creation)，我们也可以自定义查询方法，自定义查询就是根据方法名来自动生成`SQL`，主要的语法是`findXXBy,readAXXBy,queryXXBy,countXXBy, getXXBy`后面跟属性名称，例如：

```java
 List<User> findByUserName(String userName);
```

#### 事务管理

为什么需要事务？例如一个转账系统，用户A给用户B转账1000元，分成两步完成：

+ 用户A帐号减少1000元
+ 用户B帐号增加1000元

假如在第一步的时候服务出现了异常，用户A的钱被扣掉了，而用户B却并没有收到钱。事务就是要确保以上两步（或者整个“事务”）都能完成，或者一起取消。

在`Spring Boot`中，当我们使用了`spring-boot-starter-data-jpa`依赖的时候，框架会自动默认注入`JpaTransactionManager`。所以我们不需要任何额外配置就可以用`@Transactional`注解进行事务的使用。

假如数据库要求用户名唯一

```sql
ALTER TABLE `db_user` ADD unique(`user_name`);
```

假如我们在一次操作中添加了两个相同用户名的用户。

```java
    public void addTwo(){
        User u = new User();
        u.setUserName("flwcy1");
        u.setPassword("123456");
        u.setEmail("flwcy1@qwe.com");
        userRepository.save(u);
        User _u = new User();
        _u.setUserName("flwcy1");
        _u.setPassword("123456");
        _u.setEmail("flwcy1@qwe.com");
        userRepository.save(_u);
    }
```

执行测试用例，会发现控制台抛出异常，此时查询数据库，发现第一条记录插入成功，第二条记录未插入。而我们希望保证完整性操作，希望在发生异常的时候被回退。在方法上加上`@Transactional`注解即可。

##### 事务的隔离级别

隔离级别是指若干个并发的事务之间的隔离程度，与我们开发时候主要相关的场景包括：脏读取、重复读、幻读。

+ 脏读：一个事务读到了另一个事务的未提交的数据 。
+ 不可重复读：一个事务读到了另一个事务已经提交的`update`的数据导致多次查询结果不一致。
+ 幻读：一个事务读到了另一个事务已经提交的`insert`的数据导致多次查询结果不一致。

我们可以看`org.springframework.transaction.annotation.Isolation`枚举类中定义了五个表示隔离级别的值：

```java
public enum Isolation {
    DEFAULT(-1),
    READ_UNCOMMITTED(1),
    READ_COMMITTED(2),
    REPEATABLE_READ(4),
    SERIALIZABLE(8);
}
```

- `DEFAULT`：这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是：`READ_COMMITTED`。
- `READ_UNCOMMITTED`：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读和不可重复读，因此很少使用该隔离级别。
- `READ_COMMITTED`：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
- `REPEATABLE_READ`：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。即使在多次查询之间有新增的数据满足该查询，这些新增的记录也会被忽略。该级别可以防止脏读和不可重复读。
- `SERIALIZABLE`：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

指定方法：通过使用`isolation`属性设置，例如：

```java
@Transactional(isolation = Isolation.DEFAULT)
```

##### 传播行为

所谓事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。

我们可以看`org.springframework.transaction.annotation.Propagation`枚举类中定义了6个表示传播行为的枚举值：

```java
public enum Propagation {
    REQUIRED(0),
    SUPPORTS(1),
    MANDATORY(2),
    REQUIRES_NEW(3),
    NOT_SUPPORTED(4),
    NEVER(5),
    NESTED(6);
}
```

- `REQUIRED`：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- `SUPPORTS`：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- `MANDATORY`：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
- `REQUIRES_NEW`：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- `NOT_SUPPORTED`：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- `NEVER`：以非事务方式运行，如果当前存在事务，则抛出异常。
- `NESTED`：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于`REQUIRED`。

指定方法：通过使用`propagation`属性设置，例如：

```java
@Transactional(propagation = Propagation.REQUIRED)
```

#### 参考文章

[Spring Boot中的事务管理](http://blog.didispace.com/springboottransactional/)
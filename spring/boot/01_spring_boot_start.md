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

`Application.java`文件将声明`main`方法以及基本的`@SpringBootApplication`，如下所示：

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

假如我们要获取多个属性呢？是否需要将上述代码重复多遍呢，答案是否定的，我们通过如下方式来实现：

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


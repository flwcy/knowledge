### Spring Boot进阶篇

#### 表单验证

在进行`Web`开发的时候，通常会有表单提交操作，虽然前端也会进行验证，但是出于安全性考虑，服务端也需要进行表单验证。

首先需要添加依赖：

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-validation</artifactId>
		</dependency>
```

修改`User`类：

```java
package com.flwcy.boot.entity;

import lombok.Data;
import org.springframework.boot.autoconfigure.domain.EntityScan;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotEmpty;
import java.util.Date;

@Entity(name="db_user")
@Data
public class User {

    @Id
    @GeneratedValue
    private Integer id;

    /**
     * 用户名
     */
    @NotEmpty(message="用户名不能为空")
    private String userName;

    /**
     * 密码
     */
    @Size(min=6,message="密码长度不能低于6位")
    private String password;

    /**
     * 邮箱
     */
    @NotEmpty(message="邮箱不能为空")
    private String email;

    /**
     * 生日
     */
    private Date birthday;
}
```

然后在`controller`中使用：

```java
    @PostMapping("/addUser")
    public User addUser(@Validated User user, BindingResult bindingResult) {
        if(bindingResult.hasErrors()) {
            log.info(bindingResult.getFieldError().getDefaultMessage());
            return null;
        }
        return userRepository.save(user);
    }
```

#### AOP统一处理请求日志

`AOP`是一种编程范式，是一种程序设计思想，与具体的计算机编程语言无关，所以不止是`Java`，像`.Net`等其他编程语言也有`AOP`的实现方式。`AOP`的思想理念就是将通用逻辑从业务逻辑中分离出来。

在项目开发中，我们通常需要打印用户得请求参数以及接口返回的数据，如果在每个接口中都写一遍如下代码：

```java
log.info("getUser accepted parameter:{}",id);
log.info("getUser return the data:{}",user);
```

这样就有很多重复代码了，因此我们可以使用`AOP`来统一处理请求日志。首先定义切面：

```java
package com.flwcy.boot.aspect;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;

@Aspect
@Slf4j
@Component
public class LogAspect {

    @Before("execution(public * com.flwcy.boot.controller.*.*(..))")
    public void doBefore(JoinPoint joinPoint){
        ServletRequestAttributes servletRequestAttributes = (ServletRequestAttributes)RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = servletRequestAttributes.getRequest();
        // url
        log.info("url->{}",request.getRequestURL());
        // ip
        log.info("IP->{}",request.getRemoteAddr());
        //class_method
        log.info("class_method->{}.{}",joinPoint.getSignature().getDeclaringTypeName(),joinPoint.getSignature().getName());
        //args
        StringBuilder sb = new StringBuilder();
        for(int i=0;i<joinPoint.getArgs().length;i++){
            sb.append(joinPoint.getArgs()[i]);
            if(i != joinPoint.getArgs().length - 1) {
                sb.append(",");
            }
        }
        log.info("args->{}",sb.toString());
    }

    @AfterReturning(returning = "object",pointcut = "execution(public * com.flwcy.boot.controller.*.*(..))")
    public void doAfterReturning(JoinPoint joinPoint,Object object){
        log.info("{}.{} response->{}",joinPoint.getSignature().getDeclaringTypeName(),joinPoint.getSignature().getName(),object);
    }
}
```

我们发现`pointcut`代码重复了，修改成如下代码：

```java
    @Pointcut("execution(public * com.flwcy.boot.controller.*.*(..))")
    public void log(){}
    
    @Before("log()")
```

#### API接口返回格式统一

`API`接口返回得数据格式应该统一，便于接口调用者获取数据后解析。因此首先定义统一格式`RetResult`：

```java
package com.flwcy.boot.core;

import lombok.Data;

@Data
public class RetResult<T> {

    /**
     * 错误码
     */
    private Integer code;

    /**
     * 提示信息
     */
    private String msg;

    /**
     *具体的内容
     */
    private T data;

    public RetResult(T data) {
        this.data = data;
    }

    public RetResult(ResultEnum resultEnum) {
        this.code = resultEnum.getCode();
        this.msg = resultEnum.getMsg();
    }

    public RetResult(Integer code ,String msg) {
        this.code = code;
        this.msg = msg;
    }

    public RetResult(Integer code ,String msg, T data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }
}
```

接下来将返回结果转换为封装后的对象：

```java
package com.flwcy.boot.utils;

import com.flwcy.boot.core.ResultEnum;
import com.flwcy.boot.core.RetResult;

public class ResultUtil {

    public static <T> RetResult<T> success(Object data){
        RetResult retResult = new RetResult(ResultEnum.SUCCESS.getCode(),ResultEnum.SUCCESS.getMsg(),data);
        return retResult;
    }

    public static <T> RetResult<T> success(){
        return success(null);
    }

    public static <T> RetResult<T> error(String msg){
        RetResult retResult = new RetResult(ResultEnum.FAIL.getCode(),msg);
        return retResult;
    }

    public static <T> RetResult<T> error(Integer code,String msg){
        RetResult retResult = new RetResult(code,msg);
        return retResult;
    }
}
```

修改`controller`中的代码：

```java
    @PostMapping("/addUser")
    public RetResult<User> addUser(@Validated User user, BindingResult bindingResult) {
        if(bindingResult.hasErrors()) {
            log.info(bindingResult.getFieldError().getDefaultMessage());
            return ResultUtil.error(bindingResult.getFieldError().getDefaultMessage());
        }
        return ResultUtil.success(userService.save(user));
    }
```

最终返回格式如下：

```json
{
    "code": 200,
    "data": {
        "email": "qwer123@1qwe.com",
        "id": 12,
        "password": "qwerq1123",
        "userName": "qwer12"
    },
    "msg": "success"
}
```

#### 统一异常处理

实际项目开发中， 程序往往会出现各式各样的`Bug`，如果直接将错误的信息直接暴露给用户，这样的体验可想而知。所以我们需要对异常进行捕获，然后给予相应的处理，来减少程序异常对用户体验的影响。

在应用开发过程中，通常需要进行一些业务判断，除系统自身的异常外，不同业务场景中用到的异常也不一样，因此定义一个业务类异常（`Spring`只对`RuntimeException`进行事务回滚）：

```java
package com.flwcy.boot.core;

import lombok.Data;

import java.io.Serializable;

@Data
public class ServiceException extends RuntimeException implements Serializable {

    private static final long serialVersionUID = 1213855733833039552L;

    private Integer code;

    public ServiceException() {
        super();
    }

    public ServiceException(String message) {
        super(message);
    }

    public ServiceException(ResultEnum resultEnum) {
        super(resultEnum.getMsg());
        this.code = resultEnum.getCode();
    }

    public ServiceException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

此时我们可以进行统一异常处理：

```java
package com.flwcy.boot.core;

import com.flwcy.boot.utils.ResultUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletResponse;

@ControllerAdvice
public class GlobalExceptionResolver {

    private final Logger logger = LoggerFactory.getLogger(GlobalExceptionResolver.class);

    /**
     * 异常统一处理
     */
    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public RetResult exceptionHandler(Exception e) {
        if(e instanceof ServiceException) {
            ServiceException serviceException = (ServiceException) e;
            return ResultUtil.error(serviceException.getCode() == null ? ResultEnum.FAIL.getCode() : serviceException.getCode(),serviceException.getMessage());
        }
        RetResult<Object> result = new RetResult<>(ResultEnum.INTERNAL_SERVER_ERROR);
        logger.error(e.getMessage(), e);
        return result;
    }
}

```

上述代码中，我们的状态码与提示信息并没有关联起来，为了防止状态码重复，定义一个枚举类型来统一管理：

```java
package com.flwcy.boot.core;
import lombok.Getter;

@Getter
public enum ResultEnum {

    // 成功
    SUCCESS(200,"成功"),

    // 失败
    FAIL(400,"失败"),

    // 未认证（签名错误）
    UNAUTHORIZED(401,"签名错误"),

    // 接口不存在
    NOT_FOUND(404,"接口不存在"),

    // 服务器内部错误
    INTERNAL_SERVER_ERROR(500,"服务器打酱油了，请稍后再试~");

    private Integer code;

    private String msg;

    ResultEnum(Integer code,String msg) {
        this.code = code;
        this.msg = msg;
    }
}
```

假如我们添加用户时，不允许添加用户名为`admin`的用户，代码如下：

```java
    public User save(User user) {
        if(user == null || "admin".equals(user.getUserName())) {
            throw new ServiceException("不允许添加该用户名的用户！");
        }
        return userRepository.save(user);
    }
```

此时`controller`层并不需要进行任何代码修改。

#### 单元测试

对于我们编写的代码需进行单元测试后才进行提交，对于一般的`service`层代码：

```java
package com.flwcy.boot.service;

import com.flwcy.boot.core.ServiceException;
import com.flwcy.boot.entity.User;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import java.util.List;

@RunWith(SpringRunner.class)
@SpringBootTest
public class UserServiceTest {

    @Autowired
    UserService userService;

    @Test
    public void findByUserName() {
        User u = userService.getUser(1);
        Assert.assertEquals("qwer",u.getUserName());
    }

}
```

对于`controller`我们应该使用




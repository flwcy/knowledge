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

#### AOP统一处理请求日志

#### 统一异常处理

##### 返回结果格式统一

##### 业务逻辑，统一异常处理

##### code message统一管理关联


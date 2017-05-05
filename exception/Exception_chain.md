# [采用异常链传递异常](http://blog.csdn.net/p106786860/article/details/11889327)

标签（空格分隔）： Exception

---

一、分析

 异常需要封装，但是仅仅封装还是不够的，还需要传递异常。一个系统的友好型的标识，友好的界面功能是一方面，另一方面就是系统出现非预期的情况的处理方式了。         
二、场景

  比如我们的JEE项目一般都又三层：持久层、逻辑层、展现层，持久层负责与数据库交互，逻辑层负责业务逻辑的实现，展现层负责UI数据的处理。
  有这样一个模块：用户第一次访问的时候，需要持久层从user.xml中读取数据，如果该文件不存在则提示用户创建之，那问题就来了：如果我们直接把持久层的异常FileNotFoundException抛弃掉，逻辑层根本无从得知发生任何事情，也就不能为展现层提供一个友好的处理结果，最终倒霉的就是展现层：没有办法提供异常信息，只能告诉用户“出错了，我也不知道出了什么错了”—毫无友好性而言。
          正确的做法是先封装，然后传递，过程如下：
          1.把FileNotFoundException封装为MyException。
          2.抛出到逻辑层，逻辑层根据异常代码（或者自定义的异常类型）确定后续处理逻辑，然后抛出到展现层。
          3.展现层自行确定展现什么，如果管理员则可以展现低层级的异常，如果是普通用户则展示封装后的异常。
          异常封装如下：
        
```java
public classIOException extends Exception{  
    //定义异常的原因  
    publicIOException(String message){  
        super(message);  
    }  
  
    //定义异常原因，并携带原始的异常  
    publicIOException(String message,Throwable cause){  
        super(message,cause);  
    }  
  
    //保留原始异常信息  
    publicIOExcepiton(Throwable cause){  
        super(cause);  
    }  
}  
```
  链中传递异常代码如下：
```java
try{  
    //DoSomethind  
}catch(Exceptione){  
    //这种形式也可以叫异常转译，调用者获得该异常后在调用getCause()方法即可获得Exception的异常信息，如此即可以方便查找异常的根本信息，便于解决问题。  
    thrownew IOException(e);  
}  
```
三、建议

   异常需要封装和传递，我们在进行系统开发的时候，不要“吞噬”异常，也不要“赤裸裸”的抛出异常，封装后在抛出，或者通过异常链传递，可以达到系统更健壮、友好的目的。





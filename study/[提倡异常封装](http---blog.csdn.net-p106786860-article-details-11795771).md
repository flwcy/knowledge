# [提倡异常封装](http://blog.csdn.net/p106786860/article/details/11795771)
标签（空格分隔）： Exception

---

一、分析 
Java语言的异常处理机制可以确保程序的健壮性，提高系统的开发效率，但是Java API提供的异常都是比较低级（这里的低级指的是“低级别的异常”），只有开发人员才能看的懂，才明白发生了什么问题。对于终端用户来说，这些异常基本上是天书，与业务无关，是纯计算机语言的描述。 
这就需要我们对异常进行封装了。 
二、场景 
异常封装有三方面的优点： 
1.提高系统的友好性 
例如，打开一个文件，如果文件不存在，则会报FileNotFoundException异常，如果该方法的编写不做任何处理，直接上抛上层，则会降低系统的友好性，代码如下所示： 
```java
public static void doStuff()throws Exception{   
    InputStream is = new FileInputStream("无效文件.txt");   
    /*文件操作*/   
}  
```
此时doStuff方法的友好性极差：出现异常时（比如文件不存在），该方法直接把FileNotFoundException异常抛出到上层应用中（或者是终端用户），而上层应用（或用户）要么自己处理，要么接着抛出，最终的结果就是让用户对着“天书”式的文字发呆，用户不知道这是什么问题，只是系统告诉他"哦，我出错了，什么错误？你自己看着办吧"。 
解决办法就是封装异常，可以把系统的阅读者分为两类：开发人员和用户。开发人员查找问题，需要打印出堆栈信息，而用户则需要了解具体的业务原因，比如文件太大，不能同时编写文件等，代码如下： 
```java
public static void doStuff2()throws MyBussinessException{   
    try{   
        InputStream is = new FileInputStream("无效文件.txt");   
    }catch(FileNotFoundException e){   
        //为了方便开发和维护人员而设置的异常信息   
        e.printStackTree();   
        //抛出业务异常   
        throw new MyBussinessException(e);   
    }   
}   
```
2.提高系统的可维护性 
```java
public void doStuff(){   
    try{   
        //do something   
    }catch(Exception e){   
        e.printStackTrace();   
    }   
}   
```
这是很多程序员容易犯的错误，抛出异常是吧？分类处理多麻烦，就写一个catch块来处理所有异常吧。而且还信誓旦旦的说”JVM会打印出栈中的错误信息“，虽然这没有错，但是该信息只有开发人员自己才看的懂，维护人员看见这段异常基本上无法处理，因为需要深入到代码逻辑中去分析问题。 
正确的做法是对异常进行分类处理，并进行封装输出，代码如下： 
```java
public void doStuff(){   
try{   
    //do something   
    }catch(FileNotFoundException e){   
        log.info("文件夹未找到，使用默认配置文件....");   
    }catch(SecurityException 3){   
        log.info("无权访问，可能原因是....");   
        e.printStackTrace();   
    }   
}  
```
如此包装后，维护人员看到这样子的异常就有了初步的判断，或者检查配置，或者初始化环境，不需要直接到代码层级去分析了。 
 
3.解决Java异常机制自身的缺陷 
Java中的异常一次只能抛出一个，比如，doStuff方法有两个逻辑代码片段，如果在第一个逻辑片段中抛出异常，则第二个逻辑片段就不执行了，也就无法抛出第二个异常了。那么如何才能一次抛出两个异常呢？ 
其实，使用自行封装的异常可以解决问题，代码如下： 
```java
class MyException extends Exception{   
    //容纳所有异常   
    private List<Throwable> causes = new ArrayList<Throwable>();   
    //构造函数，传递一个异常列表   
    public MyException(List<? extends Throwable> _causes){   
        cause.addAll(_causes);   
    }   
   
    //读取所有的异常   
    public List<Throwable> getException(){   
        return causes;   
    }   
}  
```
MyException异常只是一个异常容器，可以容纳多个异常，但它本身并不代表任何异常含义，它所解决的是一次抛出多个异常的问题，具体调用如下： 
```java
public static void doStuff()throws MyException{   
    List<Throwable> list = new ArrayList<Throwable>();   
    //第一个逻辑片段   
    try{   
        //Do something   
    }catch(Exception e){   
        list.add(e);   
    }   
   
    //第二个逻辑片段   
    try{   
        //Do something   
    }catch(Exception e){   
        list.add(e);   
    }   
   
    //检查是否有必要抛出异常   
    if(list.size() > 0){   
        throw new MyException(list);   
    }   
}   
```
这样一来，doStuff方法的调用者就可以一次获得多个异常，也能够为用户提供完整的例外情况说明。 
那么在什么情况下，需要一个方法抛出多个异常呢？比如Web界面注册时，展示层依次把User对象传递给逻辑层，Register方法需要对各个Field进行校验并注册，例如，用户名不能重复，密码必须符合密码策略等，不要出现第一次提交系统提示”用户名重复“，在修改在提交后，系统提示“密码长度少6位”的情况，这种操作模式用户体验非常糟糕，最好的解决办法就是封装异常，建立异常容器，一次性地对User对象进行校验，然后返回所有异常。 
 
三、建议 
在开发的过程中，根据具体的情况和需要，对异常进行封装。





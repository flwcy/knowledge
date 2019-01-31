### 代理模式

代理模式的作用：为其他对象提供一种代理，以控制对这个对象的访问。

在某些情况下，一个客户不想或不能直接引用另一个对象，而代理对象可以在客户端和目标之间起到**中介**的作用。

举个简单的例子来说明代理的作用：代理模式就像VPN，你无法直接访问Facebook，而是通过了一个你可以访问它且它可以访问Facebook的服务器代理你去访问，并把数据给你。VPN可以帮你完成想要完成的事（访问Facebook），还可以增加自己想要做的事情（统计本次所用流量）。这就是代理模式在现实中的一个例子。


>装饰模式正如其名，就好像你买了一个手机套，但这个手机套不能完全满足你的要求，于是你在手机套上刻上了你的大名，这样就满足了你的要求了。但这个手机套从核心上来说，还是你之前的手机套。
>
>[Java中“装饰模式”和“代理模式”有啥区别？ - CodeTYOUKYO的回答 - 知乎](https://www.zhihu.com/question/41988550/answer/932511380)

#### 一些概念

代理模式中涉及到的角色有：

+ **抽象角色**：声明真实对象和代理对象的共同接口。
+ **代理角色**：代理对象角色内部含有对真实对象的引用，从而可以操作真实对象，同时代理对象提供与真实对象相同的接口以便在任何时刻都能代替真实对象。同时，代理对象可以在执行真实对象操作时，附加其他的操作，相当于对真实对象进行封装。
+ **真实角色**：代理角色所代表的真实对象，是我们最终要引用的对象。

UML类图 & 组成：

![proxy_pattern_01](../img/design_pattern/proxy_pattern_01.jpg)

#### 静态代理

我们写代码来模拟VPN访问Facebook的这个过程：

抽象角色接口：提供了访问Facebook的操作

```java
package com.flwcy.proxy;

/**
 * 抽象角色：提供代理角色和真实角色对外提供的公共方法
 *
 */
public interface Subject {

    /**
     * 访问Fackbook
     *
     */
    public void requestFacebook();
}

```

代理角色实现类：代理角色中代理了真实角色所需要的操作（访问Facebook）

```java
package com.flwcy.proxy;

/**
 * 代理角色（VPN）
 */
public class ProxySubject implements Subject {

    /**
     * 真实角色（网民）
     */
    private Subject user = new RealSubject(); //代理角色内部持有真实角色的引用

    public void requestFacebook() {
        // 获取Facebook的数据（真实角色操作之前所附加的操作）
        System.out.println("获取Facebook的数据");

        // 网民需要访问Facebook，只是代理
        user.requestFacebook();

        // 记录用户本次访问用了多少流量（真实角色操作之后所附加的操作）
        System.out.println("已用40M流量");
    }
}
```

真实角色实现类：这里的真实角色中其实只做了一个访问Facebook的操作，这是真实角色真正的业务逻辑部分

```java
package com.flwcy.proxy;

/**
 * 真实角色（网民）
 */
public class RealSubject implements Subject {

    public void requestFacebook() {
        System.out.println("访问Facebook");// 真实角色的操作：真正的业务逻辑
    }
}
```

测试代理类：

```java
    public static void main(String[] args) {
        // 就是一个代理对象
        ProxySubject vpn = new ProxySubject();

        // 通过VPN（代理对象）访问Facebook
        vpn.requestFacebook();//执行的是代理的方法
    }
```

**静态代理总结:**

1.可以做到在不修改目标对象的功能前提下,对目标功能扩展.

2.缺点:

- 在实际使用中，一个真实角色必须对应一个代理角色，如果大量使用会导致类的急剧膨胀。同时,一旦接口增加方法，目标对象与代理对象都要维护。

如何解决静态代理中的缺点呢？答案是可以使用动态代理方式

#### 动态代理

在java的动态代理机制中，有两个重要的类或接口，一个是`InvocationHandler(Interface)`，另一个则是 `Proxy(Class)`，这一个类和接口是实现我们动态代理所必须用到的。首先我们先来看看`java API`是怎么样对这两个类进行描述的：

`InvocationHandler`

```
InvocationHandler is the interface implemented by the invocation handler of a proxy instance.
Each proxy instance has an associated invocation handler. When a method is invoked on a proxy instance, the method invocation is encoded and dispatched to the invoke method of its invocation handler.
```

每一个动态代理类都必须要实现`InvocationHandler`这个接口，并且每个代理类的实例都关联到了一个`handler`，当我们通过代理对象调用一个方法的时候，这个方法的调用就会被转发为由`InvocationHandler`这个接口的`invoke`方法来进行调用。我们来看看`InvocationHandler`这个接口的唯一一个方法`invoke`方法：

```java
Object invoke(Object proxy, Method method, Object[] args) throws Throwable
```

invoke()方法有三个参数：

+ `proxy`参数是实现要代理接口的动态代理对象，通常情况下不需要它。
+ `Method`参数代表了被动态代理的接口中要调用的方法
+ `args`参数包含了被动态代理的方法需要的方法参数。注意基本类型(int,long)会被装箱成对象类型(Interger, Long)。

接下来我们来看看`Proxy`这个类：

```
Proxy provides static methods for creating dynamic proxy classes and instances, and it is also the superclass of all dynamic proxy classes created by those methods. 
```

Proxy这个类的作用就是用来动态创建一个代理对象的类，它提供了许多的方法，但是我们用的最多的就是`newProxyInstance`这个方法：

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,  InvocationHandler h)  throws IllegalArgumentException

Returns an instance of a proxy class for the specified interfaces that dispatches method invocations to the specified invocation handler.
```

`newProxyInstance()`方法有三个参数：

+ 类加载器`ClassLoader`用来加载动态代理类。
+ 一个要实现的接口的数组。
+ `InvocationHandler`接口。所有动态代理类的方法调用，都会交由`InvocationHandler`接口实现类里的`invoke()`方法去处理。这是动态代理的关键所在。



需要动态代理的接口：

```java
package com.flwcy.dynamicproxy;

/**
 * 抽象角色，真实角色与代理角色均需要实现该接口
 * 动态代理只能代理接口
 */
public interface Subject {

    public String sayHello(String name);

    public void request();
}
```

需要代理的真实角色：

```java
package com.flwcy.dynamicproxy;

/**
 * 真实角色
 */
public class RealSubject implements Subject {
    public String sayHello(String name) {
        return String.format("Hello,%s",name);
    }

    public void request() {
        System.out.println("From real subject.");
    }
}
```

我们就要定义一个动态代理类了，前面说过，每一个动态代理类都必须要实现`InvocationHandler`这个接口，因此我们这个动态代理类也不例外：

```java
package com.flwcy.dynamicproxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * 每一个动态代理类都必须要实现 InvocationHandler 这个接口
 */
public class InvocationHandlerImpl implements InvocationHandler {

    /**
     *  这个就是我们要代理的真实对象
     */
    private Object obj;//这是动态代理的好处，被封装的对象是Object类型，接受任意类型的对象

    public InvocationHandlerImpl(Object obj){
        this.obj = obj;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 在代理真实对象前我们可以添加一些自己的操作
        System.out.println("before calling:" + method);
        // 当代理对象调用真实对象的方法时，其会自动的跳转到代理对象关联的handler对象的invoke方法来进行调用
        Object result = method.invoke(obj, args);
        // 在代理真实对象后我们也可以添加一些自己的操作
        System.out.println("after calling:" + method);
        return result;
    }
}
```

测试：

```java
package com.flwcy.dynamicproxy;

import sun.misc.ProxyGenerator;

import java.io.*;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Modifier;
import java.lang.reflect.Proxy;

/**
 * 动态代理演示
 */
public class Client {

    public static void main(String[] args) {
        // 我们要代理的真实对象
        RealSubject realSubject = new RealSubject();
        /**
         * InvocationHandlerImpl 实现了 InvocationHandler 接口，并能实现方法调用从代理类到委托类的分派转发
         * 其内部通常包含指向委托类实例的引用，用于真正执行分派转发过来的方法调用.
         * 即：要代理哪个真实对象，就将该对象传进去，最后是通过该真实对象来调用其方法
         */
        InvocationHandler handler = new InvocationHandlerImpl(realSubject);
        // 通过Proxy的newProxyInstance方法来创建我们的代理对象
        Subject subject = (Subject) Proxy.newProxyInstance(handler.getClass().getClassLoader(),realSubject.getClass().getInterfaces(),handler);

        //这里可以通过运行结果证明subject是Proxy的一个实例，这个实例实现了Subject接口
        System.out.println(subject instanceof Proxy);

        //这里可以看出subject的Class类是$Proxy0,这个$Proxy0类继承了Proxy，实现了Subject接口
        System.out.println("subject的Class类是："+subject.getClass().toString());

        subject.request();
        System.out.println(subject.sayHello("flwcy"));

        // 将生成的字节码保存到本地
        createProxyClass(realSubject.getClass().getInterfaces());
    }

    private static void createProxyClass(Class<?>[] interfaces){
        String proxyName = "ProxySubject";
        BufferedOutputStream out = null;
        File file = new File(String.format("E:/tmp/%s.class",proxyName));

        /*
         * Look up or generate the designated proxy class.
         */
        byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
                proxyName, interfaces, Modifier.PUBLIC);

        try {
            out = new BufferedOutputStream(new FileOutputStream(file));
            out.write(proxyClassFile);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if(out != null)
                    out.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

输出结果如下：

```
true
subject的Class类是：class com.sun.proxy.$Proxy0
before calling:public abstract void com.flwcy.dynamicproxy.Subject.request()
From real subject.
after calling:public abstract void com.flwcy.dynamicproxy.Subject.request()
before calling:public abstract java.lang.String com.flwcy.dynamicproxy.Subject.sayHello(java.lang.String)
after calling:public abstract java.lang.String com.flwcy.dynamicproxy.Subject.sayHello(java.lang.String)
Hello,flwcy
```

执行`subject.sayHello("flwcy")`时，为什么会自动调用`InvocationHandlerImpl`的`invoke`方法？

```
因为JDK生成的最终真正的代理类，它继承自Proxy并实现了我们定义的Subject接口，在实现Subject接口方法的内部，通过反射调用了InvocationHandlerImpl的invoke方法。
```

查看`Proxy`类的静态方法`newProxyInstance`的源代码发现`JDK`会为我们生成真正的代理类，并实现接口中的方法（省略了反编译的过程）：

```java
public class ProxySubject extends Proxy
  implements Subject

...
  
public final String sayHello(String paramString)
    throws 
  {
    try
    {
      return (String)this.h.invoke(this, m4, new Object[] { paramString });
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }

...
  
  static
  {
    try
    {
      m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[] { Class.forName("java.lang.Object") });
      m4 = Class.forName("com.flwcy.dynamicproxy.Subject").getMethod("sayHello", new Class[] { Class.forName("java.lang.String") });
      m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
      m3 = Class.forName("com.flwcy.dynamicproxy.Subject").getMethod("request", new Class[0]);
      m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
      return;
    }
    catch (NoSuchMethodException localNoSuchMethodException)
    {
      throw new NoSuchMethodError(localNoSuchMethodException.getMessage());
    }
    catch (ClassNotFoundException localClassNotFoundException)
    {
      throw new NoClassDefFoundError(localClassNotFoundException.getMessage());
    }
  }  
```

为什么我们这里可以将其转化为`Subject`类型的对象？

```java
 Subject subject = (Subject) Proxy.newProxyInstance(handler.getClass().getClassLoader(),realSubject.getClass().getInterfaces(),handler);
```

所谓`DynamicProxy`是这样一种`class`：它是在**运行时**生成的`class`，在生成它时你必须提供一组`interface`给它，然后该`class`就宣称它实现了这些`interface`。你当然可以把该`class`的实例当作这些`interface`中的任何一个来用。当然，这个`DynamicProxy`其实就是一个`Proxy`，它不会替你作实质性的工作，在生成它的实例时你必须提供一个`handler`，由它接管实际的工作。

#### 总结

一个典型的动态代理创建对象过程可分为以下四个步骤：

1、通过实现`InvocationHandler`接口创建自己的调用处理器`IvocationHandler handler = new InvocationHandlerImpl(...);`

2、通过为`Proxy`类指定`ClassLoader`对象和一组`interface`创建动态代理类`Class clazz = Proxy.getProxyClass(classLoader,new Class[]{...});`

3、通过反射机制获取动态代理类的构造函数，其参数类型是调用处理器接口类型
`Constructor constructor = clazz.getConstructor(new Class[]{InvocationHandler.class});`

4、通过构造函数创建代理类实例，此时需将调用处理器对象作为参数被传入`Interface Proxy = (Interface)constructor.newInstance(new Object[] (handler));`

```java
// InvocationHandlerImpl 实现了 InvocationHandler 接口，并能实现方法调用从代理类到委托类的分派转发
// 其内部通常包含指向委托类实例的引用，用于真正执行分派转发过来的方法调用
InvocationHandler handler = new InvocationHandlerImpl(..); 
 
// 通过 Proxy 为包括 Interface 接口在内的一组接口动态创建代理类的类对象
Class clazz = Proxy.getProxyClass(classLoader, new Class[] { Interface.class, ... }); 
 
// 通过反射从生成的类对象获得构造函数对象
Constructor constructor = clazz.getConstructor(new Class[] { InvocationHandler.class }); 
 
// 通过构造函数对象创建动态代理类实例
Interface Proxy = (Interface)constructor.newInstance(new Object[] { handler });
```

实际使用过程更加简单，因为`Proxy`的静态方法`newProxyInstance`已经为我们封装了步骤 2 到步骤 4 的过程，只需两步即可完成代理对象的创建：

```java
// InvocationHandlerImpl 实现了 InvocationHandler 接口，并能实现方法调用从代理类到委托类的分派转发
InvocationHandler handler = new InvocationHandlerImpl(..); 
 
// 通过 Proxy 直接创建动态代理类实例
Interface proxy = (Interface)Proxy.newProxyInstance( classLoader, 
     new Class[] { Interface.class }, 
     handler );
```

> 动态代理：用户调用proxy类（代理类）的newInstance方法产生实现对应接口的proxy实例，该实例调用对应接口的方法的时候，会交由与代理类所对应的实现了InvocationHandler的处理器类的invoke方法处理，该处理器类内部封装了一个在构造时传递进来的真实对象，在invoke方法执行的时候会通过反射机制执行真实对象的对应的方法，并将方法的返回值返回出去。

**缺点**：我们可以看到，无论是静态代理还是动态代理，它都需要一个接口。那如果我们想要包装的方法，它就没有实现接口怎么办呢？可以使用CGLib动态代理，CGLib 是一个类库，它可以在运行期间动态的生成字节码，动态生成代理类。

### Read More

[Java JDK 动态代理（AOP）使用及实现原理分析](http://blog.csdn.net/jiankunking/article/details/52143504)

[Java 动态代理机制分析及扩展，第 1 部分](https://www.ibm.com/developerworks/cn/java/j-lo-proxy1/index.html)

[Spring 容器AOP的实现原理——动态代理](http://wiki.jikexueyuan.com/project/ssh-noob-learning/dynamic-proxy.html)

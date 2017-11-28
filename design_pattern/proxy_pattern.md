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

- 因为代理对象需要与目标对象实现一样的接口,所以会有很多代理类,类太多.同时,一旦接口增加方法,目标对象与代理对象都要维护.

如何解决静态代理中的缺点呢?答案是可以使用动态代理方式

#### 动态代理


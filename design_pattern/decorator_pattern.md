### 装饰模式

在《JAVA与模式》一书中开头是这样描述装饰（Decorator）模式的：

​	**装饰模式又称包装（Wrapper）模式。装饰模式以对客户端透明的方式扩展对象的功能，是继承关系的一个替代方案。**

装饰模式以对客户透明的方式动态地给一个对象添加一些额外的职责。换言之，客户端并不会觉得对象在装饰前和装饰后有什么不同。装饰模式可以在不使用创造更多子类的情况下，将对象的功能加以扩展。

#### 角色组成

![decorator_pattern_01.jpg](../img/design_pattern/decorator_pattern_01.jpg)

在装饰模式中的角色有：

  + **抽象构件(Component)角色**：给出一个抽象接口，以规范准备接收附加责任的对象。

  + **具体构件(ConcreteComponent)角色**：定义一个将要接收附加责任的类。

  + **装饰(Decorator)角色**：持有一个构件(Component)对象的实例，并定义一个与抽象构件接口一致的接口。

  + **具体装饰(ConcreteDecorator)角色**：负责给构件对象“贴上”附加的责任。


#### 举例说明

我们拿《深入浅出设计模式》中的例子来做说明，就是我们平常喝咖啡，可以加各种配料，你可以加糖、冰、牛奶等等好多配料，那么生成调配方案时，可以通过继承来实现，但是大家也都知道排列组合，继承的类系统庞大可想而知了。

> 装饰者模式强调的是**动态**的扩展, 而继承关系是**静态的**.

下面我们写代码来实现上述的例子，首先我们需要创建一个对象的抽象，也就是饮料接口：

```java
package com.flwcy.decorator;

/**
 * 抽象组件，装饰者和被装饰者都继承自它
 */
public interface Drink {
    public String make();
}
```

实现具体构建角色，也就是咖啡类:

```java
package com.flwcy.decorator;

/**
 * 具体组件角色(ConcreteComponent) :被装饰者
 */
public class Coffee implements Drink {

    public String make() {
        return "这是一杯咖啡";
    }
}

```

加了牛奶的饮料还应该是饮料, 故而, 牛奶这个装饰也实现了Drink接口, 加牛奶这个装饰是在饮料的基础上的, 所以我们还持有了一个Drink对象:

```java
package com.flwcy.decorator;

public class Milk implements Drink {

    private Drink drink;

    public Milk(Drink drink){
        this.drink = drink;
    }

    public String make() {
        return String.format("%s,加了%s",drink.make(),"牛奶");
    }
}

```

来实验下:

```java
package com.flwcy.app;

import com.flwcy.decorator.Coffee;
import com.flwcy.decorator.Drink;
import com.flwcy.decorator.Milk;

/**
 * Hello world!
 *
 */
public class App 
{
    public static void main( String[] args )
    {
        // 做一杯咖啡
        Drink coffee = new Coffee();
        System.out.println(coffee.make());
        // 加了牛奶的奶茶
        Drink milkCoffee = new Milk(new Coffee());
        System.out.println(milkCoffee.make());
    }
}

```

结果:

```
这是一杯咖啡
这是一杯咖啡,加了牛奶
```

咖啡可以加了糖、冰、牛奶等等好多配料，为了满足开闭原则，我们可以抽象出一个配料类：

```java
package com.flwcy.decorator;

/**
 * 为了满足开闭原则
 * 配料
 */
public abstract class Stuff implements Drink {
    private Drink drink;
    public Stuff(Drink drink){
        this.drink = drink;
    }

    public String make(){
        return String.format("%s,加了%s",drink.make(),stuffName());
    }

     abstract String stuffName();
}

```

其中Stuff类中值得注意的两个关系:

1. 我们的Stuff(配料)也是实现了Drink接口的, 这是为了说明加了配料(Stuff)的饮料还是饮料.
2. Stuff中还聚合了一个Drink(drink)实例, 是为了说明这个配料是加到饮料中的.

牛奶配料改为继承Stuff:

```java
package com.flwcy.decorator;

public class Milk extends Stuff {
    public Milk(Drink drink) {
        super(drink);
    }

    String stuffName() {
        return "牛奶";
    }
}

```

### Read More

[设计模式（九）装饰模式（Decorator）](https://www.kancloud.cn/digest/xing-designpattern/143730)

[可乐要加冰才好喝啊---装饰模式](http://blog.lmj.wiki/2016/11/22/design-pattern/decorator/index.html)
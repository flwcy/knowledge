### [译]什么是Java字符串池

> [What is Java String Pool?](https://www.journaldev.com/797/what-is-java-string-pool)

顾名思义，`Java字符串池`是[Java堆内存](https://www.journaldev.com/4098/java-heap-space-vs-stack-memory)中的存储字符串的一个池。我们知道`String`是`Java`中的特殊类，我们可以使用`new`运算符创建`String`对象，也可以用双引号提供值。

#### Java字符串池

下面的图清楚地说明了如何在[Java堆](https://www.journaldev.com/4098/java-heap-space-vs-stack-memory)空间中维护字符串池，以及当我们使用不同方法创建字符串时会发生什么。

![String Pool](../../img/JavaSe/basic/String-Pool-Java1.png)

字符串池是可能的，因为[String在Java中是不可变的](https://www.journaldev.com/802/string-immutable-final-java)，它是[字符串驻留(String interning)](https://en.wikipedia.org/wiki/String_interning)概念的实现。字符串池也是[享元设计模式](https://www.journaldev.com/1562/flyweight-design-pattern-java)的示例。

字符串池有助于为`Java`运行时节省大量空间，尽管创建字符串需要更多时间。

当我们使用双引号创建字符串时，它首先在字符串池中查找具有相同值的字符串，如果找到，它将返回引用，否则它将在池中创建一个新字符串，然后返回引用。

但是，使用`new`操作符，我们强制`String`类在堆空间中创建一个新的`String`对象。我们可以使用`intern()`方法将它放入池中，或者从具有相同值的字符串池中引用其他`String`对象。

下面是字符串池图的Java程序：

```java
package com.journaldev.util;

public class StringPool {

    /**
     * Java String Pool example
     * @param args
     */
    public static void main(String[] args) {
        String s1 = "Cat";
        String s2 = "Cat";
        String s3 = new String("Cat");
        
        System.out.println("s1 == s2 :"+(s1==s2));
        System.out.println("s1 == s3 :"+(s1==s3));
    }

}
```

上述程序的输出是：

```java
s1 == s2 :true
s1 == s3 :false
```

有时在[java面试](https://www.journaldev.com/2366/core-java-interview-questions-and-answers)中，你会被问到关于字符串池的问题。例如，在下面的语句中创建了多少个字符串：

```java
String str = new String("Cat");
```

在上面的语句中，将创建1或者2个字符串。如果池中已存在一个字符串文字"Cat"，那么只会在堆空间中创建一个`str`的`String`对象。如果池中不存在字符串文字"Cat"，那么它将首先在池中创建，然后在对空间中创建，因此总共将创建2个字符串对象。

**阅读**: [Java字符串面试问题](https://www.journaldev.com/1321/java-string-interview-questions-and-answers)

#### Read More

[Questions about Java's String pool](https://stackoverflow.com/questions/1881922/questions-about-javas-string-pool)

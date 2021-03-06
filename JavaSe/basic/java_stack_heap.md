#### Java中的栈内存和堆空间

> [Stack Memory and Heap Space in Java](https://www.baeldung.com/java-stack-heap)

#### 1.简介

为了以最佳方式运行应用程序，JVM将内存划分为栈和堆内存。**每当我们声明新的变量和对象，调用新方法，声明一个字符串或者类似这样的操作时，JVM就会从栈内存或者堆空间中为这些操作指定内存**。

在这篇文章中，我们将会讨论这些内存模型。我们将列出它们之间的一些关键区别，它们如何存储在RAM中，它们提供的特性以及在哪使用它们。

#### 2. Java中的栈内存

**在Java中的栈内存被用于静态内存分配和线程的执行**。它包含特定于方法的基本数据类型的值，以及对该方法中引用的对堆中对象的引用。

访问这个内存是基于后进先出（LIFO）规则。无论何时调用新方法，都会在栈顶部创建一个新块，它包含特定于方法的值，如基本数据类型的变量和对象的引用。

当方法执行完成时，对应的栈帧被刷新，流程返回到调用方法，并且空间可用于下一个方法。

##### 2.1 栈内存的主要特点

除了我们目前为止讨论的，以下是堆栈内存的一些其他特性：

+ 它随着新方法的调用和返回而增长和缩小。
+ 栈中的变量只在创建它们的方法运行时存在
+ 当方法完成执行时，它自动分配和释放
+ 如果此内存已满，Java抛出java.lang.StackOverFlowError
+ 与堆内存相比，对该内存的访问速度更快
+ 这个内存是线程安全的，因为每个线程都在自己的栈中运行

#### 3 Java中的堆空间

**Java中的堆空间用于在运行时为Java对象和JRE类分配动态内存**。新对象总是在堆空间中创建，对该对象的引用存储在栈内存中。

这些对象具有全局访问权，可以从应用程序中的任何位置访问。

这个内存模型进一步细分为更小部分，称为代，它们是：

1. **年轻代** —— 这是分配和老化所有新对象的地方。当被填满时会发生较小的垃圾回收。

2. **老年代** —— 这是存储长期存活对象的地方。当对象存储在年轻代时，设置对象年龄的阀值，当达到该阀值时，对象奖移动到老年代。

3. **永久代** —— 它由运行时类和应用程序方法的JVM元数据组成。

它们的不同之处也在这篇文章中也有讨论—— [JVM,JRE和JDK之间的差异](https://www.baeldung.com/jvm-vs-jre-vs-jdk)。

我们总是可以根据需要来操作堆内存的大小。有关更多信息，请访问这篇链接的[Baeldung的文章](https://www.baeldung.com/jvm-parameters)。

##### 3.1 Java堆内存的主要特点

除来目前为止我们讨论的，以下也是关于堆空间的其他一些特点：

+ 它是通过复杂的内存管理技术访问的，这些技术包括年轻代，老年代和永久代。
+ 如果堆空间满了，Java就会抛出`java.lang.OutOfMemoryError`
+ 对该内存的访问相对于栈内存要慢。
+ 与栈相比，此内存不会自动释放。它需要垃圾收集器来释放未使用的对象，以保持内存使用的效率。
+ 不像栈，堆不是线程安全的，需要通过正确同步代码块来加以保护。

#### 4. 例子

基于我们目前学到的知识。让我们分析一段简单的Java代码，让我们评估一下如何管理内存：

```java
class Person {
    int pid;
    String name;
     
    // constructor, setters/getters
}
 
public class Driver {
    public static void main(String[] args) {
        int id = 23;
        String pName = "Jon";
        Person p = null;
        p = new Person(id, pName);
    }
}
```

让我们一步步分析：

1. 在进入main()方法时，将在栈内存中创建一个空间来存储方法的基本数据类型和引用
   + 基本数据类型整数id的值将直接存储在栈内存中
   + Person类型的引用变量p也将在栈内存中创建，并且指向堆中实际的对象。
2. main()对带参构造函数Person(int, String)的调用，将在前一个栈的顶部分配更多内存。这将存储：
   + 在栈内存中调用对象的this对象的引用
   + 栈内存中的基本数据类型id的值
   + String参数的引用变量personName将会指向在堆内存中的字符串池中的实际字符串。
3. 这个默认构造函数进一步调用setPersonName()方法，对于该方法的，将在前一个栈内存的顶部进行进一步的分配。这将再次以上述方式存储变量。
4. 但是，对于新创建的Person类型的对象p，所有实例变量都将存储在堆内存中。

这个分配情况在图中说明：

![Stack-Memory-vs-Heap-Space-in-Java.jpg](../../img/JavaSe/basic/Stack-Memory-vs-Heap-Space-in-Java.jpg)

#### 5. 总结

在我们结束本文之前，让我们快速总结一下堆空间和栈内存之间的区别：

|   参数    |                        栈内存                        |                            堆空间                            |
| :-------: | :--------------------------------------------------: | :----------------------------------------------------------: |
| 应用程序  |          栈在局部使用，线程执行期间一次一个          |                整个应用程序在运行时使用堆空间                |
|   大小    |       栈的大小限制取决于操作系统，通常小于堆，       |                       堆上没有大小限制                       |
|   存储    | 仅存储基本数据类型的变量和对堆空间中创建的对象的引用 |                  所有新创建的对象都存储在这                  |
|   规则    |     它使用后进先出（LIFO）内存分配系统进行访问。     | 该内存通过复杂的内存管理技术访问，这些技术包括年轻代、老代或终身代以及永久代。 |
| 生命周期  |             栈内存只在当前方法运行时存在             |                只要应用程序运行，堆空间就存在                |
|   效率    |              与堆相比，分配速度要快的多              |                    与栈相比，分配速度较慢                    |
| 分配/释放 |    当分别调用和返回方法时，将自动分配和释放该内存    | 堆空间在创建新对象时分配，并且当不再引用这些对象时垃圾收集器将释放它们。 |

#### 6. 结论

栈和堆是Java分配内存的两种方式。在本文中，我们了解它们如何工作以及何时使用它们来开发更好的Java程序。

要了解更多关于Java内存管理的知识，请看这里的[这篇文章](https://www.baeldung.com/java-memory-management-interview-questions)。我们还讨论了JVM垃圾收集器，[本文](https://www.baeldung.com/jvm-garbage-collectors)将简要讨论它。






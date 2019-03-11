#### Java堆空间 VS 栈 ——Java中的内存分配

> [Java Heap Space vs Stack – Memory Allocation in Java](https://www.journaldev.com/4098/java-heap-space-vs-stack-memory)

以前我写过几篇关于[Java垃圾回收](https://www.journaldev.com/2856/java-jvm-memory-model-memory-management-in-java)和[Java是通过值传递的](https://www.journaldev.com/3884/java-is-pass-by-value-and-not-pass-by-reference)，在那之后，我收到了许多希望我讲解**Java堆空间，栈内存，Java中的内存分配以及他们的区别是什么**的邮件。

你也许在Java，Java EE书籍和教程中看到大量对堆和栈内存的引用，但很难完整的解释什么是程序方面的堆和栈内存。

#### 1 Java堆空间

Java堆空间用于Java运行时为对象和JRE类分配内存。无论我们何时创建任何对象，它都是在堆空间中创建的。

垃圾回收在堆内存上运行，用于释放没有任何引用的对象所使用的内存。在堆空间中创建的任何对象都具有全局访问权，可以从应用程序的任何位置引用。

##### 1.1 Java栈内存

Java栈内存用于执行线程。它们包含方法的特定的值，这些值是短期的，并且引用从该方法引用的堆中的其他对象。

栈内存始终以LIFO（后进先出）的顺序引用。每当调用一个方法时，将在栈内存中开辟一个新的块，以便该方法保存本地基元值以及方法中其他对象的引用。

方法结束后，块将变成未被使用状态，并可用于下个方法。与堆内存相比，栈内存的大小要小的多。

##### 1.2 Java程序中的堆和栈内存

让我们通过一个简单的程序来理解堆和栈内存的使用情况。

```java
package com.journaldev.test;

public class Memory {

	public static void main(String[] args) { // Line 1
		int i=1; // Line 2
		Object obj = new Object(); // Line 3
		Memory mem = new Memory(); // Line 4
		mem.foo(obj); // Line 5
	} // Line 9

	private void foo(Object param) { // Line 6
		String str = param.toString(); //// Line 7
		System.out.println(str);
	} // Line 8

}
```

下图显示了与上述程序相关的栈和堆内存情况，以及它们如何用于存储基元，对象和引用变量。
![Java-Heap-Stack-Memory.png](../../img/JavaSe/basic/Java-Heap-Stack-Memory.png)

让我们来完成程序的执行步骤。




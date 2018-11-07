### 数据结构之数组

数组是相同类型的元素的集合。数组中的元素可以通过索引来访问，索引从0开始。

+ 数组最大的优点：快速查询
+ 数组最好应用于“索引有语义”的情况

索引可以有语义也可以没有语义。例如我们只是想存放77，66，88这三个数字，那么它们保存在索引为0，1，2的这几个地方或者其他任何地方都可以。但是如果它们变成了学号为1，2，3这几个同学对应的成绩，那么它们的索引就有了语义，索引0对应了学号为1的同学的成绩，索引1对应了学号2的同学，索引2对应了学号3的同学。但是，并非所有有语义的索引都适合数组：例如身份证号，位数较多，开辟空间过大。

+ 索引没有语义，如何表示没有元素
+ 如何添加元素？如何删除元素？

#### 封装自己的数组类

java中的数组并没有提供CRUD的操作，因此我们需要基于java的数组，二次封装属于我们自己的数组类。大体结构如下：

![data-structure-array3](../img/data_structure/data-structure-array3.png)

代码如下：

```java
package com.flwcy.array;

/**
 * 封装自定义数组
 * Created by flwcy on 2018/11/6.
 */
public class Array {
    private int[] data;

    /**
     * 实际大小
     */
    private int size;
}

```

我们需要一个成员变量来保存我们的数据，这里是`data`，然后需要一个`int类型`来存放我们的有效元素的个数，在这里我们没有必要再多定义一个表示数组容量的变量，因为这里的容量就是`data.length`；

我们需要通过构造函数初始化数组，用户知道数组的容量时，构造函数如下：

```java
    /**
     * 根据数据的容量构造数组
     * @param capacity 数组的容量
     */
    public Array(int capacity) {
        this.data = new int[capacity];
        this.size = 0;
    }
```

我们也可以为用户提供一个默认构造函数：

```java
    /**
     * 无参构造函数，默认数组的容量capacity=10
     */
    public Array() {
        this(10);
    }
```

一些通用的方法

```java
    /**
     * 获取数组实际大小
     * @return 数组实际大小
     */
    public int getSize() {
        return size;
    }

    /**
     * 获取数组容量
     * @return 容量
     */
    public int getCapacity(){
        return data.length;
    }

    /**
     * 数组是否为空
     * @return
     */
    public boolean isEmpty(){
        return size == 0;
    }
```

#### 添加元素

##### 向数组末位添加元素

涉及到添加元素时，我们需要对临界值进行判断。初始的时候，整个数组为空，此时`size=0`，现在向数组末尾添加一个新的元素，我们只要在`data[0]`这个位置放上66，同时维护一下`size++`(这个`size`表示数组中元素个数)，而`size`变量指向的是数组中首个没有元素的位置。我们向数组末尾添加元素，其实就等同于向`size`这个位置添加元素。添加下个元素时同理。

图解：

![data-structure-array](../img/data_structure/data-structure-array4.jpg)

代码如下：

```java
    /**
     * 在数组末尾追加数据
     */
    public void addLast(int ele){
        if(size == data.length)
            throw new IllegalArgumentException("The array is full");
        data[size] = ele;
        size++;
    }
```

##### 向指定位置添加元素

我们需要把当前索引（比如1）为指定位置的的元素和它后面的元素，都向后挪一个位置。

- 先把最后一个元素（索引为3）挪到size（大小为4）这个位置
- 前一个元素（索引为2）诺到size-1（也就是3），如此反复……
- 最后挪完以后，索引为1就腾出来了，然后我们只要直接插入数据77就好了。
- 全部完成操作之后，需要维护一下size++，保证了它指向数组中首个没有元素的位置。

图解：

![data-structure-array5](../img/data_structure/data-structure-array5.jpg)

代码如下：

```java
/**
     * 在指定位置添加元素
     * @param index
     * @param ele
     */
    public void add(int index,int ele) {
        if(index < 0 || index > size)
            throw new IllegalArgumentException("Add failed. Require index >= 0 and index <= size.");
        if(size == data.length)
            throw new IllegalArgumentException("Add failed. Array is full");
        // 从最后一个元素开始向后挪动一个位置
        for(int i = size; i >= index; i--)
            data[i] = data[i - 1];
        data[index] = ele;
        size++;
    }
```

此时我们之前的`addLast`方法可以复用这个方法， 也可以造一个`addFirst`方法：

```java
    /**
     * 在数组末尾追加数据
     */
    public void addLast(int ele){
        add(size,ele);
    }

    /**
     * 在数组开始添加元素
     */
    public void addFirst(int ele) {
        add(0,ele);
    }
```



https://loubobooo.com/2018/07/21/%E5%88%9D%E5%AD%A6%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84-%E6%95%B0%E7%BB%84/

https://www.jianshu.com/p/7b93b3570875
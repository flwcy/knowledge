#### 集合

在我们编写代码的过程中，通常需要将一类数据集中保存，之前学习的数组是一个很好的选择，但是数组需要我们提前知道要保存对象的数量，也就是说数组初始化后，数组的长度就是不可变的，为了保存动态变化的数据，Java提供类集合类库来解决这个问题。

集合类也被称为容器类，位于`java.util`包下，并且Java集合类库将接口（interface）与实现（implementation）分离。

##### Collection接口

在Java类库中，集合类的基本接口是`Collection`接口。

```java
public interface Collection<E> {
    /**
    * 添加元素
    * @return 集合发生变化就返回true
    * 			集合未发生变化就返回false
    */
    boolean add(E element);
    
    Iterator<E> iterator();
}
```

##### 迭代器Iterator

`Iterator`接口包含四个方法：

```java
public interface Iterator<E> {
    E next();
    boolean hasNext();
    void remove();
    default void forEachRemaining(Consumer<? super E> action);
}
```

反复调用`add`方法，可以逐个访问集合中的每个元素。但是，如果到达了集合的末尾，`next`方法将抛出一个`NoSuchElementExcetption`。因此，需要再调用`next`之前调用`hasNext`方法。

```java
Collection<String> c = ...;
Iterator<String> iterator = c.iterator();
while(iterator.hasNext) {
    String element = iterator.next();
    // do something with element
}
```

当然我们可以使用更加简练的`for each`循环来实现集合的遍历。

```java
for(String element : collection) {
    // do something with element
}
```

`for each`循环可以与任何实现了`Iterable`接口的对象一起工作。

```java
public interface Iterable<E> {
    Iterator<E> iterator();
}
```

`Collection`接口扩展了`Iterable`接口，因此，对于标准类库中的任何集合都可以使用`for each`循环。

在Java 8中，也可以通过为`forEachRemaining`方法提供一个`lambda`表达式来实现。

```java
iterator.forEachRemaining(element -> do something with element);
```

迭代器位于**两个元素之间**，当调用`next`时，迭代器就越过下一个元素，并返回刚刚越过的那个元素的引用。`Iterator`接口的`remove`方法将会删除上次调用`next`时返回的元素。

```java
Iterator<String> it = c.iterator();
it.next(); // skip over the first element
it.remove(); // now remove it
```

![iterator next method](../../img/JavaSe/basic/iterator_next_method.png)

更重要的事，对`next`方法和`remove`方法的调用具有互相依赖性。如果调用remove方法之前没有调用`next`方法将是不合法的。如果这样做，将会抛出一个`IllegalStateException`异常。如果想要删除两个相邻的元素，不能直接这样调用：

```java
iterator.remove();
iterator.remove(); // Error!
```

必须先调用`next`越过将要删除的元素。

```java
iterator.remove();
iterator.next();
iterator.remove(); // OK
```

##### 泛型实用方法

`Collection`接口声明了很多实用的方法，所有的实现类都必须提供这些方法。

![collection method](../../img/JavaSe/basic/collection_method.png)

如果实现`Collection`接口的每一个类都要提供如此多的例行方法将是一件很烦人的事情。为了能够让实现者更容易地实现这个接口，Java类库提供了一个类`AbstractCollection`，它将基础方法`size`和`iterator`抽象化了，但是在此提供了例行方法，例如：

```java
public abstract AbstractCollection<E> {
    public abstract Iterator<E> iterator();
    
    public boolean contains(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    return true;
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return true;
        }
        return false;
    }
}
```

此时，一个具体的集合类可以扩展`AbstractCollection`类了。现在要由具体的集合类提供`iterator`方法，而`contains`方法已由`AbstractCollection`超类提供了。

##### 集合框架中的接口

Java集合框架为不同类型的集合定义了大量的接口，如下图所示：

![collection interface](../../img/JavaSe/basic/java_collection_interface.png)

集合有两个基本的接口：`Collection`和`Map`。Collection中添加元素方法如下：

```java
boolean add(E element);
```

`Map`包含键/值对，因此使用`put`方法添加元素：

```java
V put(K key,V value);
```

`Collection`提供迭代器来访问元素，而`Map`提供`get`方法来通过键获取对应的值：

```java
V get(K key);
```

`List`是一个有序集合（ordered collection），可以采用两种方式访问元素：使用迭代器访问，或者使用一个整数索引来访问。后一种方法称为随机访问（random access），因为这样可以按任意顺序访问元素。与之不同，使用迭代器访问时，必须顺序地访问元素。

`Set`是一个不允许出现重复元素，并且无序的集合。

##### 具体的集合

![specific collection](../../img/JavaSe/basic/java_specific_collection.png)

###### LinkedList

`Array`和`ArrayList`都有一个重大的缺陷，这就是从数组的中间位置删除一个元素要付出很大的代价，其原因是数组中处于被删除元素之后的所有元素都要向数组的前端移动。在数组中间位置插入一个元素也是如此。

![delete a element form array](../../img/JavaSe/basic/delete_a_element_from_array.png)

链表（linked list）解决了这个问题，数组在连续的存储位置上存放对象引用，但是链表将每个对象存放在独立的结点中，每个结点还存放着序列中下一个结点的引用。Java中所有的链表都是双向的（double linked）——即每个结点还存放着指向前驱结点的引用。

![double linked](../../img/JavaSe/basic/java_linked_list.png)

从链表中间删除元素很容易，只需要更新被删除元素附近的链接。

![delete a element from linked list](../../img/JavaSe/basic/delete_a_element_from_linked_list.png)

下面的代码演示了添加3个元素，然后再将第二个元素删除：

```java
List<String> staff = new LinkedList<>(); //LinkedList implements List
staff.add("Amy");
staff.add("Bob");
staff.add("Carl");
Iterator iter = staff.iterator();
String first = iter.next(); // visit first element
String second = iter.next(); // visit second element
iter.remove(); // remove last visited element
```

由于链表是一个有序的集合（ordered collection），`LinkedList.add`将对象添加到链表的末尾，但是，通常需要将元素添加到链表的中间。由于迭代器是描述集合中的位置的，所以这种依赖于位置的`add`方法将由迭代器负责。只有对自然有序的集合使用迭代器添加元素才有实际的意义。而`Set`集中的元素是完全无序的，因此在`Iterator`接口中就没有`add`方法。集合类库提供了子接口`ListIterator`，其中包含`add`方法：

```java
interface ListIterator<E> extends Iterator<E> {
    void add(E element); // 假定添加操作总会改变链表
    // 下面两个方法用于反向遍历链表
    E previous(); // 返回越过的对象
    boolean hasPrevious();
}
```

`LinkedList`类的`listIterator`返回一个实现了`ListIterator`接口的迭代器：

```java
ListIterator<String> listIterator = staff.listIterator();
```

下面的代码将越过链表中的第一个元素，并在第二个元素之前添加“Juliet“：

```
List<String> staff = new LinkedList<>(); //LinkedList implements List
staff.add("Amy");
staff.add("Bob");
staff.add("Carl");
Iterator iter = staff.listIterator();
iter.next(); // skip past firest element
iter.add("Juliet");
```

![add element by listIterator](../../img/JavaSe/basic/add_element_by_list_iterator.png)

`set`方法用一个新元素取代调用`next`或`previous`方法返回的上一个元素。

```java
// 用一个新值取代链表的第一个元素
ListIterator<String> iter = staff.listIterator();
String oldValue = iter.next(); // returns first element
iter.set(newValue); // sets first element to newValue
```

如果迭代器发现它的集合被另一个迭代器修改了，或是被该集合自身的方法修改了，就会抛出一个`ConcurrentModificationException`异常。  

```java
        LinkedList<String> list  = new LinkedList<>();
        list.add("Hello");
        list.add("1");
        list.add("2");
        Iterator<String> iter = list.iterator();
        while (iter.hasNext()) {
            String str = iter.next(); // ConcurrentModificationException
            if("Hello".equals(str)) {
                list.remove(str);
            }
        }
```

链表不支持快速地随机访问。如果要查看链表中第n个元素，就必须从头开始，越过n-1个元素。`get`方法做了微小的优化：如果索引大于`size()/2`就从链表尾端开始搜索元素。  

`LinkedList`与`ArrayList`的对比：

1. 顺序插入速度`ArrayList`会比较快，因为`ArrayList`是基于数组实现的，数组是事先`new`好的，只要往指定位置塞一个数据就好了；`LinkedList`则不同，每次顺序插入的时候`LinkedList`将`new`一个对象出来，如果处理对象比较大。那么`new`的时间势必会长一点，再加上一些引用赋值的操作，所以顺序插入`LinkedList`必然慢于`ArrayList`
2. 基于上一点，因为`LinkedList`里面不仅维护了待插入的元素，还维护了`Entry`的前置`Entry`和后置`Entry`，如果一个`LinkedList`中的`Entry`非常多，那么`LinkedList`将比`ArrayList`更耗费一些内存
3. 使用各自遍历效率最高的方式，`ArrayList`的遍历效率会比`LinkedList`的遍历效率高一些

- `LinkedList`做插入、删除的时候，慢在寻址，快在只需要改变前后`Entry`的引用地址

- `ArrayList`做插入、删除的时候，慢在数组的批量`copy`，快在寻址

  所以，如果待插入、删除的元素是在数据结构的前半段尤其是非常靠前的位置的时候，`LinkedList`的效率将大大的快过于`ArrayList`，因为`ArrayList`将批量`copy`大量的元素；越往后，对于`LinkedList`来说，因为它是双向链表，所以在第2个元素后面插入一个数据和在倒数第2个元素后面插入一个元素在效率上基本上没有差别，但是ArrayList由于要批量`copy`的元素越来越少，操作速度必然追上甚至超过`LinkedList`。

  从这个分析看出，如果你十分确定你插入、删除的元素是在前半段，那么就使用`LinkedList`；如果你十分确定你插入、删除的元素在比较靠后的位置，那么可以考虑使用`ArrayList`。如果你不能确定你要做的插入、删除是在哪？那么还是建议你使用`LinkedList`吧，因为一来`LinkedList`整体插入、删除的执行效率比较稳定，没有`ArrayList`这种越往后越快的情况；二来插入元素的时候，弄的不好`ArrayList`就要进行一次扩容，记住，`ArrayList`底层数组扩容是一个既消耗时间又消耗空间的操作。

  `ArrayList`使用普通的`for`循环遍历，`LinkedList`使用`foreach`循环比较快。如果使用普通`for`循环遍历`LinkedList`，在大数据量的情况下，其遍历速度将慢得令人发指。

###### ArrayList

`ArrayList`的优点如下：

1. `ArrayList`底层以数组实现，是一种随机访问模式，再加上它实现了`RandomAccess`接口，因此查找也就是`get`的时候非常快。
2. `ArrayList`在顺序添加一个元素的时候非常方便，只是往数组里面添加了一个元素而已。

不过`ArrayList`的缺点也十分明显：

1. 删除元素的时候，涉及到一次元素复制，如果要复制的元素很多，那么就会比较耗费性能。
2. 插入元素的时候，涉及到一次元素复制，如果要复制的元素很多，那么就会比较耗费性能。

因此`ArrayList`比较适合顺序添加、随机访问的场景。`ArrayList`底层封装了一个动态再分配的`Object`数组。通常面试会问`ArrayList`与`Vector`的区别：

1. `ArrayList`不是线程安全的，`Vector`是线程安全的。
2. `ArrayList`进行扩容时增加50%，`Vector`提供了扩容时的增量设置，但通常将容量扩大1倍。

###### 散列集

有一种众所周知的数据结构，可以快速地查找所需要的对象，这就是散列表（hash table）。散列表为每个对象计算一个整数，称为`散列码`(hash code)。散列码是由对象的实例域产生的一个整数。更准确地说，具有不同数据域的对象将产生不同的散列码。`key1 != key2`的情况下，通过散列函数处理，`hash(key1) == hash(key2)`，这种现象被称为散列冲突（hash collision）。

在Java中，散列表用链表数组实现。每个列表被称为桶（bueket）。要想查找表中对象的位置，就要先计算它的散列码，然后与桶的总数取余，所得到的结果就是保存这个元素的桶的索引。

> 在Java SE 8中，桶满时会从链表变为平衡二叉树。如果选择的散列函数不当，会产生很多冲突。

如果大致知道最终会有多少个元素要插入到散列表中，就可以设置桶数。通常，将桶数设置为预计元素个数的75%~150%。最好将桶数设置为一个素数，以防键的集聚。标准类库使用的桶数是2的幂，默认值为16（为表大小提供的任何值都将自动地转换为2的下一个幂）。

当然，并不是总能够知道需要存储多少个元素的，也有可能最初的估计过低。如果散列表太满，就需要再散列（rehashed）。如果要对散列表再散列，就需要创建一个桶数更多的表，并将所有元素插入到这个新表中，然后丢弃原来的表。装填因子（load factor）决定何时对散列表再散列。例如，如果装填因子为0.75（默认值），而表中超过75%的位置已经填入元素，这个表就会用双倍的桶数自动地进行再散列。对于大多数应用程序来说，装填因子为0.75是比较合理的。

##### HashMap

`HashMap`是一种非常常见、方便和有用的集合，是一种键值对（K-V）形式的存储结构。

使用一个`Map`统计一个单词在文件中出现的次数，看到一个单词时，我们将计数器加1:

```java
counts.put(word,counts.get(word) + 1);
```

第一次看到`word`时，`get`方法会返回`null`，因此抛出`NullPointerException`，我们可以使用`getOrDefault`方法来避免这个问题：

```java
counts.put(word,counts.getOrDefault(word,0) + 1);
```

另一种方法是首先调用`putIfAbsent`方法。

```java
counts.putIfAbsent(word,0);
counts.put(word,counts.get(word) + 1); 
```

另外可以使用`merge`方法简化这个操作：

```java
counts.merge(word,1,Integer::sum);
```

`Map`有三种返回视图的方法：

```java
Set<K> keySet();
Collection<V> values();
Set<Map.Entry<K,V>> entrySet();
```

如果想同时获取键值对：

```java
for(Map.Entry<String,Employee> entry : staff.entrySet()) {
    String key = entry.getKey();
    Employee employee = entry.getValue();
    // do something with key,value
}
```

也可以采用`lambda`表达式：

```java
counts.forEach((k,v) -> {
    // do something with key,value
});
```

看一下`put`方法的源码：

```java
if ((p = tab[i = (n - 1) & hash]) == null)
    tab[i] = newNode(hash, key, value, null);
```

`(n - 1) & hash`实际上是计算出`key`在`tab`中索引位置，这里使用了`&`，移位加快一点代码运行效率。另外这个取模操作的正确性依赖于`length`必须是2的`N`次幂，因此注意`HashMap`构造函数中，如果你指定`HashMap`初始数组的大小`initialCapacity`，如果`initialCapacity`不是2的`N`次幂，`HashMap`会算出大于`initialCapactiy`的最小2的`N`次幂的值，作为`Entry`数组的初始化大小。

`HashMap`中对`Key`的`HashCode`要做一次`rehash`，防止一些糟糕的`Hash`算法生成糟糕的`HashCode`，那么为什么要防止糟糕的`HashCode`？糟糕的`HashCode`意味着的是`Hash`冲突，即多个不同的`Key`可能得到同一个`HashCode`，糟糕的`Hash`算法意味着的就是`Hash`冲突的概率增大，这意味着`HashMap`的性能将下降，表现在两个方面：

1. 有10个`Key`，可能6个`Key`的`HashCode`都相同，另外四个`Key`所在的`Entry`均匀分布在`table`的位置上，而某一个位置上却连接了6个`Entry`。这就失去了`HashMap`的意义，`HashMap`这种数据结构高性能的前提是，`Entry`均匀地分布在`table`位置上，但现在却是`1 1 1 1 6`的分布。所以，我们要求`HashCode`有很强的随机性，这样就尽可能地可以保证`Entry`分布的随机性，提升了`HashMap`的效率。
2. `HashMap`在一个某个`table`位置上遍历链表的时候的代码：`if (e.hash == hash && ((k = e.key) == key || key.equals(k)))`，由于采用了`&&`运算符，因此先比较`HashCode`，`HashCode`都不相同就直接`pass`了，不会再进行`equals`比较了。`HashCode`因为是`int`值，比较速度非常快，而`equals`方法往往会对比一系列的内容，速度会慢一些。`Hash`冲突的概率大，意味着`equals`比较的次数势必增多，必然降低了`HashMap`的效率了。

> [Why do I need to override the equals and hashCode methods in Java?](https://stackoverflow.com/questions/2265503/why-do-i-need-to-override-the-equals-and-hashcode-methods-in-java)
>
> Joshua Bloch says on Effective Java
>
> ```
> You must override hashCode() in every class that overrides equals(). Failure to do so will result in a violation of the general contract for Object.hashCode(), which will prevent your class from functioning properly in conjunction with all hash-based collections, including HashMap, HashSet, and Hashtable.
> ```

##### TODO

```
阅读ArrayList／LinkedList/HashMap的源码，写代码实现自己的ArrayList／LinkedList／HashMap，并编写测试代码，测试通过后对比ArrayList/LinkedList/HashMap的源码实现方式与自己代码的区别

练习题：实现斗地主发牌，发三个人牌，并留出三张底牌，并自动理牌。
```

##### 参考及引用

[ArrayListc初始化](https://zhuanlan.zhihu.com/p/27873515)

[ArrayList底层数组扩容原理](https://zhuanlan.zhihu.com/p/27878015)

[三顾ArrayList](https://zhuanlan.zhihu.com/p/27938717)

[LinkedList初探](https://zhuanlan.zhihu.com/p/28101975)

[LinkedList元素的删除原理](https://zhuanlan.zhihu.com/p/28373321)

[HashMap底层实现原理（上）](https://zhuanlan.zhihu.com/p/28501879)

[HashMap底层实现原理（下）](https://zhuanlan.zhihu.com/p/28587782)

[图解集合1：ArrayList](https://www.cnblogs.com/xrq730/p/4989451.html)

[图解集合2：LinkedList](https://www.cnblogs.com/xrq730/p/5005347.html)

[图解集合4：HashMap](https://www.cnblogs.com/xrq730/p/5030920.html)
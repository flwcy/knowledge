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


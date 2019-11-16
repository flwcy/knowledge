#### 泛型程序设计

通过提供实际类型参数替换形式类型参数来实例化泛型类型以形成**参数化类型**。诸如`LinkedList<E>`之类的类是泛型类型，它有一个类型参数`E`，实例化（例如`LinkedList<Integer>`或`LinkedList<String>`）称为参数化类型，`String`和`Integer`分别是各自的实际类型参数。

泛型（generic）的本质是类型参数（type parameters），类型参数使程序更好的可读性和安全性。

来看下面这样一个例子：

```java
List list = new ArrayList();
list.add("Hello World");
list.add(10086);
for(int i=0; i<list.size(); i++) {
    String item = (String) list.get(i);
    system.out.println(item); // java.lang.ClassCastException
}
```

使用泛型就能避免上述问题，当调用`get`时，也无需进行强制类型转换，并且编译器可以进行检查，避免插入错误类型的对象。

> “Precisely, Generics in Java is nothing but a syntactic sugar to your code for Type Safety and all such type information is erased by Type Erasure feature by compiler.”
>
> 准确地说，Java中的泛型只不过是为了类型安全而在代码中添加的语法糖，所有这些类型信息都会被编译器的类型擦除（Type Erasure）特性清除。

##### 定义泛型类或泛型接口

一个泛型类（Generic class）就是具有一个或多个类型参数的类。

```java
class DemoClass<T> {
    // T stands for "Type"
    private T t;
    
    public void set(T t) {
        this.t = t;
    }
    
    public T get() {
        return t;
    }
}
```

泛型类定义时只需要在类名后面加上类型参数即可，当然你也可以添加多个参数，类似于<T,K>或<T,K,V>等。

+ E：元素element
+ T：类型type
+ N：数字number
+ K：键key
+ V：值value

现在我们能够保证不会被错误的类型误用。

```java
DemoClass<String> instance = new DemoClass<String>();
instance.set("fpx"); // Correct usage
instance.set(10); // This will raise compile time error
```

> 在JavaSE 7及之后的版本中，构造函数中可以省略泛型类型：ArrayList<String> list = new ArrayList<>();

泛型接口的定义基本上和泛型类的定义相同。

```java
// Generic interface definition
interface DemoInterface<T1,T2> {
    T2 doSomeOperation(T1 t);
    T1 doReverseOperation(T2 t);
}

// a class implementing generic interface
class DemoClass implements DemoInterface<String, Integer> {
    public Integer doSomeOperation(String t) {
        // some code
    }
    
    public String doReverseOperation(Integer t) {
        // some code
    }
}
```

此处有两点需要注意：

- 泛型接口未传入泛型实参时，与泛型类的定义相同，在声明类的时候，需将泛型的声明也一起加到类中。例子如下：

```java
/* 即：class DataHolder implements Generator<T>{
 * 如果不声明泛型，如：class DataHolder implements Generator<T>，编译器会报错："Unknown class"
 */
class FruitGenerator<T> implements Generator<T>{
    @Override
    public T next() {
        return null;
    }
}
```

- 如果泛型接口传入类型参数时，实现该泛型接口的实现类，则所有使用泛型的地方都要替换成传入的实参类型。例子如下：

```java
class DataHolder implements Generator<String>{
    @Override
    public String next() {
    	return null;
    }
}
```

##### 泛型方法或构造器

泛型方法与泛型类非常相似。它们唯一的不同之处在于类型信息的作用域只在方法（或构造函数）内部。泛型方法是引入自己的类型参数的方法。

```java
public static <T> int countAllOccurrences(T[] list, T item) {
   int count = 0;
   if (item == null) {
      for ( T listItem : list )
         if (listItem == null)
            count++;
   }
   else {
      for ( T listItem : list )
         if (item.equals(listItem))
            count++;
   }
   return count;
}  
```

类型变量放在修饰符（这里是`public static`）的后面，返回类型的前面。泛型方法可以定义在普通类中，也可以定义在泛型类中。

接下来看一个泛型构造器的例子：

```java
class Dimension<T>
{
   private T length;
   private T width;
   private T height;
 
   //Generic constructor
   public Dimension(T length, T width, T height)
   {
      super();
      this.length = length;
      this.width = width;
      this.height = height;
   }
}
```

##### 泛型数组

在Java中，在运行时将任何不兼容的类型放入熟组中都会抛出`ArrayStoreException`。这意味着数组在运行时保留类型信息，而泛型使用类型擦除或在运行时删除任何类型信息。由于上述冲突，**不允许在java中实例化泛型数组**。

```java
public class GenericArray<T> {
    // this one is fine
    public T[] notYetInstantiatedArray;
  
    // causes compiler error; Cannot create a generic array of T
    public T[] array = new T[5];
}
```

与上述泛型类型类和方法相同，我们可以在java中拥有泛型数组。众所周知，数组是相似类型元素的集合，并且放入任何不兼容的类型都会在运行时抛出`ArrayStoreException`。这与`Collection`类的情况不同。

```java
Object[] array = new String[10];
array[0] = "lokesh";
array[1] = 10;      //This will throw ArrayStoreException
```

数组不支持泛型的另一原因是数组是协变的，这意味着超类型引用的数组是子类型引用的数组的超类型。也就是说，`Object[]`是`String[]`的超类型，并且可以通过`Object[]`类型的引用变量访问字符串数组。

```java
Object[] objArr = new String[10];  // fine
objArr[0] = new String(); 
```

##### 类型变量的限定

看这样一段代码：

```java
class ArrayAlg {
    public static <T> T min(T[] a) {
        if(a == null || a.length == 0) {
            return null;
        }
        T smallest = a[0];
        for(int i = 1; i<a.length; i++) {
        	if(smallest.compareTo(a[i]) > 0) {
                smallest = a[i];
        	}
        }
        
        return smallest;
    }
}
```

变量`smallest`类型为`T`，这意味着它可以是任何一个类的对象，怎么才能确保`T`所属类有`compareTo`方法呢？

可以通过对类型变量设置限定（bound）：

```java
public static <T extends Comparable> T min(T[] a)...
```

`<T extends BoundingType>`表示`T`应该是绑定类型的子类型（subtype），`T`和绑定类型可以是类，也可以是接口。一个类型变量或通配符可以有多个限定：

```java
T extends Comparable & Serializable
```

限定类型用“ &”分隔，而逗号用来分隔类型变量。在 Java 的继承中， 可以根据需要拥有多个接口超类型， 但限定中至多有一个类。 如果用一个类作为限定， 它必须是限定列表中的第一个。

##### 类型擦除（Type erasure）

无论何时定义一个泛型类型，都自动提供类一个相应的原始类型（raw type）。原始类型的名字就是删去类型参数后的泛型类型名。擦除（erased）类型变量，并替换为限定类型（无限定的变量用Object）。

例如，`Pair<T>`的原始类型如下：

```java
public class Pair {
    private Object first;
    private Object second;
    
    public Pair(Object first,Object second) {
        this.first = first;
        this.second = second;
    }
    
    // getter and setter method
    public Object getFirst() {
        return this.first;
    }
    
    public Object getSecond() {
        return this.second;
    }
    
    public void setFirst(Object first) {
        this.first = first;
    }
    
    public void setSecond(Object second) {
        this.second = second;
    }
}
```

因为T是一个无限定的变量，所以直接用`Object`替换。在程序中可以包含不同类型的`Pair`，例如，`Pair<String>`或`Pair<LocalDate>`。而擦除类型后就变成原始的`Pair`类型了。

假如声明了一个限定的泛型类：

```java
public class Interval<T extends Comparable & Serializable> implements Serializable {
    private T lower;
    private T upper;
    // ...
    public Interval(T first,T second) {
        if(first.compareTo(second) < 0) {
            lower = first;
            upper = second;
        } else {
            lower = second;
            upper = first;
        }
    }
}
```

原始类型`Interval`如下所示：

```java
public Interval implements Serializable {
    private Comparable lower;
    private Comparable upper;
    //...
 	public Interval(Comparable first,Comparable second) {
    	//...
    }   
}
```

> 应该将标签（tagging）接口（即没有方法的接口）放在边界列表的末尾。

##### 通配符类型

通配符类型中，允许类型参数变化，例如，通配符类型`Pair<? extends Employee>`表示任何泛型`Pair`类型，它的类型参数是`Employee`的子类，如`Pair<Manager>`，但不是`Pair<String>`。

###### 无边界的通配符（Unbounded Wildcards）

无边界的通配符就是<?>，无边界的通配符的主要作用就是让泛型能够接受未知类型的数据。

```java
ArrayList<?>  list = new ArrayList<Long>();  
//or
ArrayList<?>  list = new ArrayList<String>();  
//or
ArrayList<?>  list = new ArrayList<Employee>();
```

有一点我们必须明确, **我们不能对List<?>使用add方法, 仅有一个例外, 就是add(null)**. 为什么呢? 因为我们不确定该`List`的类型, 不知道`add`什么类型的数据才对, 只有`null`是所有引用数据类型都具有的元素。

```java
public static void getTest(List<?> list) {
    // String s = list.get(0); // 编译报错
    // Integer i = list.get(1); // 编译报错
    Object o = list.get(2);
}
```

###### 上界通配符（Upper bounded wildcards）

假设我们想编写一个适用于`List<Integer>`和`List<Double>`的方法，则可以使用上界通配符来实现此目的，例如，我们可以指定`List<? extends Number>`，`Integer`和`Double`都是`Number`的子类。如果您希望泛型表达式接受特定类型的所有子类，则可以使用`extends`关键字来使用上界通配符。

```java
public class GenericsExample<T>
{
   public static void main(String[] args)
   {
      //List of Integers
      List<Integer> ints = Arrays.asList(1,2,3,4,5);
      System.out.println(sum(ints));
       
      //List of Doubles
      List<Double> doubles = Arrays.asList(1.5d,2d,3d);
      System.out.println(sum(doubles));
       
      List<String> strings = Arrays.asList("1","2");
      //This will give compilation error as :: The method sum(List<? extends Number>) in the 
      //type GenericsExample<T> is not applicable for the arguments (List<String>)
      System.out.println(sum(strings));
       
   }
    
   //Method will accept 
   private static Number sum (List<? extends Number> numbers){
      double s = 0.0;
      for (Number n : numbers)
         s += n.doubleValue();
      return s;
   }
}
```

来看另外一个例子：

```java
Pair<Manager> managerBuddies = new Pair<>(ceo,cfo);
Pair<? extends Employee> wildcardBuddies = managerBuddies; // OK
wildcardBuddies.setFirst(lowlyEmployee); // compile-time error
```

编译器只知道需要某个`Employee`的子类型，但不知道具体是什么类型。它拒绝传递任何特定的类型。毕竟`?`不能用来匹配。使用`getFirst`方法就不存在这个问题，将`getFirst`的返回值赋值给一个`Employee`的引用完全合法。

##### 下界通配符

如果希望泛型表达式接受特定类型的`super`类型或特定类型的父类的所有类型，则为此使用下界通配符，即使用`super`关键字。

```java
package test.core;
 
import java.util.ArrayList;
import java.util.List;
 
public class GenericsExample<T>
{
   public static void main(String[] args)
   {
      //List of grand children
      List<GrandChildClass> grandChildren = new ArrayList<GrandChildClass>();
      grandChildren.add(new GrandChildClass());
      addGrandChildren(grandChildren);
       
      //List of grand childs
      List<ChildClass> childs = new ArrayList<ChildClass>();
      childs.add(new GrandChildClass());
      addGrandChildren(childs);
       
      //List of grand supers
      List<SuperClass> supers = new ArrayList<SuperClass>();
      supers.add(new GrandChildClass());
      addGrandChildren(supers);
   }
    
   public static void addGrandChildren(List<? super GrandChildClass> grandChildren) 
   {
      grandChildren.add(new GrandChildClass());
      System.out.println(grandChildren);
   }
}
 
class SuperClass{
    
}
class ChildClass extends SuperClass{
    
}
class GrandChildClass extends ChildClass{
    
}
```

我们看到，`List<? super GrandChildClass>`可以调用`add`方法的，因为传入的`list`不管是什么，都一定是`GrandChildClass`或其父类泛型的`List`。但是我们不能使用`get`方法：

```java
public static void getTest(List<? super GrandChildClass> list) {
    GrandChildClass grandClildClazz = list.get(0); // 编译报错
    Object obj = list.get(1); // OK
}
```

因为我们所传入的类都是`GrandChildClass`的类或其父类，所传入的数据类型可能是`GrandChildClass`到`Object`之间的任何类型，这是无法预料的，也就无法接收。唯一能确定的就是`Object`，因为所有类型都是其子类型。

##### PECS原则

即`Producer Extends,Consumer Super`，来源参考《Effective Java》这本书。

+ 如果你只需要从集合中获得类型T , 使用<? extends T>通配符
+ 如果你只需要将类型T放到集合中, 使用<? super T>通配符
+ 如果你既要获取又要放置元素，则不使用任何通配符。例如`List<Integer>`

##### 泛型的约束与局限性

###### 不能使用静态类型的字段

不能在类中定义静态泛型参数化成员，任何这样做的尝试都会产生编译时错误：不能对非静态类型`T`进行静态引用。

```java
public class GenericsExample<T>
{
   private static T member; //This is not allowed
}
```

###### 不能实例化类型变量

不能使用像`new T(...)`，`new T[...]`或`T.class`这样的表达式中的类型变量；

```java
public class GenericsExample<T>
{
   public GenericsExample(){
      new T();
   }
}
```

###### 不能用基本类型实例化类型参数

不能声明像`List<int>`或`Map<String,double>`这样的通用表达式。可以使用包装类来代替基本类型，然后在传递实际值时使用基本类型。这些基本类型的值可以通过使用自动装箱将基本类型转换为相应的包装器类来接受。

```java
final List<int> ids = new ArrayList<>();    //Not allowed
 
final List<Integer> ids = new ArrayList<>(); //Allowed
```

###### 运行时类型查询只适用于原始类型

虚拟机中的对象总有一个特定的非泛型类型。因此，所有的类型查询只产生原始类型。

```java
if(a instanceof Pair<String>) // Error
if(a instanceof Pair<T>) // Error
Pair<String> p = (Pair<String>) a; // Warning--can only test that a is a Pair
```

同理，`getClass`方法总是返回原始类型。

```javascript
Pair<String> stringPair = ...;
Pair<Employee> employeePair = ...;
if(stringPair.getClass() == employeePair.getClass()) // they are equal
```

其比较结果是`true`，这是因为两次调用`getClass`都将返回`Pair.class`。

###### 不能抛出或捕获泛型类的实例

既不能抛出也不能捕获泛型类对象。实际上，甚至泛型类扩展`Throwable`都是不合法的。

```java
// causes compiler error:The generic class GenericException may not subclass java.lang.Throwable.
public class GenericException<T> extends Exception {}
```

`catch`子句中不能使用类型变量。

```java
public static <T extends Throwable> void doWork(Class<T> t)
{
    try
    {
    	do work
    } catch (T e) // Error can't catch type variable 
    {
        Logger, global .info(...);
    } 
}
```

不过， 在异常规范中使用类型变量是允许的。 以下方法是合法的：

```java
public static <T extends Throwable> void doWork(T t) throws T // OK
{
    try {
        do work
    } catch(Throwable realCause) {
        t.initCause(realCause);
        throw t;
    }
}
```

##### 参考资料

[Complete Java Generics Tutorial](https://howtodoinjava.com/java/generics/complete-java-generics-tutorial)

[Java核心技术·卷 I（原书第10版）](https://book.douban.com/subject/26880667/)

[Effective Java](https://book.douban.com/subject/3998727/)

[深入理解Java泛型](https://juejin.im/post/5b614848e51d45355d51f792#heading-3)
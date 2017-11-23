### 反射

反射（`Reflection`）是指程序在**运行时**可以访问、检测和修改它本身状态或行为的一种能力。

Java反射框架主要提供以下功能：

- 在运行时判断任意一个对象所属的类

- 在运行时构造任意一个类的对象

- 在运行时判断任意一个类所具有的成员变量和方法（通过反射甚至可以调用private方法）

- 在运行时调用任意一个对象的方法

**重点：是运行时而不是编译时**

在`JDK`中主要由以下类来实现Java反射机制，这些类都位于`java.lang.reflect`包中。在Java中，无论生成某个类的多少个对象，这些对象都会对应同一个`Class`对象。
+ 如果要使用反射，就要获得操作这个类所对应的`Class`对象 
+ 如果要调用方法，就要获得与这个方法所对应的`Method`对象  
+ 如果要使用属性，就要获得与这个属性所关联的`Field`对象 
+ 如果要调用构造方法，就要获得与这个构造方法对应的`Constructor`对象

#### 获得Class对象

要想使用反射，首先需要获得待处理的类或对象所对应的`Class`对象。获得某个类或对象所对应的`Class`的对象有三种常用的方法

(1)使用`Class类`的`forName`静态方法:`Class.forName("java.lang.String");`

(2)使用类的.class语法，比如:`Class clazz = String.class;`

(3)调用某个对象的`getClass()`方法,比如:

```java
String str = "hello";
Class<?> clazz = str.getClass();
```

#### 创建实例

1、若想通过类的**不带参数的构造方法**来生成对象，我们有两种方式。

a)先获得`Class`对象，然后通过该`Class`对象的`newInstance()`方法直接生成即可：

```java
Class<?> clazz = String.class;
 Object obj = clazz.newIntence();
```

b)先获得`Class`对象，然后通过该对象获得对应的`Constructor`对象，再通过该`Constructor`对象的`newInstance()`方法生成：

```java
Class<?> clazz = Customer.class;
Constructor cons = clazz.getConstructor(new Class[] {});
Object obj = cons.newInstance(new Object[] {});
```

2、若想通过类的带参数构造方法生成对象，只能使用下面这一种方式：

```java
Class<?> clazz = Customer.class;
Constructor cons = clazz.getConstructor(new Class[] {String.class,int.class});
Object obj = cons.newInstance(new Object[] {"hello",3});
```

#### 获取方法

获取某个Class对象的方法集合，主要有以下几个方法：

`getDeclaredMethods()`方法返回类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法。

```java
public Method[] getDeclaredMethods() throws SecurityException
```

`getMethods()`方法返回某个类的所有公用（`public`）方法，包括其继承类的公用方法。

```java
public Method[] getMethods() throws SecurityException
```

`getMethod`方法返回一个特定的方法，其中第一个参数为方法名称，后面的参数为方法的参数对应Class的对象

```java
public Method getMethod(String name, Class<?>... parameterTypes)
```

#### 调用方法

当我们从类中获取了一个方法后，我们就可以用invoke()方法来调用这个方法。invoke方法的原型为:

```java
public Object invoke(Object obj, Object... args)
        throws IllegalAccessException, IllegalArgumentException,
           InvocationTargetException
```

下面是一个实例：

```java
package com.flwcy.reflect;

import org.junit.Test;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class InvokerTest {
    @Test
    public void test() throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        // 反射获得class对象
        Class<?> clazz = Class.forName("com.flwcy.reflect.MethodClass");//MethodClass.class;
        // 通过class对象获取实例
        Object obj = clazz.newInstance();
        // 通过class获取某个方法，第一个参数为方法名，第二个参数为方法所接受参数的class对象
        Method method = clazz.getMethod("add",new Class[]{int.class,int.class});
        //通过method调用方法并传递参数，第一个参数为调用该方法的对象，第二个参数为方法的实际参数
        Object result = method.invoke(obj,new Object[]{3,4});// add(3,4);
        System.out.println(result);

    }
}

class MethodClass {
    public int add(int a,int b){
        return a + b;
    }
}
```

另外，使用反射机制可以调用对象的私有方法、访问对象的私有成员变量，因此可能会破坏封装性而导致安全问题。

```java
        // 反射调用私有方法
        Class<?> clazz = MethodClass.class;
        // getDeclaredConstructor以调用任何类型声明的构造方法
        Constructor<?> constructor = clazz.getDeclaredConstructor(new Class[]{});
        Object obj = constructor.newInstance(new Object[]{});
        Method method = clazz.getDeclaredMethod("sub",new Class[]{int.class,int.class});
        // 忽略权限检查
        method.setAccessible(true);
        System.out.println(method.invoke(obj,new Object[]{3,4}));
```

实现一个`JavaBean`的拷贝：

```java
    public Object copy(Object oldObj) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        Object newObj = null;
        Class<?> clazz = oldObj.getClass();
        // 获取构造方法
        Constructor<?> constructor = clazz.getDeclaredConstructor(new Class[]{});
        // 创建实例
        newObj = constructor.newInstance(new Object[]{});
        // 获得属性数组
        Field[] fields = clazz.getDeclaredFields();
        for(Field field : fields){
            // 获得属性名称
            String fieldName = field.getName();
             // 首字母大写
            String firstLetter = fieldName.substring(0,1).toUpperCase();
            String getMethodName = String.format("get%s%s",firstLetter,fieldName.substring(1));
            String setMethodName = String.format("set%s%s",firstLetter,fieldName.substring(1));

            Method getMethod = clazz.getDeclaredMethod(getMethodName,new Class[]{});

            Method setMethod = clazz.getDeclaredMethod(setMethodName,new Class[]{field.getType()});

            // 执行get方法获取值
            Object value = getMethod.invoke(oldObj,new Object[]{});
            // 调用set方法
            setMethod.invoke(newObj,new Object[]{value});
        }
        return newObj;
    }
```

#### 利用反射创建数组



### Read More

[深入解析Java反射（1） - 基础](http://www.sczyh30.com/posts/Java/java-reflection-1/#%E4%B8%80%E3%80%81%E5%9B%9E%E9%A1%BE%EF%BC%9A%E4%BB%80%E4%B9%88%E6%98%AF%E5%8F%8D%E5%B0%84%EF%BC%9F)
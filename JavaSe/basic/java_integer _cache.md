#### Integer Cache

先看一段代码：

```java
package com.flwcy.test;

public class Test {
    public static void main(String[] args) {
        Integer integer1 = 3;
        Integer integer2 = 3;

        System.out.println(integer1 == integer2);
        Integer integer3 = 300;
        Integer integer4 = 300;

        System.out.println(integer3  == integer4);
    }
}
```

运行结果为`true  false`，这是为什么呢？让我们反编译下`Test.class`文件看看：

```java
package com.flwcy.test;

import java.io.PrintStream;

public class Test
{
  public static void main(String[] args)
  {
    Integer integer1 = Integer.valueOf(3);
    Integer integer2 = Integer.valueOf(3);

    System.out.println(integer1 == integer2);
    Integer integer3 = Integer.valueOf(300);
    Integer integer4 = Integer.valueOf(300);

    System.out.println(integer3 == integer4);
  }
}
```

我们反编译后发现`Integer integer1 = 3`；实际上变成了`Integer integer1 = Integer.valueOf(3);`，看下`valueOf`方法的实现：

```java
    /**
     * Returns an {@code Integer} instance representing the specified
     * {@code int} value.  If a new {@code Integer} instance is not
     * required, this method should generally be used in preference to
     * the constructor {@link #Integer(int)}, as this method is likely
     * to yield significantly better space and time performance by
     * caching frequently requested values.
     *
     * This method will always cache values in the range -128 to 127,
     * inclusive, and may cache other values outside of this range.
     *
     * @param  i an {@code int} value.
     * @return an {@code Integer} instance representing {@code i}.
     * @since  1.5
     */
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

上述代码意思是在一个区间之内，直接用`IntegerCache.cache[]`数组里面的数返回，否则`new`一个新对象。

```java
    /**
     * Cache to support the object identity semantics of autoboxing for values between
     * -128 and 127 (inclusive) as required by JLS.
     *
     * The cache is initialized on first usage.  The size of the cache
     * may be controlled by the {@code -XX:AutoBoxCacheMax=<size>} option.
     * During VM initialization, java.lang.Integer.IntegerCache.high property
     * may be set and saved in the private system properties in the
     * sun.misc.VM class.
     */
	// IntegerCache是一个静态内部类，该类只能在Integer这个类的内部访问。
    private static class IntegerCache {
        static final int low = -128; //区间的最低值
        static final int high; // 区间的最高值，后面默认赋值为127，也可以用户手动设置虚拟机参数
        static final Integer cache[]; // jdk事先缓存的Integer

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127); //虽然设置了但是还是不能小于127
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1); // 也不能超过最大值
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            // 循环将区间的数赋值给cache[]数组
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```

`Javadoc`详细的说明这个类是用来实现缓存支持，并支持`-128`到`127`之间的自动装箱过程。缓存在首次使用时初始化。最大值`127`可以通过`JVM`的启动参数`-XX:AutoBoxCacheMax=size`修改。

因此如果是处在`-128~127`之间的数直接从缓存数组中取，否则才构造新的`Integer`对象。在`Java`中，`==`比较的是对象引用，无法对`==`操作符进行重载，而对于`equals`方法，`Integer`里面的`equals`方法重写了`Object`的`equals`方法，查看`Integer`源码可以看出`equals`方法进行的是数值比较。

```java
    /**
     * Compares this object to the specified object.  The result is
     * {@code true} if and only if the argument is not
     * {@code null} and is an {@code Integer} object that
     * contains the same {@code int} value as this object.
     *
     * @param   obj   the object to compare with.
     * @return  {@code true} if the objects are the same;
     *          {@code false} otherwise.
     */
    public boolean equals(Object obj) {
        if (obj instanceof Integer) {
            return value == ((Integer)obj).intValue();
        }
        return false;
    }
```


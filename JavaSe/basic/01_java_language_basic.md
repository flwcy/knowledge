#### Java语言基础

关键字

标识符：

+ 英文大小写字母、数字、下划线、$
+ 不能以数字开头
+ 不能是java中的关键字
+ Java严格区分大小写

注释：

+ 单行 //
+ 多行 /*          */
+ 文档注释 /**        */

Java语言是强类型语言

基本数据类型：byte(1)、short(2)、int(4)、long(8)、float(4)、double(8)、char(2)、boolean(1)

引用数据类型：class、interface、[]

整数默认是int类型

浮点数默认是double类型

长整型用L或者l标记，建议使用L

单精度浮点数用F或者f标记，建议使用F

数据类型转换：byte、short、char ---- int --- long --- float --- double

运算符

```java
int x = 3; // 把3赋值给int类型的变量;
```

--、++ 数字打头，先使用再计算。符号打头先计算再使用。

> a++是先办事（进行相关运算），后给钱（然后再将a加1）
>
> ++a是先给钱（先给a加1），后办事（然后参与运算）

> 对于整数a，b来说，取模运算或者求余运算的方法要分如下两步：
>
> 1.求整数商：c=a/b
>
> 2.计算模或者余数：r=a-(c*b)
>
> 求模运算和求余运算在第一步不同
>
> 取余运算在计算商值向0方向舍弃小数位
>
> 取模运算在计算商值向负无穷方向舍弃小数位
>
> 例如：4/(-3)约等于-1.3
>
> 在取余运算时候商值向0方向舍弃小数位为-1
>
> 在取模运算时商值向负无穷方向舍弃小数位为-2
>
> 所以
>
> 4rem(-3)= 4-(-1 * -3) =1
>
> 4mod(-3)= 4- (-2 * -3) = -2

整数相除只能得到整数，如果想得到小数，必须把数据转化为浮点数。`x * 1.0 / y`

> 面试题：
>
> short s = 1; s = s + 1;  // 编译不通过的，提示损失精度 
>
> 隐式类型转换可以从小到大自动转换，即byte→short→int→long；反过来会丢失精度，必须进行显示类型转换；
>
> s = s+1这句先执行s+1然后把结果赋给s，由于1为int类型，所以s+1的返回值是int，编译器自动进行了隐式类型转换；所以将一个int类型赋给short就会出错。
>
> short s = 1; s += 1;
>
> s += 1; // 不是等价于 s = s + 1; 而是等价于 s = (s的数据类型)(s + 1);

逻辑运算符：&&（短路与），&（与），|（或），||（短路或），!（逻辑非）

+ `&&`和`&`都是表示“与”，区别是`&&`只要第一个条件为`false`，则后面条件就不再判断。而`&`要对所有的条件都进行判断。
+ `||`（短路或）和`|`（或）都是表示“或”，区别是`||`只要第一个条件为`true`，则后面的条件不再判断 ，而`|`要对所有的条件进行判断。


三目运算符:其中"(a<b)?a:b"是一个"条件表达式",它是这样执行的:如果a<b为真,则表达式取a值,否则取b值.

> 键盘录入？
>
> import java.util.Scanner;
>
> Scanner sc = new Scanner(System.in);
>
> int x = sc.nextInt();

`if`语句、`switch`语句、
### [译]到底什么是词法作用域

> [What is Lexical Scope Anyway?](http://astronautweb.co/javascript-lexical-scope/)

JavaScript中一个相对基本的概念是每个声明的函数都创建自己的作用域。更让人费解的是闭包的概念——一个能够记住并访问其词法作用域的函数，即使该函数在其词法作用域之外执行。

词法作用域是一种在JavaScript语言中使用的作用域模型，它不同于其他使用动态作用域的语言。词法作用域是词法分析阶段定义的作用域。

#### 那么，什么是词法分析阶段？

本文将深入研究JavaScript引擎的工作机制。尽管通常被称为解释语言，JavaScript在执行代码之前立即编译代码。例如语句`var a= 2;`在词法分析阶段被分为两个单独的步骤：

+ `var a`这在代码执行之前声明了作用域内的变量。
+ `a = 2`如果在可用作用域内找到，则将值2赋给变量a。

编译的词法分析阶段决定了在哪里以及如何声明所有的标识符。以及在执行期间如何查找它们。这是导致“提升”变量的相同机制。变量实际上并未在源代码中移动，声明仅仅发生在词法分析阶段，因此JavaScript引擎在执行之前就知道这些声明。

考虑这些例子：

示例1：

```javascript
var a = 1;
console.log('a:', a); // a: 1
```

示例2：

```javascript
console.log('a:', a); // a: undefined
var a = 1;
```

示例3：

```javascript
console.log('a:', a); // Uncaught ReferenceError: a is not defined
```

示例1很简单并且按预期工作，但请注意其他两个例子之间的细微差别。示例2打印了a的值为undefined，但是标识符a本身已被声明；与没有声明标识符a的示例3相比，这会导致引用错误。

这说明在词法分析阶段，JavaScript引擎首先声明变量，然后在接下来的步骤中将值分配给标识符——这就是提升。因为函数也是在这个时候定义的（词法分析阶段），我们可以说，词法作用域是由写代码时变量和块级作用域的位置来决定的，因此在词法分析阶段结束时被锁定。作用域不是在运行时定义的，而是可以在运行时访问它。

同样，闭包是指函数能够记住并访问其词法作用域，即使该函数在其词法作用域之外执行也是如此。

```javascript
function foo() {  // 'scope of foo' aka lexical scope for bar
   var memory = 'hello closure';
   return function bar() {
      console.log(memory);
   }
}
 
// returns the bar function and assigns it to the identifier 'closure’;
const closure = foo();
 
closure(); // hello closure
```

所以...词法作用域是由闭包创建的书写代码时的作用域。它是闭包内定义的函数的“外部”作用域。

> 外部函数的函数作用域 === 内部函数的词法作用域
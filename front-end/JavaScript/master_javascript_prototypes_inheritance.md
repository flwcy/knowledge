### [译]掌握JavaScript原型和继承

> [Master JavaScript Prototypes & Inheritance](https://codeburst.io/master-javascript-prototypes-inheritance-d0a9a5a75c4e)

#### 继承

继承指的是对象从另一个对象访问方法和其他属性的能力。对象能够从其他对象中继承东西。JavaScript中的继承是通过一种叫做原型的东西来工作的，这种继承形式通常称为原型继承。

在这篇文章中，我们将介绍许多看似无关的主题，并在最后将它们联系在一起。最后还有一个太长(若)不看(请看这里)，给那些想要简短版本的人。

#### 对象，数组和函数

JavaScript给我们提供了三个全局函数：`Object`，`Array`和`Function`。是的，它们都是函数。

```javascript
console.log(Object); // -> ƒ Object() { [native code] }
console.log(Array); // -> ƒ Array() { [native code] }
console.log(Function); // -> ƒ Function() { [native code] }
```

您不知道它，但每次创建对象字面量时，JavaScript引擎就会有效地调用`new Object()`。一个对象字面量是一个通过编写`{}`创建的对象，如`var obj = {};`。因此，对象字面量是对`Object`的隐式调用。

数组和函数也是如此。我们可以认为数组来自`Array`构造函数，而函数来自`Function`构造函数。

#### 对象原型

##### `__proto__`

所有的JavaScript对象都有一个原型。浏览器通过`__proto__`属性实现原型，这就是我们引用它的方式。这通常被成为`dunder proto`，双下划线原型的简写。永远不要重新分配这个属性或直接使用它。`__proto__`的[MDN页面](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)用红色的大方块警告我们永远不要这样做。

##### 原型

函数还有一个`prototype`属性。这不同于它们的`__proto__`属性。这使得讨论相当混乱，因此我将阐明我将使用的语法。当我提到原型并且`prototype`这个词没有突出显示为灰色时，我指的是`__proto__`属性。当我使用灰色的`prototype`时，我指的是函数的`prototype`属性。

如果我们在Chrome中记录对象的原型，这就是我们所看到的。

```javascript
var obj = {};
console.log(obj.__proto__);
// -> {constructor: ƒ, __defineGetter__: ƒ, …}
```

`__proto__`属性是对另一个具有多个属性的对象的[引用](https://codeburst.io/explaining-value-vs-reference-in-javascript-647a975e12a0)。我们创建的每个对象字面量都有`__proto__`属性指向同一个对象。一个对象字面量是一个通过编写`{}`创建的对象，如`var obj = {};`。

有几点很重要：

+ 对象字面量的`__proto__`等于`Object.prototype`
+ `Object.prototype`的`__proto__`是`null`

我们很快就会解释原因。

#### 原型链

要理解对象原型，我们需要讨论对象的查找行为。当我们寻找对象的属性时，JavaScript引擎首先检查对象本身是否存在属性。如果没找到，它将转到对象的原型并检查该对象。如果找到，它将使用该属性。

如果没有找到，它将转到原型的原型，直到找到一个`__proto__`属性等于`null`的对象。因此，如果我们试图从上面的obj对象上查找属性`someProperty`，引擎将首先检查对象本身。

它找不到它，然后转到它的`__proto__`对象，它等于`Object.prototype`。它也不会在那里找到它，当看到下一个`__proto__`是`null`时，它将返回`undefined`。

这被称为原型链，它通常被描述为一个向下的链条，在顶部是`null`，在底部是我们使用的对象。

执行查找时，引擎将遍历链查找属性并返回它找到的第一个属性，如果原型链中没有，则为`undefined`。

```javascript
__proto__ === null
|
|
__proto__ === Object.prototype
|
|
{ object literal }
```

这可以证明。在这里，我们将直接使用`__proto__`进行演示。再说一次，永远不要这样做。

```javascript
var obj = {};
obj.__proto__.testValue = 'Hello!';
console.log(obj); // -> {}
console.log(obj.testValue); // -> Hello!
```

该原型链如下所示。

```javascript
__proto__ === null
|
|
__proto__ === Object.prototype -> testValue: 'Hello!'
|
|
obj
```

当我们打印`obj`时，我们得到一个空对象，因为`testValue`没有直接显示在对象上。然而，打印`obj.testValue`会触发查找。引擎沿着原型链向上，在对象的原型上找到`testValue`，然后我们看到这个值打印出来。

#### hasOwnProperty

在对象上有一个方法叫做`hasOwnProperty`。它将根据对象本身是否包含被测试的属性返回`true`或`false`。但是，测试`__proto__`将始终返回`false`。

```javascript
var obj = {};
obj.__proto__.testValue = 'Hello!';
console.log(obj.hasOwnProperty('testValue'));
// -> false
console.log(obj.__proto__.hasOwnProperty('testValue'));
// -> true
```

#### 函数原型

如上所述，函数都有一个与`__proto__`属性不同的`prototype`属性。它是一个对象。一个函数的`prototype`的`__proto__`属性等于`Object.prototype`。换一种说法：

```javascript
function fn() {}
console.log(fn.prototype.__proto__ === Object.prototype);
// -> true
```


### [译]尽可能简单的解释JavaScript的“new”关键字

> [Javascript’s “new” Keyword Explained as Simply as Possible](https://codeburst.io/javascripts-new-keyword-explained-as-simply-as-possible-fec0d87b2741)

#### 普通的函数调用

为了解释new的工作原理，让我们从一个不使用new的普通函数开始。我们想写一个能够创建“person”对象的函数。它将根据所接受的参数为这些对象提供`name`和`age`属性。

```javascript
function personFn(name, age) {
    var personObj = {};
    personObj.name = name;
    personObj.age = age;
    
    return personObj;
}
var alex = personFn('Alex', 30);
// -> { name: 'Alex', age: 30 }
```

很简单。我们创建了一个对象，为它添加属性，最后返回它。

#### new

![object_oriented_programming.png](../../img/html_css_js/object_oriented_programming.png)

让我们创建一个做相同事情的方法，但是我们想通过`new`关键字调用它。此方法将创建与上面相同的对象。通常的做法是首字母大写的函数表示使用`new`调用。这些函数也被称为**构造函数**。

```javascript
function PersonConstructor(name, age) {
    this.name = name;
    this.age = age;
}
var alex = new PersonConstructor('Alex', 30);
// -> { name: 'Alex', age: 30 }
```

正常调用`personFn`和使用`new`调用`PersonConstructor`都会导致创建相同的对象。这是怎么回事？

`new`关键字以特殊方式调用函数。它添加了一些我们看不到的隐式代码。让我们展开上面的函数来显示正在发生的一切。注释行是伪代码，表示使用`new`时JS引擎添加的功能。

```JavaScript
function PersonConstructor(name, age) {
    // this = {};
    // this.__proto__ = PersonConstructor.prototype;
    
    // 设置如下逻辑: 如果这里有一个return语句
    // 在函数体中返回除了一个对象，数组或者函数之外的
    // 任意东西：返回“this”（最新的构造对象）而不是return语句；
    this.name = name;
    this.age = age;
    // return this;
}
```

让我们把它分解一下。`new`：

1. 创建一个新的对象并将其绑定到`this`关键字。
2. 设置对象的内部[[Prototype]]属性，`__proto__`设置为构造函数的原型。这也使得新对象的构造函数是原型继承的。
3. 设置逻辑，以便在函数体中返回除对象、数组或函数之外的任何类型的变量时，返回`this`，返回新构造的对象，而不是函数要返回的内容。
4. 在函数末尾，如果函数体中没有`return`语句，则返回`this`。

让我们逐一证明这些陈述是有效的。

```javascript
function Demo() {
    console.log(this);
    this.value = 5;
    return 10;
}
/*1*/ var demo = new Demo(); // -> {}
/*2*/ console.log(demo.__proto__ === Demo.prototype); // -> true
      console.log(demo.constructor === Demo); // -> true
/*3*/ console.log(demo); // -> { value: 5 }
function SecondDemo() {
    this.val = '2nd demo';
}
/*4*/ console.log(new SecondDemo()); // -> { val: '2nd demo' }
```

如果您不熟悉构造函数或原型，请不要过于担心。您将在继续学习JavaScript时遇到它们。现在，只需要理解由构造函数隐式返回的新对象将能够继承属性和方法。

#### 使用new调用非构造函数

如果我们使用`new`调用一个像`personFn`这样的普通函数会发生什么？没什么特别的。适用相同的规则。对于`personFn`来说，我们看不到任何明确的事情发生。

```javascript
var alex = new personFn('Alex', 30);
// -> { name: 'Alex', age: 30 }
```

为什么？让我们在`personFn`中添加我们的隐式代码。

```javascript
function personFn(name, age) {
    // this = {};
    // this.constructor = PersonConstructor;
    // this.__proto__ = PersonConstructor.prototype;

    // 设置如下逻辑: 如果这里有一个return语句
    // 在函数体中返回除了一个对象，数组或者函数之外的
    // 任意东西：返回“this”（最新的构造对象）而不是return语句；
    var personObj = {};
    personObj.name = name;
    personObj.age = age;
    
    return personObj;
    
    // return this;
}
```

隐式代码仍然添加在：

+ 它将`this`绑定到一个新对象并设置其构造函数和原型。
+ 它添加了返回`this`而不是非对象的逻辑。
+ 它在最后添加了一个隐含的`return this`。

这不会影响我们的代码，因为我们在代码中没有使用`this`关键字。我们还明确地返回一个对象，`personObj`，所以返回逻辑并`return this`没有用。实际上，使用`new`来调用我们的函数对输出没有影响。如果我们使用`new`或者我们并没有返回一个对象，当使用或不使用`new`时，函数将具有不同的效果。

如果觉得这篇文章对您有用，请点击❤，并随时订阅和查看我的其他一些文章。

[Master Map & Filter, Javascript’s Most Powerful Array Functions](https://codeburst.io/array-functions-map-filter-18a6e5f75da1)

[Master the Power Behind Javascript’s Logical Operators](https://codeburst.io/javascript-and-logical-operators-89b2ac3409f8)

[Master Javascript’s New, Cutting-Edge Object Spread Operator](https://codeburst.io/master-javascripts-object-spread-operator-3803430e99aa)

**仅此而已，去写一些代码吧。**
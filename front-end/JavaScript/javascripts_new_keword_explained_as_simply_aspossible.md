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
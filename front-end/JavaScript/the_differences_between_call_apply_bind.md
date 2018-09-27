### [译]call/apply/bind之间的区别

> [The difference between call / apply / bind](https://medium.com/@ivansifrim/the-differences-between-call-apply-bind-276724bb825b)

JavaScript是一门动态语言，并且足够灵活，可以让您执行诸如多继承之类的操作。这时，对象或类可以从多个父类继承特征。这可以使用以下三种方法之一：call/apply/bind。请阅读以下内容，或从[此处](https://github.com/ivansifrim/call-apply-bind)克隆代码亲自尝试一下。

它们之间有什么区别呢？

#### Call

假设我们有一个名为**obj**的对象。它仅有一个名为things的属性，其值为3。让我们创建一个与这个对象完全无关的名为addThings的函数。

```javascript
let obj = {things: 3};
let addThings = function(a, b, c){
 return this.things + a + b + c;
};
```

请注意`this.things`。为什么addThings函数会提到这个，甚至它从来没有things？我们需要传递给它一个上下文。我们能够通过call来实现这个。如果我们执行下面的代码：

```javascript
console.log( addThings.call(obj, 1,4,6) );
```

它将会返回数字14。

那是因为call的第一个参数是我们想要**"this"**引用的上下文。我们将拥有**things**属性的**obj**传递给它。在我们传递上下文之后，我们可以像平常一样传递参数。在当前情况下，我们传递了**1,4,6**。所以这一行：

```javascript
return this.things + a + b + c;
```

将以这种方式填充：

```javascript
return 3 + 1 + 4 + 6;
```

结果是14。

这就是**call**的作用！让我们来看看**apply**方法。

#### Apply

**apply**与**call**是如此相似，我个人并不认为它有什么价值。一语双关。

我将解释它，因为你可能需要在工作面试中了解它。主要的区别就是我们传递参数的方式，我们可以将它们作为数组传递。从脑海中清除前面的代码，让我们重新开始。

```javascript
let obj = {things: 3};
let addThings = function(a, b, c){
 return this.things + a + b + c;
};
let arr = [1,4,6];
console.log( addThings.apply(obj, arr) );
```

现在让我们来看看Bind。

#### Bind

```javascript
let obj = {things: 3};
let addThings = function(a, b, c){
 return this.things + a + b + c;
};
console.log( addThings.bind(obj, 1,4,6) );
```

我们期望输出数字14，但是没有用。相反，它返回了一个函数。Bind的工作方式是返回函数的一个副本，但是使用不同的上下文。我们传递**obj**作为上下文，但是我们没有执行它，让我们试试：

```javascript
console.log( addThings.bind(obj, 1,4,6)() );
```

它有效果了！我们也可以这样传递参数：

```javascript
console.log( addThings.bind(obj)(1,4,6) );
```

好了！现在你知道了Call/Apply/Bind。现在去应用它，快乐的编码（囧），

Ivan
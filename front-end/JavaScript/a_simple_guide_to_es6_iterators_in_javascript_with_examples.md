### [译]JavaScript中ES6迭代器的简单指南和示例

> [A Simple Guide to ES6 Iterators in JavaScript with Examples](https://codeburst.io/a-simple-guide-to-es6-iterators-in-javascript-with-examples-189d052c3d8e)

我们将在本文中分析迭代器。**迭代器是循环JavaScript中任何集合的一种新方法。**它们是在ES6中引入的，由于广泛的用途和在不同的地方使用而变得非常流行。

我们准备通过例子来从概念上理解迭代器是什么以及在哪里使用它们。我们还将在JavaScript中看到它的一些实现。

#### 介绍

想象你需要这样一个数组——

```javascript
const myFavouriteAuthors = [
  'Neal Stephenson',
  'Arthur Clarke',
  'Isaac Asimov', 
  'Robert Heinlein'
];
```

在某些时候，你需要返回数组中所有单个值，以便在屏幕中打印它们、操纵它们，或者对它们执行一些操作。如果我问你，你会怎么做？你也许会说——这很简单。我会使用`for`，`while`，`for-of`或者[其中](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration)一种循环方法来循环它们。就像下面的例子所实现的那样——

```javascript
// For loop
for (let index = 0; index < myFavouriteAuthors.length; index++) {
  console.log(myfavouriteAuthors[index]);
}

// while loop
let index = 0;
while (index < myFavouriteAutors.length) {
  console.log(myFavouriteAuthors[index]);
  index++;
}

// for-of loop
for(const value of myFavouriteAuthors) {
  console.log(value);
}
```

现在，想象一下，你有一个自定义数据结构来代替前面的数组，用于保存所有作者。就像这样——

```javascript
const myFavouriteAuthors = {
  allAuthors: {
    fiction: [
      'Agatha Christie',
      'J. K. Rowlling',
      'Dr. Seuss'
    ],
    scienceFiction: [
      'Neal Stephenson',
      'Arthur Clarke',
      'Isaac Asimov',
      'Robert Heinlein'
    ],
    fantasy: [
      'J. R. R. Tolkien',
      'J. K. Rowling',
      'Terry Pratchett'
    ],
  },
}
```

现在，`myFavouriteAuthors`是一个包含另一个对象`allAuthors`的对象。`allAuthors`对象包含三个key值为`fiction`，`scienceFiction`和`fantasy`的数组。**现在，如果我要求你循环`myFavouriteAuthors`来获取所有作者，你的方法是什么？**你可以继续尝试一些循环的组合来获取所有的数据。

然而，如果你像这样做的话——

```javascript
for (let author of myFavouriteAuthors) { 
  console.log(author)
}
// TypeError: {} is not iterable
```

你会得到一个`TypeError`说这个对象是不可迭代的。**让我们看看什么是迭代，以及如何使对象可迭代。**在这篇文章的结尾，你将了解怎么在自定义对象上使用`for-of`循环，在本例中，在`myFavouriteAuthors`上。

#### 迭代和迭代器

你在上一节中看到了问题。没有简单的方法可以从我们自定义对象中获取所有作者。我们需要某种方法，通过它我们可以顺序地公开所有内部数据。

让我们在`myFavouriteAuthors`中添加一个返回所有作者的方法`getAllAuthors`，就像这样——

```javascript
const myFavouriteAuthors = {
    allAuthors: {
        ...
    },
    getAllAuthors() {
        const authors = [];
        for (const author : of this.allAuthors.fiction) {
            authors.push(author);
        }
        
        for (const author of this.allAuthors.scienceFiction) {
            authors.push(author);
        }
        
        for(const author : of this.allAuthors.fantasy) {
            authors.push(author);
        }
    },
};
```

这是一个简单的方法。它完成了我们当前获取所有作者的任务。但是，这种实现可能会出现一些问题。有些是——

+ `getAllAuthors`这个名字非常具体。如果其他人想要创建自己的`myFavouriteAuthors`，他们可能将其命名为`retrieveAllAuthors`。
+ 作为开发人员，我们总是需要知道返回所有数据的特定方法。在本例中，它名为`getAllAuthors`。
+ `getAllAuthors`返回一个所有作者的字符串数组。如果其他开发人员想返回一个像如下格式的对象数组，该怎么办——

```
[ {name: 'Agatha Christie'}, {name: 'J. K. Rowling'}, ... ]
```

开发人员需要知道返回所有数据的方法的**确切名称和返回类型。**

如果我们制定一个**规则**，方法的**名称**和**返回类型**都是**固定的和不可改变的。**该怎么办？

让我们将这个方法命名为——**iteratorMethod。**

**[ECMA](https://en.wikipedia.org/wiki/Ecma_International)**采用类似的步骤来标准化循环自定义对象的过程。但是，ECMA没有使用`iteratorMethod`这个名字，而是使用`Symbol.iterator`这个名字。[Symbols](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol)提供唯一且不与其他属性名称冲突的名称。当然，`Symbol.iterator`将会**返回一个叫做`iterator`的对象。**这个`iterator`将会拥有一个名为`next`的方法，它将返回一个键为`value`和`done`的对象。

`value`键包含当前的值。它可以是任意类型。`done`的类型是boolean，它表示是否已获取所有值。

一张图也许便于帮助我们建立**iterables**，**iterators**和**next**之间的关系。**这种关系称为迭代协议。**

![Relationship between iterables, iterators, and next.](../../img/html_css_js/relationship_between_iterables_iterators_and_next.png)

根据[Axel Rauschmayer](https://medium.com/@rauschma)的书[Exploring JS](http://exploringjs.com/es6/ch_iteration.html)——

+ **迭代**是一种数据结构，它希望公众可以访问其元素。它通过实现一个键为`Symbol.iterator`的方法来实现。该方法是**迭代器**的工厂，也就是说，它将创建**迭代器。**
+ **迭代器**是用于遍历数据结构元素的指针。

#### 创建迭代对象

正如我们上一节所学习到的，我们需要实现一个名为`Symbol.iterator`的方法。我们将使用[计算属性语法](http://es6-features.org/#ComputedPropertyNames)来设置该键。一个简短的例子——

![Example of iterable](../../img/html_css_js/example_of_iterable.png)

在第4行，我们创建了一个迭代器。它是一个默认带有`next`方法的对象。`next`方法通过变量`step`来返回值。在第25行，我们检索迭代器。第27行，我们调用`next`方法。我们一直调用`next`方法直到`done`为`true`为止。

这正是`for-of`循环中发生的事情。`for-of`循环接受一个**迭代（iterable）**，并创建它的**迭代器（iterator）。**它一直调用`next`方法直到`done`为`true`为止。

#### JavaScript中的迭代


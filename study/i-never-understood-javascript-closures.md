### [译]我从未理解过的JavaScript闭包

> [i-never-understood-javascript-closures](https://medium.com/dailyjs/i-never-understood-javascript-closures-9663703368e8)

​        正如标题所述，JavaScript闭包对我来说一直是一个迷，我看了许多文章，在我工作中我也使用过闭包，有时候我甚至使用了闭包而没意识到我正在使用闭包。

​        最近我去参加一个演讲，有人用一种让我感兴趣的方式来解释它。我也准备在这篇文章中通过这种方式来解释闭包。让我向[CodeSmith](https://www.codesmith.io/)的优秀成员以及他们的*JavaScript The Hard Parts*系列表示敬意。

#### 在我们开始之前

在理解闭包之前我们需要理解一些重要的概念，其中之一就是**执行环境**。这篇文章[What is the Execution Context & Stack in JavaScript?](http://davidshariff.com/blog/what-is-the-execution-context-in-javascript/#first-article)对执行环境有一个很好的说明，
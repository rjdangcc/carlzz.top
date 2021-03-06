---
title: js 闭包原理剖析
id: 32
categories:
  - 前端
  - 技术
date: 2017-07-03 10:37:40
tags:
---

一直以来，`javascript`中的闭包都是非常晦涩难以理解的，这两天看`underscore`，遇到闭包，又重新翻了翻书，对闭包有了更深入的理解，整理如下

### 定义

> `Closures` (闭包)是使用被作用域封闭的变量，函数，闭包等执行的一个函数的作用域。通常我们用和其相应的函数来指代这些作用域。(可以访问独立数据的函数)
> 闭包是指这样的作用域，它包含有一个函数，这个函数可以调用被这个作用域所*封闭*的变量、函数或者闭包等内容。通常我们通过闭包所对应的函数来获得对闭包的访问。

以上是`MDN`给出的闭包概念，为了把概念讲全不惜把话说的让人看不懂。[手动斜眼]

简单来说，就是在 `A`函数中有个`B`函数，`B`在`A`函数外执行时依然可以访问A函数的作用域，这时候就形成了一个闭包。

闭包是一个特殊的对象，它由两部分组成，函数和函数的环境。环境由闭包创建时，在作用域中的任何局部变量组成。在我们的例子中, `myFunc` 是一个闭包：

```javascript
function A() {
    var name = "Mozilla";
    function B() {
        alert(name);
    }
    return B;
}

var mydivunc = A();
myFunc();
```

实际上，只要使用了回调函数，实际上就是在使用闭包。

### 循环和闭包

要说明闭包，`for`循环是最常见的例子

```javascript
for (var i = 1; i <= 5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, i * 1000)
}

```

正常情况下，我们对这段代码的预期是分别输出数字`1-5`，每秒一次，每次一个；`

但实际上，这段代码在运行时会以每秒一次的频率输出`5`次 `6`

###  why?

首先解释下`6`是从哪里来的，这个循环的终止条件不是`i<=5`，而是`i=6`。因此，输出显示的是循环结束时i的最终值。

即延迟函数的回调会在循环结束时才执行，而此时`i= 6`；

按照我们的理解，在每个循环中，计时器的回调函数都会捕捉到一个 `i`的副本，但是根据作用域的工作原理，实际情况是尽管循环中的五个函数是在各个迭代中分别定义的，但是它们都被封闭在一个共享的全局作用域中，因此实际上只有一个`i`。

这样的话，所有的函数都共享一个`i`的引用。循环结构让我们误以为背后还有更复杂的机制在起作用，但实际上没有。因此为了实现我们预期的功能，我们需要更多闭包作用域，特别是在循环的过程中每个迭代都需要一个闭包作用域。

`IIFE`会通过声明并立即执行一个函数来创建作用域：

```javascript
for (var i = 1; i <= 5; i++) {
    (function () {
        setTimeout(function timer() {
            console.log(i);
        }, i * 1000);
    })()
}

```

运行后，输出结果还是 `5`个`6` ....

原因是`IIFE`虽然会在每次迭代时，把创建的作用域封闭起来，但如果作用域如果为空的话，仅仅封闭不会起任何作用。

因此我们需要给`IIFE` 一个自己的变量，用来在迭代中保存i的值：

```javascript
for (var i = 1; i <= 5; i++) {
    (function (j) {
        setTimeout(function timer() {
            console.log(j);
        }, j * 1000);
    })(i)
}

```

在迭代内使用`IIFE`会为每个迭代生成一个新的作用域，使得延迟函数的回调可以将新的作用域封闭在每个迭代内部，每个迭代中都会含有一个具有正确值的变量供我们访问。

### 利用块作用域完成迭代计数

我们使用`IIFE`在每次迭代时都创建一个新的作用域，换句话说，每次迭代我们都需要一个块作用域。

因此我们可以使用`let`声明，用来挟持作用域，并在这个块作用域中声明一个变量。

本质上是将一个块转换成了一个可以关闭的作用域。

```javascript
for (let i = 1; i <= 5; i++) {
    setTimeout(function () {
        console.log(i);
    }, i * 1000)
}
```

以上。

#### 参考：
·  你不知道的 javascript (上)
· {% link developer.mozilla.org http://es6.ruanyifeng.com/#docs/promise %}
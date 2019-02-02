---
showDate: true
date: "2015-09-19"
layout: post
tags: ['tech', 'JavaScript', 'ES6']
title: "\u6DF1\u5165 ES6  - \u751F\u6210\u5668"
---

> 原文出自 [ES6 in depths](https://hacks.mozilla.org/2015/05/es6-in-depth-generators/), 作者 [Jason Orendorff](https://blog.mozilla.org/jorendorff/), 翻译：落在深海

*ES6 In Depth* 系列将详细解读 ES6 的新特性。 
  
我很激动，因为今天我们要聊一聊 ES6 里最神奇的特性。  

怎么个神奇法?  对于初学者来说，这个概念跟以往 Js 里其他概念截然不同，以至于初次接触有些晦涩难懂。在某种意义上，它完改变了语言的行为。 如果这都不算神奇，那么？  

<!--more-->

还没完: 这项特性将大大简化代码，并奇迹般将你从“回调地狱”拯救。  

我是不是夸的太过了？好吧，让我们走近科学，是否神奇，最终决定权在你。  

### ES6 generators  

什么是 generators?  

先看下面代码:  

```javascript
function* quips(name) {
    yield "hello " + name + "!";
    yield "i hope you are enjoying the blog posts";
    if (name.startsWith("X")) {
        yield "it's cool how your name starts with X, " + name;
    }
    yield "see you later!";
}
```
    
这段代码来自[a talking cat](//people.mozilla.org/~jorendorff/demos/meow.html), 可能是目前互联网最重要的一类应用了(点链接，跟猫玩玩。如果感到困惑，回来找答案)。  

它看起来像是一个函数。它被称作是 generator 函数，跟函数有很多相似之处。但你可以立马看出它们之间的两个不同:  

+ 普通函数以 `function` 开头，而 generator 函数以 `function*` 开始。

+ generator 函数里， `yield` 语法看起来很像 `return`, 不同之处在于函数(即使是 generator 函数) 也只能 `return` 一次，不过 generator 函数可以 `yield` 许多次。**yield** 表达式将 generator 阻塞，好让其稍候可以继续执行。  

这就是 generator 函数与普通函数最重要的区别。 普通函数无法阻塞自己的执行，而 generator 函数可以。  

### generators 究竟做了什么  

当你调用`quips()` 时，发生了什么？  

```javascript
> var iter = quips("jorendorff");
    [object Generator]
> iter.next()
    { value: "hello jorendorff!", done: false }
> iter.next()
    { value: "i hope you are enjoying the blog posts", done: false }
> iter.next()
    { value: "see you later!", done: false }
> iter.next()
    { value: undefined, done: true }  
```
    
你可能非常习惯于传统函数的调用, 当发生调用时，它们立马开始执行，直到遇见`return` 或者`throw`。这对 Js 程序员来说就像天生直觉一样。  

generator 调用很类似: quips('jorendorff')。但是调用并不会立刻开始。它会返回一个 Generator 对象(上面例子里叫做 iter)。你可以把这个 Generator 对象当做一个函数调用，只是被冻住了。刚好暂停在 generator 函数的第一行代码。  

每次调用 generator 对象的 .next() 方法，函数会将自己解冻并执行，直到遇见下一个 **yield** 表达式。  

这就是为什么每次调用 iter.next(), 我们得到不同的字符串结果。这些结果是 quips 内部 yield 表达式返回的。  

最后一句 iter.next(), 终于触到了 generator 函数的结尾，所以 .done 的结果为*true*。到达函数的结尾类似于返回 `undefined`, 这就是为什么 .value 的值为 undefined 的缘故。  

是时候回去看看 [the talking cat demo page](//people.mozilla.org/~jorendorff/demos/meow.html) 究竟发生了什么。 试着放一个`yield`在循环里，会发生什么？  

在技术上，每次 generator yields, 它会保存堆栈结构 - 局部变量，参数，临时变量，generator 内部执行到的位置，generator 主体被从堆栈中移除。然而 Generator 对象保留了(或者说是拷贝了一份)对该堆栈的引用， 好让后续的 .next() 调用知道如何恢复并执行。  

必须要提的是 **generator 不是线程**。 在有线程概念的语言里，多段代码可同时被执行， 通常会通向未知的，赛跑型的结果，性能提升非常显著。generators 截然不同。一旦被执行，他跑在单线程上。顺序执行，结果也是很明确的，从不会发生并行。并不像系统线程，generator 只能被 其内部的 `yield` 挂起。  

我们知道了什么是 generator, 了解它如何执行、暂停和继续执行。现在提出问题: 如何利用这个奇怪的特性？  

### generator 也是迭代器  

上周，我们了解了 ES6 的迭代器不仅仅是内嵌类，它极易扩展。通过实现```[symbol.iterator]()``` 和```.next()``` 你可以构造自己的迭代器。  

然而实现接口往往需要一大堆工作去做。让我们看看实际应用中实现迭代器需要做哪些事情。实现一个简单的 range 迭代器，用来简单记数，就好像 C 语言里过时的 for( ; ; ) 循环。  

```javascript
// This should "ding" three times
for (var value of range(0, 3)) {
      alert("Ding! at floor #" + value);
}
```
    
下面思路，用到了 ES6 类的概念(不知道没关系，后面我们会讲的)。  

```javascript
class RangeIterator {
    constructor(start, stop) {
        this.value = start;
        this.stop = stop;
    }

    [Symbol.iterator]() { return this; }

    next() {
        var value = this.value;
        if (value < this.stop) {
            this.value++;
            return {done: false, value: value};
        } else {
            return {done: true, value: undefined};
        }
    }
}

// Return a new iterator that counts up from 'start' to 'stop'.
function range(start, stop) {
    return new RangeIterator(start, stop);
}
```
    
像是 `java` 或者 `Swift` 实现迭代器的方式，看上去还行，并不繁琐。上面代码有 bug 么？很难讲。看起来一点不像 for( ; ; ) 循环：迭代器迫使我们拆除循环。  

你可能会觉得迭代器不是那么友好。看起来很好用，一旦实现起来还是很复杂的。  

这时候引入一个复杂难懂的新控制流来让迭代器更易被实现，可能并不太好。然而我们有了 generators, 能用在这里么？  试试看。  

```javascript
function* range(start, stop) {
  for (var i = start; i < stop; i++)
    yield i;
}  
```
    
4行的代码取代了23行复杂的`range`实现(还引入了 RangeIterator 类)。能这么做完全因为 generators 本来就是迭代器。 所有迭代器内部本身就实现了 `.next()` 和 `[Symbol.interator]()` , 你只需写下循环的行为即可。  

没有 generator 来实现的迭代器就好比一封通篇充斥着消极情绪，冗长无比的邮件。永远不直接讲你最想说的，最终又臭又长。`RangeIterator`又臭又长，原因在于用非循环的语法来描述循环，婉转晦涩。generators 才是正确答案。  

那么 generators 在迭代器上还能带给我们什么？  

+ **让所有对象变得可迭代。** 遍历 `this`, yielding 每次的 `value`。然后重写  generator 的 [Symbol.interator] 方法。 

+ **简化类数组的函数。** 假设函数每次被调用返回的是一个数组：  

```javascript
// Divide the one-dimensional array 'icons'
// into arrays of length 'rowLength'.
function splitIntoRows(icons, rowLength) {
    var rows = [];
    for (var i = 0; i < icons.length; i += rowLength) {
        rows.push(icons.slice(i, i + rowLength));
    }
    return rows;
}
```
    
有了 generator, 这样写：  
    
```javascript
 function* splitIntoRows(icons, rowLength) {
      for (var i = 0; i < icons.length; i += rowLength) {
         yield icons.slice(i, i + rowLength);
      }
 }
```
     
不同的是 generator 并不会一次性计算数组每一个元素，而是返回一个迭代器，按需供给每次的返回值。  

+ **不定长的返回值。** 你无法创建一个无限长度的数组。但可以返回 generator  对象生成无尽的序列，且每个调用的对象可以予取予求的获取返回值。 

+ **重构复杂的循环。**你是否有过巨长巨丑的函数？想把它拆成两部分么？generator 是你重构的新利器。当面对复杂的循环， 可以用它把生成数据的部分肢解成为新的 generator 函数，然后改造循环为 for(var data of myNewGenerator(args))。  

+ **用在可迭代对象上。** ES6 并没提供用来做 filtering, mapping，或者修改可迭代数据集的库。然而 generators 更适用于这几种情况，更少的代码，获得更好的效果。  

举个例子，假设你需要类似 Array.prototype.filter 这样的操作 DOM 节点列表的方法，so easy: 

```javascript
function* filter(test, iterable) {
    for (var item of iterable) {
        if (test(item))
          yield item;
    }
}
```
    
generators 很有用对吧？它无疑是实现自定义迭代器最容易的方式了，并且迭代器 ES6 里操作数据和循环一个新的标准。  

还没完，这甚至不是 generator 最重要的能耐。  

### generators 和 异步代码  

下面是我写 Js 代码经常会遇到的：  

```javascript
                        };
                    })
                });
            });
        });
    });  
```
    
或许你也写过类似代码。异步 API 依赖于回调，意味着每次需要写额外的异步函数来处理。所以有段代码要做三件事，结果并不是三行代码，而是三层锯齿状的代码。  

还有一种情形是这样的：  

```javascript
}).on('close', function () {
  done(undefined, undefined);
}).on('error', function (error) {
  done(error);
});  
```
    
异步 API 拥有错误处理约定，而不是简单抛出异常。不同 API 有不同的约定。大多数错误处理会默认被吃掉。其中有些即使是成功执行，也会默认被吃掉。  

直到今天，这些问题成了我们使用异步编程必须付出的代价。必须承认异步代码并不像同步代码那样简单舒服。  

使用 generator 可以拯救我们。  

Q.async() 是一项实验性尝试，它使用 generator 和 promise 来像写同步代码一样组装异步代码：  

```javascript
// Synchronous code to make some noise.
function makeNoise() {
  shake();
  rattle();
  roll();
}

// Asynchronous code to make some noise.
// Returns a Promise object that becomes resolved
// when we're done making noise.
function makeNoise_async() {
  return Q.async(function* () {
    yield shake_async();
    yield rattle_async();
    yield roll_async();
  });
}
```

    
最大的区别在于异步的版本必须在每个异步函数调用时加上`yield`关键字。  

在 Q.async 作用域里写 if 语句或 try catch 块跟在写同步代码一样。而对比写异步代码的其他方式，这很像在学另一门语言。  

想研究至深， James Long’s 的这篇 [very detailed post on this topic](//jlongster.com/A-Study-on-Solving-Callbacks-with-JavaScript-Generators) 可能会对你有所帮助。  

generators 创造了一种新的适合人类思维的异步编程模式。这项工作还在继续发展，至少在努力让语法变得更简单舒服。[ A proposal for async functions](https://github.com/lukehoban/ecmascript-asyncawait), 得到 C# 的特性的启发，使用 promise 跟 generator 构建，在 ES7 的计划范畴内([on the table for ES7](https://github.com/tc39/ecma262))。  

### 什么时候能用上 generator?  

服务端，今天就可以在 io.js(nodejs 里需要使用 --harmony 方式) 使用。  

浏览器端，只有 firefox 27+, chrome 39+ 支持，要让其它浏览器也支持，可以使用 babel 或 traceur 将 ES6 转为 ES5 代码。  

值得了解的故事: Brendan Eich 第一次实现了 Js 的 generators, 他的设计紧紧追随了受 [Icon](//www.cs.arizona.edu/icon/) 语言 影响的 Python generators。并且最早在2006年的火狐浏览器2.0上加上了这个特性。然而标准化的道路却非常坎坷。ES6 的 generators 由编译器黑客 [Andy wingo](//wingolog.org/) 在 Chrome 跟 火狐浏览器上实现。由 Bloomberg 赞助。  

### yield;  

关于 generators 还有很多可以讲。目前为止我们还没有讲到 .throw() 跟 .return() 方法，.next() 的可选参数， 或者 yield* 语法等。我认为这篇文章目前来说已足够长，甚至有点令人厌烦了。我们应该缓一缓。  

下周我们换个话题。 到这篇文章，我们已经探索了 ES6 的两个很深入的话题。是时候来点轻松惬意但非常实用的东西了。ES6 有不少此类特性。  

接下来：很容易融入日常使用的新特性。加入探索 ES6 template strings 之旅。

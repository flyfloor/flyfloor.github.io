---
date: "2015-09-20"
layout: post
tags: ['tech', 'JavaScript', 'ES6']
title: "\u6DF1\u5165 ES6  - \u7BAD\u5934\u51FD\u6570"
---

> 原文出自 [ES6 in depths](https://hacks.mozilla.org/2015/06/es6-in-depth-arrow-functions/), 作者 [Jason Orendorff](https://blog.mozilla.org/jorendorff/), 翻译：落在深海

*ES6 In Depth* 系列将详细解读 ES6 的新特性。 

箭头在 Javascript 早期便已存在。第一个 Javascript 指南建议将 Javascript 代码用内联 scripts 标签包裹，写在 HTML 注释里。这样对于不支持 Js 的浏览器，Javascript 代码就不会被认作是 text 了：  

```html
<script language="javascript">
<!--
  document.bgColor = "brown";  // red
// -->
</script>
```

<!-- more -->
    
老浏览器将看到的是两个不被支持的 tag 和注释；只有支持的浏览器才能执行 Js 代码。  

为了支持这项老的把戏，浏览器的 Javascript 引擎会将 ` <!-- ` 开头当做单行注释。这项技术一直存在于 Javascript 语言中，直到今天，不仅仅在内联的 `<script>` 标签里。甚至在 NodeJs 里也是如此。  

因此，[这样带箭头的注释方式在 ES6 里第一次被标准化](//people.mozilla.org/~jorendorff/es6-draft.html#sec-html-like-comments)。然而今天要讲的，此箭头非彼箭头。  

箭头 --> 同时表示 单行注释。诡异的是，HTML 标签里 --> 之前的部分是注释，而在 Js 里，箭头后面的单行部分是注释。  

更奇怪的是，当且仅当箭头在行首时才算是注释。假如出现在别的地方， --> 是 Js 的运算符，表示“直到” 运算符！  

```javascript
function countdown(n) {
  while (n --> 0)  // "n goes to zero"
    alert(n);
  blastoff();
}  
```
    
[这段代码确实能运行](//codepen.io/anon/pen/oXZaBY?editors=001)。循环从 n 到 0. 这并不是 ES6 的新特性， 熟悉的特性，带有一点误导性质。你能觉察究竟发生了什么？通常的，[stackoverflow 都能找到答案](//stackoverflow.com/questions/1642028/what-is-the-name-of-the-operator)。  

确实存在 小于或等于运算符，<=。你可能在代码里找到更多的箭头，然而停下来观察，会发现有个箭头没有用到。  

* <!-- `single-line comment`  
*  --> `“goes to” operator`    
* <=  `less than or equal to`  
* =>	???  

我们先来看看函数。    

### 到处都存在函数表达式  

在你需要函数时，你可以在任何运行的代码中写下该函数体。  

例如，你要告诉浏览器，当用户点击某个按钮做某件事，代码如下：  

```javascript
$("#confetti-btn").click(  
```
    
jQuery 的 .click() 方法有一个参数：一个函数。毫无疑问，你可以这样写：  

```javascript
$("#confetti-btn").click(function (event) {
  playTrumpet();
  fireConfettiCannon();
});  
```
    
这段代码太稀松平常了。所以在 Javascript 流行前，如果再别的地方这样写是很奇怪的，毕竟其他语言并不支持该特性。当然1958年， Lisp 拥有了类似的函数表达式，也称为 lambda 函数。然而类似 C++，Python， c# 等语言里 许多年都没有这样的语法。  

现在这四种语言都有了 lambda。新语言通常都会内建 lambda 语法。 Javascript 应该感谢 -- 那些构建库时毫无畏惧的大量使用 lambda 的早期 Js 程序员，是他们让这项特性风靡全球。  

稍显遗憾的，对比上述语言，Js 的 lambda 是最啰嗦的。 

```vim
// A very simple function in six languages.
function (a) { return a > 0; } // JS
[](int a) { return a > 0; }  // C++
(lambda (a) (> a 0))  ;; Lisp
lambda a: a > 0  # Python
a => a > 0  // C#
a -> a > 0  // Java  
```
    
### 箭袋里的“新箭”  

ES6 提供了编写函数的新语法。  

```javascript
// ES5
var selected = allJobs.filter(function (job) {
  return job.isSelected();
});

// ES6
var selected = allJobs.filter(job => job.isSelected());  
```
    
当你需要单个参数的函数时，arrow function 的语法是 `Identifier => Expression`。这样就省去写 function, return, 括号，花括号及分号。  

(我个人对此特性心存感激。对我来说，不用输入 function 非常重要，因为我经常输入`functoin`，而且需要不断地去纠正它。)  

含有多个参数的函数(不含任何参数，或者可变参数及默认参数，或者 destructuring 参数)你只需要将变量们用括号括起来。  

```javascript
// ES5
var total = values.reduce(function (a, b) {
  return a + b;
}, 0);

// ES6
var total = values.reduce((a, b) => a + b, 0);  
```

我认为它看起来很漂亮。  

arrow functions 与其他库的函数工具搭配的也非常好，例如 [Underscore.js](//underscorejs.org/) 和 [Immutable](https://facebook.github.io/immutable-js/)。事实上， [Immutable’s documentation](https://facebook.github.io/immutable-js/docs/#/) 的例子完全是用 ES6 写的， 很多函数都是用 arrow function 写的。  

那些非函数式的设置呢？arrow function 可以包含一段表达式代码块，而不仅仅是一句表达式。翻回前面的例子：  

```javascript
// ES5
$("#confetti-btn").click(function (event) {
  playTrumpet();
  fireConfettiCannon();
});  
```
    
ES6 里这么写：  

```javascript
// ES6
$("#confetti-btn").click(event => {
  playTrumpet();
  fireConfettiCannon();
});  
```
    
另外，使用 [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 变得更戏剧性， `}).then(function (result) {` 这行代码也可以省掉。  

注意到包含代码块的 arrow function 并没有显式的返回值。为此需要使用 return 表达式。  

当使用 arrow function 创建 对象字面量，切记要用圆括号将对象括起来，不然会出现 bug：  

```javascript
// create a new empty object for each puppy to play with
var chewToys = puppies.map(puppy => {});   // BUG!
var chewToys = puppies.map(puppy => ({})); // ok  
```
    
为什么呢？ 原因在于，不幸的是空对象 {} 跟 空的代码块 {} 看起来完全一样。ES6 将箭头紧跟遇到的 { 的代码当做代码块处理。 所以 `puppy => {}` 被翻译为未作任何操作的箭头函数，而最终返回 undefined。  

更易获得是，包含 {key: value} 的对象看起来像是包含了标签语句的代码块 -- 起码你的 Javascript 引擎是这么认为的。 幸运的是 { 是唯一会产生疑惑的字符，那么用圆括号将对象字面量包裹起来是唯一你需要记住的把戏。  

### 别忘了`This`  

function 跟 arrow function 有轻微不同。**Arrow functions 并没有 this**。如果在内部获取 this，得到的永远是其外部作用域的 this。  

在探索为何如此之前，让我们翻回去看看。  

Javascript 里 this 如何工作？它的值从哪里来？ [三言两语很难解释清楚](//stackoverflow.com/questions/3127429/how-does-the-this-keyword-work)。如果对你来说很容易，是因为你接触它太长时间。  

其中一个原因是 function 函数会自动返回 this 值， 不管你需要或否。你是否写过类似下面的把戏：  

```javascript
{
  ...
  addAll: function addAll(pieces) {
    var self = this;
    _.each(pieces, function (piece) {
      self.add(piece);
    });
  },
  ...
}  
```
    
在内部的函数，其实我们仅仅需要 this.add(piece)。不幸的是， 内部函数并没有继承外部的 this 值。所以内部的 this 值 将会是 window 或者 undefined。而 self 这个临时变量作用就是把外部的 this 值 传递到内部函数里(另一种方式是通过 .bind(this) 将 this 绑定到内部函数上，两种方式都不那么美观)。  

ES6里，遵循以下规则，那么有关 this 的小把戏基本可以抛弃了：  

* 使用非箭头函数，那么它会调用 object.method()，调用者将会得到一个有意义的 this 值。  

* 剩下的场景都用箭头函数。  

```javascript
// ES6
{
  ...
  addAll: function(pieces) {
    _.each(pieces, piece => this.add(piece));
  },
  ...
}  
```
        
在 ES6 里，注意到 addAll 方法会从调用者接收 this 值。而内部函数是箭头函数，自然地它会从 addAll 函数继承 this 的值。  
  
作为奖励，ES6 为对象字面量提供了另一种简写！上述代码可以改写为：  

```javascript
// ES6 with method syntax
{
  ...
  addAll(pieces) {
    _.each(pieces, piece => this.add(piece));
  },
  ...
}  
```
    
在方法跟箭头中间，我可能再也不需要输入 function 了，非常有趣。  

箭头函数跟非箭头函数还有轻微的不同：箭头函数并没有 arguments。 当然在 ES6 里， 你可以用可变参数或默认参数替代。  

### 用“箭头”戳破计算机科学黑暗的心  

关于箭头函数，我们已探讨了许多实用的技术。还有一种黑科技我想跟你聊聊：揭开计算机内心深处的神秘面纱。实用与否，你自行判断。  

1936年，Alonzo Church 和 Alan Turing 独立开发了非常强大的计算机数学模型。图灵称它为 a-machines, 然而其他人称他为图灵机。Church 写了取代函数，并将它命名为 [λ-calculus](https://en.wikipedia.org/wiki/Lambda_calculus)。(λ 是希腊字母 lambda 的小写。) 这也是为什么 Lisp 用 LAMBDA 来表示函数，也是今天 lambda 的由来。  

然而，什么是 λ-calculus？  计算模型又是什么？  

几句话很难解释的清楚， 我尝试这样解释：λ-calculus 是最早期的编程语言之一。存储计算机盛行了一二十年，起初它根本不是被设计成编程语言的，而是简单的，定制的纯数学语言，它包含了你需要的所有计算功能。Church 希望能够通过模型强大的运算能力证明一些东西。    

他发现系统只需要一种东西： 函数。  

多么非凡的想法啊。抛弃对象，数组，if 语句，while 循环，分号，赋值，逻辑运算符，甚至是循环，依然可以造出 Javascript 可以做的计算，仅仅需要函数。  

下面是数学家用 Church 的 λ 符号可能写出的程序：  

```vim
fix = λf.(λx.f(λv.x(x)(v)))(λx.f(λv.x(x)(v)))  
```
    
等价于 Javascript 里的：  

```javascript
var fix = f => (x => f(v => x(x)(v)))
               (x => f(v => x(x)(v)));  
```
                   
Javascript 实际上包含了 λ 微积分的实现， 也就是说 Javascript 拥有 λ-calculus。  

Alonzo Church 和后续研究者对于 λ-calculus 的研究，及将它无声息的融入每一个主流的编程语言的故事，远远超越了本篇文章的范围。如果你对计算机科学基础有浓厚兴趣，或者想看一门语言只用函数来实现类似循环或递归的话，强烈建议你去看看 [Church numerals](https://en.wikipedia.org/wiki/Church_encoding) 和 [fixed-point combinators](https://en.wikipedia.org/wiki/Fixed-point_combinator#Strict_fixed_point_combinator)，并且试着在 firefox console 或者 scratchpad 调试看看。拥有 ES6 箭头函数的 Javascript，称得上探索 λ-calculus 的最佳语言。  

### 什么时候能用？  

2013年，我在 firefox 里实现了 ES6 箭头函数。Jan de Mooij 优化了它。也感谢 Tooru Fujisawa 和 ziyunfei 打的补丁。  

箭头函数同样在 Microsoft Edge 预览版里实现。如果立马想用的话，可以使用 Babel, Traceur 和 TypeScript。  

下篇文章关于 ES6 的一个比较奇特的特性。届时将看到 typeof x 返回焕然一新的值。我们将会提问：什么时候名字不是字符串？我们会对等价的意义产生疑惑。这会非常的诡异。欢迎下周继续加入我们。  
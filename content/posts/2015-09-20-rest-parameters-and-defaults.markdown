---
date: "2015-09-20"
layout: post
tags: ['tech', 'JavaScript', 'ES6']
title: "\u6DF1\u5165 ES6  - \u5176\u4ED6\u53C2\u6570\u53CA\u9ED8\u8BA4\u53C2\u6570"
---

> 原文出自 [ES6 in depths](https://hacks.mozilla.org/2015/05/es6-in-depth-rest-parameters-and-defaults/), 作者 [Jason Orendorff](https://blog.mozilla.org/jorendorff/), 翻译：落在深海

*ES6 In Depth* 系列将详细解读 ES6 的新特性。 

本文主讲的两个特性：其他参数和默认参数，将使得 Javascript 更富有表达性。
  
### 其他参数  

在创建 API 时常会需要可变函数，该函数接受任意长度的参数。例如：String.prototype.concat 接受多个字符串参数。ES6 提供一种新的方式来传递可变参数。  

我们写个简单的函数 `containsAll`，用来检查字符串是否包含一系列的子字符串。例如： containsAll("banana", "b", "nan") 会返回 true， 而 containsAll("banana", "c", "nan") 返回  false。  

一般我们是这样写的：  

<!--more-->

```javascript
function containsAll(haystack) {
  for (var i = 1; i < arguments.length; i++) {
    var needle = arguments[i];
    if (haystack.indexOf(needle) === -1) {
      return false;
    }
  }
  return true;
}  
```
    
这种实现利用了 arguments， 它包含了函数接收的所有参数，类似数组的对象。然而这段代码可读性并不理想。首先，函数的参数列表仅仅有 `haystack` 一个参数，所以第一眼看并不像可以传递多个参数。另外，对参数操作时也必须从 index 为1，而不是0来迭代，因为第一个参数是 haystack。其他参数解决了这些顾虑。下面是 ES6 实现的 `containsAll`：  

```javascript
function containsAll(haystack, ...needles) {
  for (var needle of needles) {
    if (haystack.indexOf(needle) === -1) {
      return false;
    }
  }
  return true;
} 
```
    
这个版本引入了 ...needles 的语法。让我们看看当执行 `containsAll("banana", "b", "nan")` 它是如何被调用的。haystack 被 `banana` 填充，needles 前跟的省略号暗示这部分属于其他参数。 其他参数被塞到 needles 这个数组里。这里的值为 `["b", "nan"]`。函数调用其他一切正常(注意到这里用到了 ES6 的 for-of 循环)。  
 
只有最后一个参数可以作为其他参数，而前面的参数会被当做普通参数赋值。假如没有指定其他参数，那么其他参数为空数组，而不是 undefined。  
 
### 默认参数  

通常不是所有参数都需要调用时赋值，一些可能会有默认值。Javascript 没被赋值的参数默认值为 undefined。ES6  提供一种方式来指定任意的参数默认值。  

例子：  

```javascript
function animalSentence(animals2="tigers", animals3="bears") {
    return `Lions and ${animals2} and ${animals3}! Oh my!`;
}  
```
    
针对每个参数， `=` 表示如果调用者未对参数赋值时的默认值。所以，`animalSentence()` 会返回 `"Lions and tigers and bears! Oh my!"`, `animalSentence("elephants")` 返回 `"Lions and elephants and bears! Oh my!"`, 而 `animalSentence("elephants", "whales")` 返回 `"Lions and elephants and whales! Oh my!"`。  

关于默认参数有下面几个细节：  

* 并不像 python，**默认值表达式是在函数被调用时从左到右执行**。这同时表明该表达式中，可以使用左边带默认值的变量来赋值。例如：  

```javascript
function animalSentenceFancy(animals2="tigers",
    animals3=(animals2 == "bears") ? "sealions" : "bears")
{
  return `Lions and ${animals2} and ${animals3}! Oh my!`;
}
```

`animalSentenceFancy("bears")` 返回 `"Lions and bears and sealions. Oh my!"`。  
 
* 显式的传 undefined 的参数等同于未传递任何值。这样，animalSentence(undefined, "unicorns") 返回为 "Lions and tigers and unicorns! Oh my!"。  
 
* 无默认值的参数，默认值则为 undefined，所以：  

```javascript
function myFunc(a=42, b) {...}  
``` 

等同于 ： 

```javascript     
function myFunc(a=42, b=undefined) {...}  
```
                 
### 放弃使用 arguments  

我们已经看到可变参数及默认参数能够取代 arguments 的使用，使代码变得更易读。除了生涩难懂，arguments 最臭名昭著的还是 [headaches for optimizing JavaScript VMs](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers#3-managing-arguments)。  

可变参数及默认参数有望完全替代 arguments。首先函数如果使用可变参数或者默认参数，则被禁止使用 arguments，即使 arguments 短期不被移除，但还是推荐使用可变参数及默认参数。  

### 浏览器支持  

firefox 15+ 支持。  

遗憾的是，其他浏览器均不支持此特性。[V8 最近添加了支持](https://code.google.com/p/v8/issues/detail?id=2159) , 且 [这里](https://code.google.com/p/v8/issues/detail?id=2160) 有关于默认参数实现的 issue。JSC 同样也有关于 [可变参数](https://bugs.webkit.org/show_bug.cgi?id=38408) 及 [默认参数](https://bugs.webkit.org/show_bug.cgi?id=38409) 的 issues。  

Babel 和 Traceur 编译器都支持了默认参数， 可以今天就开始使用。  

### 总结  

尽管技术上来说并不允许新的方式，可变参数及默认参数使得 Javascript 函数更易于表达、可读。喜闻乐见！  

> 备注：感谢 Benjamin Peterson 在 firefox 实现此特性， 同样感谢他对该项目的贡献，以及本篇文章。  

下周将介绍 ES6 里一个简单、优雅、 实用，每天都会用的特性。语法类似数组或对象，只不过需要反过来用，构造一种简明的方式来拆解数组跟对象， 什么意思？ 为什么拆解对象？欢迎下周继续。  

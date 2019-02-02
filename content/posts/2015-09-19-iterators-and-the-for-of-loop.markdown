---
showDate: true
date: "2015-09-19"
layout: post
tags: ['tech', 'JavaScript', 'ES6']
title: "\u6DF1\u5165 ES6  - \u8FED\u4EE3\u5668\u548C for-of \u5FAA\u73AF  "
---

> 原文出自 [ES6 in depths](https://hacks.mozilla.org/2015/04/es6-in-depth-iterators-and-the-for-of-loop/), 作者 [Jason Orendorff](https://blog.mozilla.org/jorendorff/), 翻译：落在深海

*ES6 In Depth* 系列将详细解读 ES6 的新特性。

怎么循环一个数组的元素呢？20年前，Js 代码是这么写的: 

```javascript
for (var index = 0; index < myArray.length; index++) {
    console.log(myArray[index]);
}  
```
    
从 ES5 起，你可以这样写：  

```javascript
myArray.forEach(function (value) {
    console.log(value);
});
``` 
     
<!--more-->     

看起来短些，但有副作用: 无法使用 ``` break``` 或者```return```从包裹的函数中返回。  
  
倘若能使用 ```for-``` 循环语法完成对数组元素操作岂不是更好。 
 
那么试试```for-in``` ?  

```javascript
for (var index in myArray) {    // don't actually do this
    console.log(myArray[index]);
}  
``` 
     
这不是一个好选择:  
 
+ 赋值给 *index* 的实际上得到的是```"0", "1", "2"``` 等等， 并不是数字。你一定不想得到这样的字符串运算("2" + 1 == "21"), 一点也不方便。  

+ 循环体不仅对数组对象本身循环， 而且会将赋值操作后的属性也进行循环，例如，数组 myArray有个枚举属性 myArray.name, 那么循环将多循环一次，而那次的索引 index == "name"。甚至是数组 prototype 继承链上的属性都会被访问到。  

+ 更令人惊讶的是， 在某些情况下，数组内的循环可能不会按顺序执行。  

简单来说，for-in 主要作用于 plain old objects 的字符串 keys。对于数组，并不奏效。  

###强大的 for-of 循环  

还记得上周我承诺 ES6 并不会破坏原先你写的 Js 代码吧。数以百万的网站有用到 for-in 循环， 甚至是用在数组的循环上。所以去修 for-in 循环，让其适用于 arrays 是不现实的。对于 ES6，改进的唯一办法就是引入一种新的循环语法。  

tada:  

```javascript
for (var value of myArray) {
     console.log(value);
}  
```
    
额，似乎并没有很酷炫，对吧？ 后面我们会看看这葫芦里卖的什么药，目前只需知道:  

+ for-of 是迄今为止对数组循环最简洁的方式。  
+ 它避开了 for-in 所有的坑。  
+ 与 forEach 不同的是，可以使用 break, continue 和 return。  

for-in 循环用来操作对象的属性。  
for-of 用来对数组循环，但不止于此。  

### 其他支持 for-of 的集合  

for-of 不仅仅支持 Array，同样适用于类似 Array 的数据，比如 DOM NodeLists。  
同时支持字符串的循环，例如：  

```javascript
for (var chr of "😺😲") {
  alert(chr);
}  
```
    
支持 Map 跟 Set 对象。 

抱歉，你没听过 Map 和 Set？这可是 ES6 引入的新类型，我们会在后续用一整章来讲。假如你在别的语言里已用过这些东西的话，你不会感到惊讶。  

举个例子， Set 适合用来去重。  

```javascript
// make a set from an array of words
var uniqueWords = new Set(words);  
```
    
一旦创建一个 Set， 你很容易对它进行循环。  

```javascript
for (var word of uniqueWords) {
  console.log(word);
}  
```
    
Map 有些许不同：内部由键值对构成，可按照下面方式遍历：  

```javascript
for (var [key, value] of phoneBookMap) {
  console.log(key + "'s phone number is: " + value);
}  
```
    
以上方式叫做 Destructuring， 是 ES6 的新特性，后面会重点介绍。  

到此你可以看到这样景象：Js 增添越来越多的类型，而你将频繁的用 for-of 循环操作它们。  

遗憾的是 for-of 循环不支持 plain old objects， 可以通过 for-in 或 object.keys() 补充。  

```javascript
// dump an object's own enumerable properties to the console
for (var key of Object.keys(someObject)) {
  console.log(key + ": " + someObject[key]);
}
```
    
### 帽檐之下    

*“Good artists copy, great artists steal.” —Pablo Picasso*  

ES6 的特性多数并不是空穴来风，它们早已在其他语言里得到验证。  

for-of 循环， 类似于 C++, Java, C#, Python 里的循环，适用于语言本身及库提供的多种不同数据结构，同时也是语言的扩展点之一。  

与 forEach 相同，for-of 完全适用于方法调用。Arrays，Maps，Sets 以及我们提到的其他类型，它们的共同点是都拥有迭代器方法。  

什么对象可以支持迭代器？答案是任何你想要的对象。

好比你只需 myObject.toString()， Js 就知道如何将对象转换成字符串。循环亦如此，```myObject[Symbol.iterator]()``` 这样 Js 便知道如何操作对象循环。  

举个例子，假设使用 JQuery，你非常喜欢用 .each(), 那么想要 for-of 支持 JQuery 对象，应该这样写：  

```javascript
// Since jQuery objects are array-like,
// give them the same iterator method Arrays have
jQuery.prototype[Symbol.iterator] =
Array.prototype[Symbol.iterator];  
```
    
好吧，我知道你在想什么，[Symbol.iterator] 这样写太奇葩了。委员会本来决定只用 .iterator()，然而现有的代码可能有些对象本身有用到 .iterator()，所以那帮人最终决定用 symbol,  而不是挂在 string 下。  

Symbols 是 ES6 的新特新之一，后续文章会细讲。你现在只需知道的是，用 Symbol.iterator，可以保证以前的代码不会有任何冲突。这样说吧，获得更好的特性和兼容性，代价仅仅是语法看起来有点怪而已。  

我们把包含 [Symbol.iterator] 方法的对象称为可迭代的。接下来会看到的是，可迭代的对象遍布整个语言，不仅是 for-of，Map， Set，destructuring 赋值，还有新的 spread 运算符。  

### 迭代对象  

现在你压根不需要刀耕火种去实现迭代器，下周告诉你具体原因。但为了阅读完整性，让我们瞧瞧究竟(错过这部分，你会后悔的)。  

for-of 循环在集合上调用 ```[Symbol.iterator]()```，会得到一个迭代器的对象，它可以是包含 .next() 方法的任意对象； for-of 则会循环调用该方法。下面是最简单的迭代器对象:  

```javascript
var zeroesForeverIterator = {
    [Symbol.iterator]: function () {
        return this;
    },
    next: function () {
        return {done: false, value: 0};
    }
};  
```
    
每次调用  .next() 得到同样的返回值，告诉循环体 a) 循环还没结束。 b) next 的 value 为 0。也就是说这是个死循环，当然典型的循环不长这样。  

拥有 .done 跟 .value 的 Js 迭代器， 表面看上去与其他语言不同。Java 里是 .hasNext() 跟 .next()。Python 里只有 .next()，当值不存在时，会抛出 StopIteration 异常。但三种方式基本都返回同样的信息。  

迭代器对象可实现 optional 的  .return()，.throw(exc) 方法。当循环因为异常或遇到*break* 及*return*声明而提前退出时 .return() 会被调用。所以我们可以在 .return() 里做一些清理或者释放资源的工作。大多数迭代对象并不需要自己实现该方法。.throw(exc) 更是个特殊情况: for-of 循环永远不会调用它，这个会在下周细讲。  

既然知道了细节，我们可以从简单 for-of 循环入手，用这些基础方法来重写它。  

首先 for-of 循环：  

```javascript
for (VAR of ITERABLE) {
  STATEMENTS
}
```
    
这个粗糙的版本，用了些基础方法及临时变量:  

```javascript
var $iterator = ITERABLE[Symbol.iterator]();
var $result = $iterator.next();
while (!$result.done) {
  VAR = $result.value;
  STATEMENTS
  $result = $iterator.next();
}
```

上面代码里并没有写 .return() 是如何被调用的。 我可以添加这部分，但我还是倾向于暂时略过这部分。总之， for-of 极容易使用，然而背后却别有洞天。  

### 好了，快告诉我什么时候能用上？  

火狐目前已完全支持，chrome 下需要打开chrome://flags， 点击 “Experimental JavaScript”， 微软家的 Spartan 浏览器也支持，不出所料 IE 并不支持，如果要 IE 及 Safari 下也工作,  你可以使用 Babel 或者  Google 的 Traceur 等翻译工具将 ES6 代码翻译成 ES5代码。  

服务端可以直接使用 - 你可以在 io.js 里直接用 for-of 循环(或者 nodejs,  --harmony 方式)。  

{ done: true }

好啦！今天就到此为止，关于 for-of 的介绍还没有结束。  

ES6 还有一种新对象类型，非常好的适应 for-of，只字未提的原因是它是下周的主题。如果在别的语言里还未遇到过，你可能会彻底震惊。然而这是写迭代器最容易的做法，非常适合重构代码，而且将完全改变写异步调用代码的方式，无论是在服务端还是浏览器端。所以请务必持续关注。
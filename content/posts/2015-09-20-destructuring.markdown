---
date: "2015-09-20"
layout: post
tags: ['tech', 'JavaScript', 'ES6']
title: "\u6DF1\u5165 ES6  - \u89E3\u6784"
---

> 原文出自 [ES6 in depths](https://hacks.mozilla.org/2015/05/es6-in-depth-destructuring/), 作者 [Nick Fitzgerald](//fitzgeraldnick.com/), 翻译：落在深海

*ES6 In Depth* 系列将详细解读 ES6 的新特性。

编者的话：本文的早期版本，作者为 firefox 开发者工具的工程师 [Nick Fitzgerald](//fitzgeraldnick.com/)，出现在 Nick 的博客 [Destructuring Assignment in ES6](//fitzgeraldnick.com/weblog/50/)。

### 什么是 Destructuring？  

<!-- more -->

destructuring 赋值允许用类似数组对象迭代的语法给数组或对象的属性赋值。语法极致简洁，而且较原有属性赋值语法更清晰。  

在没有 destructuring 赋值之前，这样获取数组的属性：  

```javascript
var first = someArray[0];
var second = someArray[1];
var third = someArray[2];  
```
    
有了 destructuring 赋值，上面代码变得清晰可读：  
    
```javascript
var [first, second, third] = someArray;  
```
    
SpiderMonkey(firefox 的 Javascript 引擎)早已支持大多数的 destructuring 操作，然而并不是全部支持。[Track SpiderMonkey’s destructuring (and general ES6) support in bug 694100](https://bugzilla.mozilla.org/show_bug.cgi?id=694100)。  

### Destructuring 数组和迭代器  

数组的 destructuring 赋值通常语法类似：  

```javascript
[ variable1, variable2, ..., variableN ] = array;  
```
    
这会根据对应数组的元素对 variable1 到 variableN 进行赋值。你可以同时使用 `var`, `let` 或者 `const` 定义这些变量。  

```javascript
var [ variable1, variable2, ..., variableN ] = array;
let [ variable1, variable2, ..., variableN ] = array;
const [ variable1, variable2, ..., variableN ] = array;  
```
    
事实上说变量是不恰当的，原因在于赋值可以支持 n 层嵌套，只要你想。  

```javascript
var [foo, [[bar], baz]] = [1, [[2], 3]];
console.log(foo);
// 1
console.log(bar);
// 2
console.log(baz);
// 3  
```
    
此外，你可以跳过数组的某些元素：  
    
```javascript
var [,,third] = ["foo", "bar", "baz"];
console.log(third);
// "baz"  
```

也可以使用其他参数：  

```javascript
var [head, ...tail] = [1, 2, 3, 4];
console.log(tail);
// [2, 3, 4]  
```
    
当你获取数组元素超过边界，或者遇到不存在的元素时，你都将得到 `undefined`。  

```javascript
console.log([][0]);
// undefined

var [missing] = [];
console.log(missing);
// undefined  
```
    
destructuring 也支持迭代器操作：  

```javascript
function* fibs() {
  var a = 0;
  var b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

var [first, second, third, fourth, fifth, sixth] = fibs();
console.log(sixth);
// 5  
```
### Destructuring 对象  

Destructuring 赋值能够绑定变量到对象的不同属性上。你需要指定需要绑定的对象属性：  

```javascript
var robotA = { name: "Bender" };
var robotB = { name: "Flexo" };

var { name: nameA } = robotA;
var { name: nameB } = robotB;

console.log(nameA);
// "Bender"
console.log(nameB);
// "Flexo"  
```
    
当变量名跟对象属性名称相同时，可以简写为：  

```javascript
var { foo, bar } = { foo: "lorem", bar: "ipsum" };
console.log(foo);
// "lorem"
console.log(bar);
// "ipsum"  
```
    
像 destructuring 数组一样，支持嵌套赋值：  

```javascript
var complicatedObj = {
  arrayProp: [
    "Zapp",
    { second: "Brannigan" }
  ]
};

var { arrayProp: [first, { second }] } = complicatedObj;

console.log(first);
// "Zapp"
console.log(second);
// "Brannigan"  
```

同样，对未定义的属性，将得到 undefined：  

```javascript
var { missing } = {};
console.log(missing);
// undefined  
```
    
你需要注意的一个潜在 bug 是，当在对象上使用 destructuring 赋值时，如果未声明变量(未使用 let, const 或者 var)，你将得到一个错误：  

```javascript
{ blowUp } = { blowUp: 10 };
// Syntax error  
```
    
报错的原因在于 Javascript 告诉引擎把以 `{` 开始的语句当做 block 语句解析(例如 { console } 就是个 block 语句)。解决办法为用括号包裹起来：  

```javascript
({ safe } = {});
// No errors  
```
    
### Destructuring 值并不是对象、数组或者迭代器  

当你对 `null` 或者 `undefined` 做 destructuring 赋值时，得到的是类型错误：  

```
var {blowUp} = null;
// TypeError: null has no properties  
```
    
然而你可以对其他类型进行赋值，例如 booleans, numbers, strings, 得到的都是 undefined: 

```javascript
var {wtf} = NaN;
console.log(wtf);
// undefined  
```
    
可能有些意外，然而原因非常简单。当对象赋值时，被 destructured 的值要求必须是可被转换成对象的。大多数类型可以转换成对象，而 null 和 undefined 不能被转换。[拥有迭代器](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-getiterator) 的值才能进行元素赋值。  

### 默认值  

当然可以给赋值时未定义的属性提供默认值：   

```javascript
var [missing = true] = [];
console.log(missing);
// true

var { message: msg = "Something went wrong" } = {};
console.log(msg);
// "Something went wrong"

var { x = 3 } = {};
console.log(x);
// 3  
```
    
*(编辑的话：目前此特性已被 firefox 支持，但仅限于前两种用法，[见这里](https://bugzilla.mozilla.org/show_bug.cgi?id=932080))*  

### destructuring，在应用中的使用场景  

#### 函数参数定义  

作为开发者，我们可以提供更友好的 API，只需要传递包含多个属性的单个对象参数，而不是让 API 使用者传入 n 个独立的参数。现在可以使用 destructuring 来避免对单个对象参数的重复赋值：  

```javascript
function removeBreakpoint({ url, line, column }) {
  // ...
}  
```

以上代码是 firefox devtools 的一段代码(这工具也是 Javascript 实现的 :p )。这样写实在是太舒服了。  

#### 配置对象参数  

对上述例子延伸，同样可以给对象的属性提供默认值。对象用来提供配置信息，而它的部分属性存在缺省配置时，这样做非常有用。举个例子，jQuery 的 ajax 方法的第二个参数，可以被重写成：  

```javascript
jQuery.ajax = function (url, {
  async = true,
  beforeSend = noop,
  cache = true,
  complete = noop,
  crossDomain = false,
  global = true,
  // ... more config
}) {
  // ... do stuff
};
```  
    
这样就避免了类似` var foo = config.foo || theDefaultFoo; ` 的重复赋值。  

*(编辑的话：不幸的是，此特性在 firefox 暂未被实现。在本篇之前，我们对此做了很多工作，而且[仍在更新](https://bugzilla.mozilla.org/show_bug.cgi?id=932080))*  

#### ES6 迭代协议  

在这之前提到过，[ES6 定义了迭代协议](https://hacks.mozilla.org/2015/04/es6-in-depth-iterators-and-the-for-of-loop/)。当对 Maps (ES6 标准库的补充) 迭代时，得到的是[key, value] 的键值对。利用 destructure 可以轻松得到 key 跟 value：  

```javascript
var map = new Map();
map.set(window, "the global");
map.set(document, "the document");

for (var [key, value] of map) {
  console.log(key + " is " + value);
}
// "[object Window] is the global"
// "[object HTMLDocument] is the document"  
```

仅对 key 迭代:  

```javascript
for (var [key] of map) {
  // ...
}  
```
    
或者对 value 迭代：  

```javascript
for (var [,value] of map) {
  // ...
}  
```
    
#### 多返回值  

尽管多返回值没有被语言本身支持，或许并不需要因为你可以返回数组并对其做 destucture 赋值：  

```javascript
function returnMultipleValues() {
  return [1, 2];
}
var [foo, bar] = returnMultipleValues();  
```
    
或者，可以用对象做为多返回值的容器：  

```javascript
function returnMultipleValues() {
  return {
    foo: 1,
    bar: 2
  };
}
var { foo, bar } = returnMultipleValues();  
```
    
这两种方式都要比临时容器要好：  

```javascript
function returnMultipleValues() {
  return {
    foo: 1,
    bar: 2
  };
}
var temp = returnMultipleValues();
var foo = temp.foo;
var bar = temp.bar;  
```
    
或者使用继续传递的方式：  

```javascript
function returnMultipleValues(k) {
  k(1, 2);
}
returnMultipleValues((foo, bar) => ...);  
```
    
#### 引入 CommonJS 模块  

还没用 ES6 的模块？ 还在用 CommonJS 模块？没问题！  当使用 CommonJs 模块 X， 通常会得到该模块的所有方法，然而许多并不是你需要的。通过 destructuring， 你可以显式的引入你想要的模块方法，而不被别的方法污染作用域：  

```javascript
const { SourceMapConsumer, SourceNode } = require("source-map");  
```
    
(如果你已经用到 ES6 的模块，import 语法已经这么定义了。)  

### 总结  

正如你所看到的，destructuring 在许多小的场景下很实用。在 Mozilla 我们用它做了许多实验。Lars Hansen 十年前就在 Opera 浏览器里提出了 Js destructuring 的概念，而 Brendan Eich 稍晚些在 firefox 里也提供了支持，并在 firefox 2 里正式推出。destructuring 悄悄潜入了我们的日常，让我们的代码更短、更清晰。  

5周前，我提到 ES6 将改变写 Javascript 的方式。类似 destructuring 这种特性：简单的提升，不需要太多学习成本，积少成多，逐渐影响我们写的每个项目。进化最终造就革命。  

在 ES6 里 destructuring 特性的持续更新可以说是团队功劳。特别感谢 Tooru Fujisawa (arai) 和 Arpad Borsos (Swatinem) 的杰出贡献。  

destructuring 目前正在 Chrome 下开发，而其他浏览器最终也都将支持。现在使用它的话，你需要用 Babel 或者 Traceur。  

再次感谢 Nick Fitzgerald 对本篇文章提供支持。  

下周，我将带来 Js 里最基础的，长期使用的，构造代码块的另一种简化版。你会关心吗？简化版会使你异常兴奋吗？我猜答案是 yes，不过别太当真。ES6 的 arrow function, 下周加入我们，来探个究竟。
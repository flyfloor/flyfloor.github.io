---
showDate: true
date: "2015-09-20"
layout: post
tags: ['tech', 'JavaScript', 'ES6']
title: "\u6DF1\u5165 ES6  - \u6A21\u677F\u5B57\u7B26\u4E32"
---
> 原文出自 [ES6 in depths](https://hacks.mozilla.org/2015/05/es6-in-depth-template-strings-2/), 作者 [Jason Orendorff](https://blog.mozilla.org/jorendorff/), 翻译：落在深海

*ES6 In Depth* 系列将详细解读 ES6 的新特性。 

上周我承诺换个节奏。继迭代器跟 generators 后，我说要换点儿简单的，不会烧糊你大脑的东西。读完这篇文章，看看我是否信守承诺。  

现在，先从简单的开始。

### ` 的基础知识 

<!--more-->

ES6 引入了新的字符字面量语法，叫做 template strings。它看起来像是传统的字符串，区别在于用 **`** 取代了**'** 或者 **"**。简单的例子: 

```javascript
context.fillText(`Ceci n'est pas une chaîne.`, x, y);  
```
    
然而被叫做 `template strings`，而不是`冗长的除了引号不同的字符串`。原因在于 template strings 给 Javascript 带来了更简单的字符串插值方式。一种很漂亮，很方便的往字符串里插值的方式。  

使用 template strings 有很多种方式，我最喜欢下面这种:  

```javascript
function authorize(user, action) {
  if (!user.hasPrivilege(action)) {
    throw new Error(
      `User ${user.name} is not authorized to do ${action}.`);
  }
}
```
    
 ${user.name} 和 ${action} 被称作 template 占位符。Javascript 会将 user.name 和 action 的值插入字符串。最后生成类似```User jorendorff is not authorized to do hockey```。  
 
 目前为止，功能仅限于比 **+** 运算符长得好看点，以下是这种用法你需要知道的细节:  
 
 * template 占位符可以是任何 Javascript 表达式，函数调用，运算操作等等(如果你愿意，甚至可以在 template string 里嵌套 template string, 我称它为 template 盗梦空间。)  
 
 * 如果占位符的值不是字符串，它将被转义为字符串。例如，如果 action 是一个对象，.toString() 方法将会被调用。  
 
 * 如果你需要在 template strings 里使用引号，请用反斜杠进行转义： **\`\\\`\`** 与 **`** 等价。  
 
 * 同样的，要使用 **$**, 也需要转义：**\`write \${\`** 或者 **\`\$\`**。  
 
 与不同字符串不同的是，template strings 可以是多行：  

```javascript 
$("#warning").html(`
   <h1>Watch out!</h1>
    <p>Unauthorized hockeying can result in penalties
    of up to ${maxPenalty} minutes.</p>
`);
```
    
template strings 里的所有空白，包括换行跟缩进，都会一字不落的输出。  

警告: 以下内容可能有些许不适。请适当酌情阅读，或许去喝杯咖啡放松下。  

### 括住未来  

让我们讨论下 template strings 做不了什么。  

* 它不会自动转义特殊字符。为了避免跨域脚本工具，对于不信任的数据仍需要你的关注，就像处理普通字符串一样。  

* 用它来做国际化时，并不容易。template strings 并没有针对不同语言做数字、时间的处理，更不用说单复数了。  

* 它并不是[Mustache](https://mustache.github.io/) 或者 [Nunjacks](https://mozilla.github.io/nunjucks/) 之类模板引擎的替代品。  

原因在于 template strings 并不包含：例如通过对数组迭代生成 HTML 的 table 结构、甚至是条件判断等语法(你可能觉得用 template 的层层嵌套能够做这件事，但这听起来就像是个笑话)。  

ES6 提供了一种隐晦的使用 template strings 的方式，来供 Js 开发者和库设计者打破上述限制。这个特性被称作 tagged templates。  

tagged templates 的语法非常简单。只需要在引号前加上额外的*tag*。上面提到html 内容转义，对应的这个*tag* 就是 **SaferHTML**， 将需要限制的字符一一列出: 自动转义特殊字符。  

注意到 **SaferHTML** 并不是 ES6 的标注库。所以我们需要自己实现：  

```javascript
var message = 
    SaferHTML`<p>${bonk.sender} has sent you a bonk.</p>`;  
```       

这里 SaferHTML 是个标示符，还可以用例如 SaferHTML.excape, 甚至是方法调用。  

例如：`SaferHTML.escape({unicodeControlCharacters: false})`(更准确点说，任何 ES6 [ MemberExpression or CallExpression](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-left-hand-side-expressions) 可以称作是一个*tag*)。  

未用 tagged 的 template strings 只是简单字符串的连接。而带 tagged 的 templates 实际上是函数调用。  

上面代码等同于：  

```javascript
var message =
  SaferHTML(templateData, bonk.sender);
```
      
 templateData 是由 Js 引擎生成的，它存储了 template strings 的字符串部分， 是不可变数组。数组包含两个元素，也就是被插入值左右隔开的两部分。所以 templateData 作用就像是 `Object.freeze(["<p>", " has sent you a bonk.</p>"]`。  
 
 (实际上 templateData 还存着另外一个属性。虽然本章不会用到，为了完整性这里还是简单提起：templateData.raw，保存字符串所有部分，但不作任何处理，例如： `\n` 不会被变为新的一行。新的 String.raw 也用到了这个特性。)  
 
 这给了 SaferHTML 函数无限种解释字符串跟占位符的方式。  
 
 再读下去之前，你应该尝试领会 SaferHTML 的逻辑，并试着实现它。因为它本身是函数，你可以在 firefox 的控制台做调试(现在都用 chrome 了吧，我不是火狐黑。。。)   

一种解答([在这里](https://gist.github.com/jorendorff/1a17f69dbfaafa2304f0)也看得到)。  

```javascript
function SaferHTML(templateData) {
  var s = templateData[0];
  for (var i = 1; i < arguments.length; i++) {
    var arg = String(arguments[i]);

    // Escape special characters in the substitution.
    s += arg.replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;");

    // Don't escape special characters in the template.
    s += templateData[i];
  }
  return s;
}
```
 
**SaferHTML`<p>${bonk.sender} has sent you a bonk.</p>`** 最终会被拼接成 **"<p>ES6&lt;3er has sent you a bonk.</p>"**。某些恶意用户即使输入像**Hacker Steve &lt;script&gt;alert('xss');&lt;/script&gt;, sends them a bonk.**这样的内容，也不会对其他用户的安全产生影响。  

一个简单的例子很难讲清楚 tagged templates 的强大之处。让我们再回顾前面遇到的问题，看看有什么能够改善的。  

* template strings 本身不提供自动转义特殊字符的功能。正如所见，通过 tagged templates， 可以很容易的解决这个问题，你可以写出更好的。  

从安全角度来看，我写的 SaferHTML 实在太弱了。HTML 的不同部分要针对不同特殊字符来转义；SaferHTML 并没有将它们转义，但通过努力，你可以对 templateData 里的每一段字符串分析，这样它便知道哪部分是元素属性，它们要对**’**跟**"** 转义；哪部分是纯 HTML； 哪部分是 URL 请求参数，而它们需要 URL 转义 而不是 HTML 转义，等等。这样才能进行正确的转义。  

HTML 解析很慢，让这听起来像牵强附会？幸运的是，tagged template 并不会改变直到 template 被再次赋值。SaferHTML 会缓存解析的结果，在被调用时起到加速作用。(缓存属于 [WeekMap](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap) 类型， 后面会讨论的 ES6 的特性。)  

* Template strings 并没有内建的国际化特性。然而使用 tags, 我们可以实现它。[Jack Hsu 的文章](//jaysoo.ca/2014/03/20/i18n-with-es6-template-strings/) 在国际化方面开了个头。例子:  

```javascript
i18n`Hello ${name}, you have ${amount}:c(CAD) in your bank account.`
// => Hallo Bob, Sie haben 1.234,56 $CA auf Ihrem Bankkonto.  
```
        
 *name* 跟 *amount* 是 JavaScript，而 :c(CAD) 是什么？答案是库的作者 Jack 设计来处理国际化的。 研究 i18n 文档会发现， :c(CAD) 意味着 amount 是货币符号，按加拿大币结算。  
 
 以上即是 template strings。  
 
 * template strings 并不是 Mustache 或者 Nunjucks 的替代品，一部分原因是它并没有循环或者条件语句之类的语法。那我们实现它吧！Js 不提供的特性，写个 tag 来提供它吧。  

```javascript
// Purely hypothetical template language based on
// ES6 tagged templates.
    var libraryHtml = hashTemplate`
      <ul>
        #for book in ${myBooks}
          <li><i>#{book.title}</i> by #{book.author}</li>
        #end
      </ul>`;
``` 
             
强大的弹性并未就此结束。注意到 tag 函数的参数并不会自动转成字符串，它们可以是任何类型，返回值也是，甚至 tagged templates 也并不需要是字符串。你可以通过 tags 来构造正则表达式，DOM 树， 图片，包含完整异步的 promise， JS 数据结构，GL 着色器等等...    
 
**Tagged templates 的开放性，允许库开发者设计各种强大的 DSL 语言(领域特定语言)。** 这些语言语法可能看起来不像 Js，但能够无缝的插入 Js 代码，并使其与其他部分智能的协同工作。我一时半会想不出有哪个语言拥有此特性。这一特性给了我们无限可能。  

### 什么时候可以用？  

服务端，template strings 在 io.js 已经可以使用。  

浏览器端，firefox 34+，chrome 41+ 已经支持，IE 跟 safari 并不支持。暂时的，你可能需要使用 Babel 或者 Traceur 来转换。你也可以在 TypeScript 直接使用。  

### 等等，Markdown 怎么办？  

额， 好问题。  

template strings 里，Javascript 跟 Markdown 的 **`** 有着不同的意义。在 Markdown 里， 它是内联 code 的标识。  

这样就会出问题，比如：  

```javascript
To display a message, write `alert(`hello world!`)`  
```

会这样显示：  

```javascript
To display a message, write alert(hello world!).  
```
    
注意到输出里面并没有**\`** 符号， Markdown 把所有的 **`** 翻译成 code，并用 HTML 标签替换掉了。  
 
要使用内联代码块，可以用多个引号来避免这个问题：  

```javascript
To display a message, write ``alert(`hello world!`)``.  
```
    
[This Gist](https://gist.github.com/jorendorff/d3df45120ef8e4a342e5)  是用 Markdown 写的，你可以阅读源文件。  

### 下一章  

下周将探索的两个特性，程序员在别的语言里已爽了许多年：1. 函数的默认参数。 2. 函数的可变参数。  

我们将从实现者的角度亲历这一特性。欢迎你加入，通过客串作者 Benjamin Peterson 的方式来深入理解ES6 默认参数、可选参数。

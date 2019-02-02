---
showDate: true
date: "2015-10-08"
layout: post
tags: ['tech', 'JavaScript', 'ES6']
title: "\u6DF1\u5165 ES6  - \u5B50\u7C7B"
---

> 原文出自 [ES6 in depths](https://hacks.mozilla.org/2015/08/es6-in-depth-subclassing/), 作者 [Eric Faust](https://hacks.mozilla.org/author/efaustmozilla-com/), 翻译：落在深海

*ES6 In Depth* 系列将详细解读 ES6 的新特性。   

两周前，我们描述了 ES6 用来操作繁琐的对象构造的[新的类系统](https://hacks.mozilla.org/2015/07/es6-in-depth-classes/)，也介绍了如何使用它写出如下的代码：  

<!-- more -->

```javascript
class Circle {
    constructor(radius) {
        this.radius = radius;
        Circle.circlesMade++;
    };

    static draw(circle, canvas) {
        // Canvas drawing code
    };

    static get circlesMade() {
        return !this._count ? 0 : this._count;
    };
    static set circlesMade(val) {
        this._count = val;
    };

    area() {
        return Math.pow(this.radius, 2) * Math.PI;
    };

    get radius() {
        return this._radius;
    };
    set radius(radius) {
        if (!Number.isInteger(radius))
            throw new Error("Circle radius must be an integer.");
        this._radius = radius;
    };
}
```

不幸的是，正如一些人指出的，根本没时间细说 ES6 类的其他强大之处。像传统的类系统(例如 C++ 或 Java)一样，ES6 允许一个类使用另一个类作为基类的继承方式，然后通过添加更多特性来扩展类。让我们近距离看看这个新特性的能耐。  

在开始讨论子类之前，有必要花点时间回顾下属性继承跟动态原型链。  

### Javascript 继承  

当创建对象时，我们有机会为其添加属性，但它同时会继承来自其 prototype 上对象的属性。Javascript 程序员对 Object.create API 应该都很熟悉：  

```javascript
var proto = {
    value: 4,
    method() { return 14; }
}

var obj = Object.create(proto);

obj.value; // 4
obj.method(); // 14  
```
    
进一步，当我们用与 proto 对象里某个属性相同的名称给 obj 的属性赋值，obj 该属性的值将覆盖来自于 proto 的属性的值。

```javascript
obj.value = 5;
obj.value; // 5
proto.value; // 4  
```
    
### 子类的基础  

牢记上面的情况后，我们现在明白了该如何将类创建的对象们关联起来。回忆起创建类时，我们创建了与 constructor 方法一致的新函数，它包含了类的所有静态方法。同时创建了该函数的 prototype 属性对象，包含了类所有实例方法。如果创建的新类要继承该类的所有静态属性，只需让这个函数对象继承自父类的函数对象。同样的，需要将该函数的 prototype 对象继承自父类的 prototype对象，来获得实例方法。  

描述太过冗长。试个例子，展示如何用新语法将类联系起来，然后添加一个琐碎的扩充来让它更美观地赏心悦目。  

继续上个例子，假设我们希望为 Shape 父类编写子类：  

```javascript
class Shape {
    get color() {
        return this._color;
    }
    set color(c) {
        this._color = parseColorAsRGB(c);
        this.markChanged();  // repaint the canvas later
    }
}
```

当写下类似上面的代码，我们遇到了与上篇文章里 static 属性同样的问题：没有语法来改变你定义的函数的 prototype 对象。当然你可以用 Object.setPrototypeOf 绕过去, 这种办法通常比直接构造目标 prototype 对象的函数性能低下且更难优化。  

```javascript
class Circle {
    // As above
}

// Hook up the instance properties
Object.setPrototypeOf(Circle.prototype, Shape.prototype);

// Hook up the static properties
Object.setPrototypeOf(Circle, Shape);
```
    
这太丑了。我们加入类的语法，这样可以把所有类相关的逻辑放到一起，而不是用其他的“连接”逻辑。Java，Ruby，以及其他面向对象语言都有定义子类继承的语法，Js 也应该有。我们用关键字 extends 表示继承，所以可以这样写：  

```javascript
class Circle extends Shape {
    // As above
}  
```
    
你可以在 `extends` 跟任何表达式，只要它的构造函数拥有 prototype。例如：  

+ 另一个类  

+ 已有继承框架里的类似于类的函数  

+ 普通函数  

+ 包含函数或者类的变量  

+ 对象的可使用属性  

+ 函数调用  

如果你不想实例继承自 Object.prototype 的话，甚至可以使用 `null`。  

### super 属性  

于是我们能够构造子类，并继承属性，有时甚至可以覆盖被继承的方法。但如果我们不想覆盖方法呢？  

假设由于某些原因，我们需要编写一个 Circle 的子类来处理圆形的缩放。我们需要写有点牵强的代码：  

```javascript
class ScalableCircle extends Circle {
    get radius() {
        return this.scalingFactor * super.radius;
    }
    set radius() {
        throw new Error("ScalableCircle radius is constant." +
                        "Set scaling factor instead.");
    }

    // Code to handle scalingFactor
}
```
    
注意到 radius 的 getter 方法使用了 super.radius。`super` 关键字允许我们绕过自身的属性，并从自身 prototype 开始来寻找属性。同样也绕过了我们做的所有可能的覆盖。  

父类属性访问器(super[expr] 同样可行)可以在任何以方法定义的函数里使用。而这些函数可以从原始的对象扯下来，当方法初次被定义时访问器便被绑定到对象上。这意味着将该方法赋值给局部变量也不会改变 super 访问器的行为：  

```javascript
var obj = {
    toString() {
        return "MyObject: " + super.toString();
    }
}

obj.toString(); // MyObject: [object Object]
var a = obj.toString;
a(); // MyObject: [object Object]  
```
    
###子类内建命令  

你可能想要为 Javascript 内建指令编写扩展。内建的数据结构为语言提供了许多强大的能量，而且能够创造新类型像杠杆一样利用该能量实在太有用了，这也是设计的子类的基础功能之一。假设你想写个带版本控制的数组(我知道，相信我，别说话。)你应该提供能够改变数据并提交，以及回滚提交等功能。下面是通过 Array 子类的一种快速实现方式：  

```javascript
class VersionedArray extends Array {
    constructor() {
        super();
        this.history = [[]];
    }
    commit() {
        // Save changes to history.
        this.history.push(this.slice());
    }
    revert() {
        this.splice(0, this.length, this.history[this.history.length - 1]);
    }
}
```
    
VersionedArray 的实例们保存了一些重要的属性。它们完善了 map，filter 跟 sort，属于数组真正的实例。Array.isArray() 将视它们为数组，甚至它们会得到像数组一样的自动更新属性 length。更深远的，能够返回新数组的函数(像 Array.prototype.slice()) 将也能返回 VersionedArray！  

### 派生类构造函数  

你可能注意到上个例子构造函数里的 super() 函数，究竟发生了什么？  

传统的类模型中，构造函数用来初始化该类实例对象的任何内部状态。每个连续子类负责初始化跟子类相关的状态。我们想将调用串起来，这样子类之间就可以共享被继承类的同样的初始化代码。  

这次我们还是使用 super 关键字来调用父构造函数，好像这次 super 是个函数。注意这种语法仅支持在使用 `extends` 的类的构造函数中使用。利用 super 关键字，将 Shape 类重写：  

```javascript
class Shape {
    constructor(color) {
        this._color = color;
    }
}

class Circle extends Shape {
    constructor(color, radius) {
        super(color);

        this.radius = radius;
    }

    // As from above
}  
```
    
Javascript 里，我们倾向于写构造函数来操作 this 对象，插入属性以及初始化内部状态。通常当使用 `new` 调用构造函数，就像在构造函数的 prototype 属性上使用 Object.create()，this 对象也会被创建。然而某些内建指令拥有不同的内建对象布局，比如数组在内存中的储存的方式跟普通对象就不太一样。正因为想为内建指令构造子类，我们让基类的构造函数分配 this 对象。如果是内建指令，我们能得到对象的布局，但如果是普通构造函数，我们只能得到默认的 this 对象。  

可能最奇怪的结果莫过于 this 跟子类构造器的绑定方式了。除非运行基类的构造函数，且允许它来指定 this 值，不然我们将得不到 this 值。因此如果还未调用 super 构造函数之前，获取 this 的值将得到 ReferenceError。  

上篇文章我们了解到你可以省略构造函数，下面的语法可以使得衍生类的构造函数被省略：  

```javascript
constructor(...args) {
    super(...args);
}  
```
    
一些时候，构造函数并不需要 this 对象。取而代之，它们构造某些对象，初始化并直接返回。这样情况下就不需要使用 super。任何构造函数都会直接返回对象，独立于是否父类的构造函数被调用。  

### new.target  

使用基类来分配 this 对象的另一个副作用是有些时候基类压根儿不知道该分配哪种类型的对象。假设你正在写一个框架库，想要一个基类 `Collection`，但是子类中有些是数组，有些却是 maps。那么当执行 `Collection` 的构造器时，你没办法知道究竟初始化的是哪种类型的 `Collection`！  

既然我们能为内建指令构造子类，而当执行内建指令的构造器时，在内部已经能得到原始类的原型。没有它，我们将无法根据合适的实例方法构造对象。为了解决 `Collection` 这个奇怪的例子，我们加入了新的语法，好让那些信息暴露给 Javascript 代码。我们增加了新的元属性 `new.target`，它与直接与调用 `new` 获得的构造函数一致。用 `new` 的方式调用函数将设置 `new.target` 为被调用的函数，而调用 `super` 则会转发 `new.target` 值。

这不太好理解，所以举个例子给你来阐述我的意思：  

```javascript
class foo {
    constructor() {
        return new.target;
    }
}

class bar extends foo {
    // This is included explicitly for clarity. It is not necessary
    // to get these results.
    constructor() {
        super();
    }
}

// foo directly invoked, so new.target is foo
new foo(); // foo

// 1) bar directly invoked, so new.target is bar
// 2) bar invokes foo via super(), so new.target is still bar
new bar(); // bar
```

这样我们就解决了上面 `Collection` 的问题，因为 `Collection` 的构造器可以检查 new.target 的直系来源，也就能够决定使用哪个类型来构造。  

`new.target` 可以在任何函数里使用，如果函数不是通过 `new` 构造的，它将被设置为 undefined。  

#### 两全其美

希望你从新特性的洗礼中生存下来。感谢你还在坚持。让我们花点时间来讨论下问题是否都被很好地解决了。许多人对将继承编纂进语言特性是否是件好事直言不讳。你或许相信继承在创建对象方面远没有[组合](https://en.wikipedia.org/wiki/Composition_over_inheritance)好用，或者新语法的清洁换来缺乏设计灵活性的结果并不值得，相比于老款 prototype 类型。不可否认的是 mixins 在创建可扩展包含共享代码的对象方面已变成了约定俗成，有充分的理由：他们提供了一种简单的方式来分享无关联的代码到同一个对象而无需理解这两个无关联的东西是否应适合同一继承结构。

本期主题有许多强烈的信念，但我认为只有少部分值得记录。第一，类作为语言特性被添加并不代表必须强制使用。第二，同样重要的是，类作为语言特性被添加并不表示他们总是解决继承问题的最好途径！事实上，许多问题更适合用原型继承的方式来解决。在一天结束时，类仅仅是你可以使用的另一个工具；并不是唯一工具，也并不一定是最好用的。

如果你想继续使用 mixins，你可能希望类能够多重继承，这样你便能够继承自不同的 mixin，且让所有事情都变更好。很遗憾，现在修改继承模型似乎有点不和谐，所以 Javascript 并不支持类的多重继承。话虽如此，目前还有一种混合的解决方案，它允许 mixins 出现在类框架的内部。思考下面的函数，基于知名的 `extend` mixin 方言。  

```javascript
function mix(...mixins) {
    class Mix {}

    // Programmatically add all the methods and accessors
    // of the mixins to class Mix.
    for (let mixin of mixins) {
        copyProperties(Mix, mixin);
        copyProperties(Mix.prototype, mixin.prototype);
    }
    
    return Mix;
}

function copyProperties(target, source) {
    for (let key of Reflect.ownKeys(source)) {
        if (key !== "constructor" && key !== "prototype" && key !== "name") {
            let desc = Object.getOwnPropertyDescriptor(source, key);
            Object.defineProperty(target, key, desc);
        }
    }
}
```

我们现在可以使用 `mix` 函数来创造组合父类，而不需要在多个 mixins 之间创建显式的继承关系。想象正在编写合作编辑工具，这工具编辑行为需要被日记记录的，并且内容需要被序列化。你可以使用 `mix` 函数来书写 `DistributedEdit` 类：

```javascript
class DistributedEdit extends mix(Loggable, Serializable) {
    // Event methods
}
```

两全其美。也很容易看到该如何扩展模型来处理含有父类的 mixin 类：我们只需简单的把父类传给 `mix`，并在返回类中继承自他。  

### 当前可用性  

好了，我们聊了许多关于子类的内建指令和这些新东西，但现在可以用他们么？  

额，一部分吧。主要的浏览器供应商，我们今天讨论到的，Chrome 支持的最多。在严格模式下，你可以使用我们讨论过的所有东西，除了 Array 子类。其他内建类型可以工作，但 Array 带来了一些额外的挑战，它尚未完成也并不奇怪。我在为 Firefox 编写实现，旨在尽快完成同样的目标(除了 Array)。[bug 1141863](https://bugzilla.mozilla.org/show_bug.cgi?id=1141863) 查看更多信息，但这个几周之后便会在测试版 Firefox 里出现。 

进一步，Edge 支持了 super，但不支持子类内建指令，Safari 不支持所有功能。  

这里使用翻译编译器是不太有利的。他们能够创建类，也能用 `super`，但却基本上没有方法伪造子类内建指令，因为你需要引擎支持从内置方法返回基类的实例(想想 `Array.prototype.splice`)。

咳！好长一篇文章。下周，Jason Orendorff 将回来探讨 ES6 的模块系统。
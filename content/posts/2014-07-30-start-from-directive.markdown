---
showDate: true
date: "2014-07-30"
layout: post
tags: ['tech', 'AngularJs', 'note']
title: "AngularJs\u6307\u4EE4(Directive)\u521D\u63A2"
---


>完成一个简单的directive，初探angular

关于angular，大多的概念都很好理解，很快就能上手。当疑惑如何嵌入模板片段时，发现需要用directive(指令)，这部分相对难啃，下面只是个非常简单的例子，大致介绍如何使用指令。

#### main.html  

```html
<!DOCTYPE html>
<html>
  <head>
    <title></title>
    <script src="angular.js"></script>
  </head>
  <body>
  </body>
</html>
```  

<!-- more -->  

#### 然后创建输入框，作为email的receiver输入
  
```html
<input id="email_to" type="email" ng-model="to"/>
```

这里有个model```to```， 用来获取input输入，会被双向绑定到指令内部的scope上，后面再讲如何做。
  

#### 创建模块和第一个指令

module:  

```javascript
var app = angular.module('app', []);
``` 

directive:  

```javascript
app.directive('myDir', function() {
  return {
    restrict: 'A',
    require: '^mailTo',
    scope:{
      mailTo: '=',  
      onSend: '&', 
      mailObj: '=', 
      fromName: '@'
    },
    templateUrl: 'template.html'
    }
});
```

  
+ ```restrict``` 该指令如何被使用：

```html
'A' - <span ng-sparkline></span>
'E' - <ng-sparkline></ng-sparkline>
'C' - <span class="ng-sparkline"></span>
'M' - <!-- directive: ng-sparkline -->  
```

+ ```templateUrl```  

```javascript
templateUrl: 'template.html'   
```
模板片段  

+ 这里scope是该指令内部独立的scope，与外层scope隔离。  

```javascript
scope:{
  mailTo: '=',  
  onSend: '&',  
  mailObj: '=',   
  fromName: '@'
},
```

- mailTo 定义为```=```，表示他是与外层scope的model是双向绑定的
- onSend 用到```&```, 引用到外层作用域的方法，且需要传递变量进去。
- 使用```@```绑定局部变量到dom的属性上 也就是绑定到formName上。

通过这三种方式，就能使相互隔离的内外scope进行交互了。

#### 在view里使用该指令

```html
<input id="email_to" type="email" ng-model="to"/>
<div my-dir mail-to="to" mail-obj="mail" on-send="sendMail(email)" from-name="ari@fullstack.io" >
</div>
```


#### 接下来是模板片段（templateUrl）  

template.html

```html
<div>
  <button id="send_btn" ng-click="onSend(mailObj)">send</button>
  <p>from: {\{fromName}}</p>
  <p>to: {\{mailTo}}</p>
  <p>status: {\{mailObj.status}}</p>
</div>
```


button点击时候，改变email的发送状态。  
这里```onSend(mailObj)``` 就用到指令内部scope的方法及参数，onSend是对外层scope的```sendMail(mail)```的引用，而mailObj则是外层mail这个model的双向绑定，mailTo同理，目前```mailObj```、```sendMail(mail)```以及```mailObj.status```还不存在，接下来创建控制器，通过控制器来完成指令内部```mailObj```对应外部mail对象的操作。

#### 控制器

```javascript
app.controller('mydirCtr', ['$scope', function($scope) {
  $scope.mail = {
    status: 'not send'
  };

  $scope.sendMail = function(mail) {
    scope.mail.status = 'Sent';
  };
}]);
```

#### 如果这个mail对象需要在多个控制器之间操作，创建一个factory会更好点

```javascript
app.factory('Mail', function(){
  return {
    name: '',
    content: '',
    status: 'not send'
  }
});
```
  
于是控制器就可以改为

```javascript
app.controller('mydirCtr', ['$scope', 'Mail', function($scope, Mail) {
  $scope.mail = Mail;
  
  $scope.sendMail = function(mail) {
    Mail.status = 'Sent';
  };
}]);
```

#### 在main.html里添加控制器的范围

```html
<body ng-app="app" ng-controller="mydirCtr">
  <input ... />
  <div my-dir ...
...
```

这样mydirCtr控制器的scope就是body的范围，也就是外部scope，template.html即为内部scope，而```<div my-dir...></div>```就像是占位符，通过自定义指令的绑定跟引用，完成内外交互。

因为接触angular很短暂，许多东西还待深入研究。这有篇关于directive的文章，讲的就比较深入 <a href="//www.ng-newsletter.com/posts/directives.html" target="_blank">Build custom directives with AngularJS</a>，hope it can help you。  

写在最后： 从做项目或产品来说，  

1. 如果是web app， angular有很大的优势，虽略有难度，值得一试；  
2. 如果并非web app，我个人喜欢rails服务器端渲染，因为目前前端渲染的方式无论如何都有痛处。反观rails，更好的orm，强大整洁的路由，服务端模板的灵活，胶水似的helper，服务端render的多选择性，以及DRY，最重要的是让你思路清晰，真正重心放在产品本身上。谁说程序员必须是成天灰头土脸、油光满面调试bug的苦逼呢？诚然rails的确造就了一批懒人，但前提是你需要熟悉rails的思维，理顺以后，才能如鱼得水般畅快自在。
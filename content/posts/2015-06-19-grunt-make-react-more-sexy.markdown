---
date: "2015-06-19"
layout: post
tags: ['tech', 'collection', 'React', 'Grunt']
title: "\u7528Grunt\u6784\u5EFAReactJs\u5F00\u53D1\u73AF\u5883"
---

> 本文主要是讲如何使用grunt来构建react开发环境

最近计划将性感的React投入真正应用，在开发环境上小折腾了一番，主要纠结在于前端构建工具和模块化编程方式的选择。

![image](/images/grunt/grunt-react.png)

<!-- more -->

关于模块化编程，在此之前用过requireJs的方式来写模块，纠结的是移动端react压缩后也有100k以上大小，投入过多的库，可能导致移动端体验变差，于是决定放弃requireJs，采用CommonJs的方式来写，正好Sublime 的react插件自动补全也是这种方式。  

关于构建工具，由于一直使用grunt 构建工程的，而网上使用react的，大多是用webpack来做前端工程化，对于习惯使用grunt来构建项目的开发者来说，有一定学习成本。思考再三，决定继续使用grunt。以下便是使用grunt构建react开发环境的简要介绍。

####首先，安装grunt  

关于grunt的安装、使用在 [adesk-webapp-template](https://github.com/androidesk/adesk-webapp-template) 有介绍。此处省略。

####react 开发环境 

+ 安装grunt-react  
grunt react 用来将 React's JSX templates 编译成Javascript。  

详细使用方法: [grunt react](https://github.com/ericclemmons/grunt-react)

安装：  

	npm install grunt-react --save-dev

在Gruntfile里添加:  

	grunt.loadNpmTasks('grunt-react'); 
	
src目录：

	└── src                      
        ├── component     // 所有JSX文件
        ├── build           // 通过grunt react编译后的.js文件
        ... 
 
 
配置:  

    files: {
        expand: true,
    	cwd: 'js/component',
    	src: ['**/*.jsx'],
    	dest: 'js/build',
    	ext: '.js'
    }
	...

 
+ 安装grunt-browserify  
grunt browserify 用来将CommonJs风格的Javascript代码打包。使用这个插件将grunt react编译后的js文件打包，最终在浏览器中使用。  

详细使用方法：[grunt browserify](https://github.com/jmreidy/grunt-browserify)

安装：  

	npm install grunt-browserify --save-dev  
	
注册task:

	grunt.loadNpmTasks('grunt-browserify');
	
配置:  

	dist: {
        files: {
            '../js/<%= pkg.name %>.js': ['js/build/*.js', 'js/base.js'],
        },
        options: {
            transform: [ require('grunt-react').browserify ]
        }
    }
    ...

至此，react开发的环境就绪。可以开始写React了 : )。

####后续:  

>回头看，本篇旨在介绍初始化工作，工具不好用那就不算结束。  

由于我用的是 sublime text, 一开始是手写 jsx, 简直成了工作的瓶颈。心想不是还有 emmet 么，如果能像写 html 一样自动补全 jsx 就好了，google 一下，果然有办法:  

    [
        // add to Preferences > Key Bindings - User
        // see //stackoverflow.com/a/26619524 for context

        { "keys": ["tab"], "command": "expand_abbreviation_by_tab",
          "context": [
            {
              "operand": "source.js", 
              "operator": "equal", 
              "match_all": true, 
              "key": "selector"
            },
            {   
              "key": "selection_empty", 
              "operator": "equal", 
              "operand": true,
              "match_all": true 
            }
          ]
        },
        { "keys": ["tab"], "command": "next_field", "context":
          [
            { "key": "has_next_field", "operator": "equal", "operand": true }
          ]
        }
    ]

上面的的配置 copy 到 sublime key bindings user, 从此舒服了。

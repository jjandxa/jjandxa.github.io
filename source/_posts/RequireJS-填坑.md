---
title: RequireJS 填坑
date: 2016-05-09 10:10:40
tags:
- JavaScript
- RequireJS
- 前端
categories:
- JavaScript
- RequireJS
- 前端
---

**什么是 RequireJS** — 自己 Google 去。

## Start

[点我去官网](http://requirejs.org/)

刚认识用 RequireJS 之前，我还不知道前端还能这样模块化编程，尽管我是个渣，但当我第一次用上之后就回不去以前了，模块化管理你的 JS 是很有必要的，有利于前端工程的规范化管理，页面也不用写很多 link 标签和 script 标签了。

## AMD

由于 RequireJS 是基于 AMD 规范的，我们需要来了解它是怎么使用的。

```
//hello.js
define(['jquery'],function($) {
    //$('body')
})
```

这是一个基于的 AMD 规范定义的模块，它接收两个参数，第一个为依赖的字符串数组，第二个为回调的函数。这个例子表明它依赖于 jquery ，RequireJS 在调用加载这个模块时会提前将 jquery 加载进来，并作为参数传进回调函数。

然而我们要怎么使用这个模块呢？

`var hello = require(['hello'])`

这样就可以将一个 define 模块加载进来，并且使用它。然并卵，我们还是结合实例讲解更容易让人理解。

## Hello RequireJS

以往我们调用 JS 无非是使用外联标签 script 将 JS 文件引入，这样无可避免的会出现如下丑陋的场景

```javascript
<script src="a.js" ></script>
<script src="b.js" ></script>
<script src="c.js" ></script>
```

可以想象，当你需要引入的 JS 文件越来越多时会是一副什么景象。正确的做法是

```javascript
<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Hello RequireJS</title>
  <script src="require.js" ></script>
</head>
<body>
</body>
<script>
  var a = require('a')
  var b = require('b')
  var c = require('c')
</script>
</html>
```

怎么样？是否简洁了很多呢？然而还不够，我们希望 JS 代码尽量抽离出 HTML ，所以我们可以这样写

```
// main.js
define(['a', 'b', 'c'], function(a, b, c) {

})
// index.html
<!DOCTYPE html>
<html lang="en"><head>
<meta charset="UTF-8">
	<title>Hello RequireJS</title>
	<script data-main="main" src="require.js" ></script>
</head>
<body>
</body>
</html>
```

实在太简洁了，优美的代码总是让人愉悦，不是么？

## 开始使用啦

由于我们的实际项目没有这么简单，所以我们需要对 RequireJS 进行一些基础配置先看一下项目目录
[![project](/images/20160627REQUIREJS1.png)](https://jjandxa.github.io/images/20160627REQUIREJS1.png)

```
// main.js
require.config({
  // 所有的路径会以 baseUrl 为基础    
  baseUrl: 'static/lib',    
  // 定义模块    
  paths: {        
    app: '../js'.        
    jquery: 'jquery-3.0.0'    
  }
})
// js/hello.js
define(['jquery'], function ($) {    
	$('body').append('<p>Hello RequireJS!</p>')
})
// index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta content="width=device-width, initial-scale=1" name="viewport"/>
  <title>Hello RequireJS</title>
  <script src="static/require.js" ></script>
</head>
<body>
</body>
<script>
require(['static/main'], function () {
	require(['app/hello'])
})
</script>
</html>
```

完成啦，只需要在 HTML 页面引入 require.js ，并且依赖 main.js 全局配置文件，然后就可以尽情依赖自己的模块了！！
打开 index.html
[![index](/images/20160627REQUIREJS2.png)](https://jjandxa.github.io/images/20160627REQUIREJS2.png)

## 非AMD模块的兼容

我们知道 jquery 有很多优秀的插件，它们都依赖于 jquery ，但是很遗憾的是有很多插件并不支持 AMD 规范，这导致我们依赖这些插件时会遇到一些错误，我们该怎么处理呢？

```
require.config({    
  //省略    ...    
  shim: {        
    'jquery-plugin': {
      deps: ['jquery']
    }    
  }
})
```

RequireJS 提供了 shim 选项为非 AMD 模块进行兼容，我们为 jquery-plugin 插件手动配置了其依赖于 jquery ，这样 RequireJS 会确保 jquery 在 jquery-plugin 之前加载！
**是不是很完美呢？**附上源码:
[requirejs-demo](https://github.com/jjandxa/requirejs-demo)
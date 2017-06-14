---
title: Webpack2 学习记录 — 第二章·模块热替换/兼容配置
date: 2017-06-13 12:28:37
tags:
- JavaScript
- Webpack
- NodeJS
- 前端
categories:
- JavaScript
typora-root-url: ../../source
---

![lines-of-code](/images/webpack-guide-2/lines-of-code.jpg)

<!-- more -->

# 前言

　　上一章我们完成了 Webpack 的基础配置。对项目常用的资源都进行了模块化，这一章主要说明在基于 Webpack 的项目上实现**模块热替换 (HMR)**。**模块热替换**表现在开发上，就是当我们修改某段 html 或者 js 时。浏览器会自动完成代码的替换切不需要刷新浏览器。别看这个事情很小，一旦习惯了这种设定，就很难回到传统的开发方式了。它对开发效率上的提升是肥肠有必要的，所以我们今天就在基于 Webpack 的项目上来实现这一特性。Webpack 里实现 **HMR** 有两种方式：

1. webpack-dev-server

   自带服务器、配置简便。

2. webpack-dev-middleware

   这是一个中间件，必须搭配 express 等其他服务器来使用。配置较为复杂，需要有 express 等 Node 服务器使用经验。

这两个我们都会讲到

> HMR： Hot Module Replacement

# webpack-dev-server

　　基于上一章的[源代码](https://github.com/jjandxa/webpack-demo/tree/demo2)进行开发，我们为其添加 webpack-dev-server 依赖 `yarn add webpack-dev-server` ，并且在 **package.json** 文件中为 **scripts** 属性添加一条脚本。

```json
// package.json

{
  // 省略代码
  "scripts": {
    "dev": "webpack-dev-server --open"
  },
  // 省略代码
}
```

最后执行 `yarn run dev` ，正常情况下会自动打开浏览器并且访问项目。仔细观察，当我们改动 html 或 js 的时候会触发 Webpack 的构建过程，并且浏览器自动刷新了。这个阶段我们叫做**实时重载**，如果没有太多要求，这个时候也就已经够用了。但如果项目是一个 SPA 应用，那就需要更进一步的实现 **HMR** ，实现重新加载模块而不刷新页面。因为对于 SPA 应用来说 **HMR** 能更好的发挥它的作用。将 dev 脚本更改为 `"dev": "webpack-dev-server --open --inline -- hot"` 启用热替换功能。更改 **common.css** 中 box 的背景色，保存后查看浏览器状态。会发现页面在不刷新的情况下更新了 div.box 的背景色。观察控制台的内容，提示已经更新了模块。![2](/images/webpack-guide-2/2.png)

> 需要注意的是 index.html 并不在依赖索引中，尽管会触发 webpack 的构建过程，但 webpack-dev-server hmr 并不会重载 index.html 的内容。

[源代码](https://github.com/jjandxa/webpack-demo/tree/webpack-dev-server)

# webpack-dev-middleware

　　同样使用上一章[源代码](https://github.com/jjandxa/webpack-demo/tree/demo2)进行开发。相对于 **webpack-dev-server** 来说，因为需要另外配置服务器，所以**webpack-dev-middleware** 的使用相对复杂一点。

> 这里需要有点 express 基础

添加 **wepack-dev-middleware** 与 **express**依赖 `yarn add wepack-dev-middleware express` 。新建 **build/dev-server.js** 文件：

```javascript
// dev-server.js

var express = require("express");
var webpackDevMiddleware = require("webpack-dev-middleware");
var webpack = require("webpack");
var webpackConfig = require("../webpack.config");

var app = express();
var compiler = webpack(webpackConfig);

app.use(webpackDevMiddleware(compiler, {
  publicPath: "/" // 大部分情况下和 `output.publicPath`相同
}));

app.listen(8080, function () {
  console.log("Listening on port 8080!");
});
```

 以上代码的作用是启动 webpack ，并且配置 webpack-dev-middleware 中间件，最后启动 express。这仅仅是创建了一个基于 webpack 的开发服务器，实现 HMR 我们需要再添加一个依赖 **webpack-hot-middleware** `yarn add webpack-hot-middleware` 。在 webpack.config.js 中修改一下内容：

```javascript
// webpack.config.js
// 省略相关代码
var webpack = require("webpack")

module.exports = {
  entry: ['./build/dev-client.js', './src/main.js'],
  // 省略相关代码
  plugins: [
    // 省略相关代码
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoErrorsPlugin()
  ]
}
```

创建 **build/dev-client.js** 文件：

```javascript
// dev-client.js
require('eventsource-polyfill')
var hotClient = require('webpack-hot-middleware/client?noInfo=true&reload=true')

hotClient.subscribe(function (event) {
  if (event.action === 'reload') {
    window.location.reload()
  }
})
```

安装 eventsource-polyfill 依赖以兼容不支持事件源的浏览器 `yarn add eventsource-polyfill` ，添加完以上内容后，修改 **dev-server.js** 文件：

```javascript
// dev-server.js
// 省略相关代码
var webpackHotMiddleware = require("webpack-hot-middleware")

var compiler = webpack(webpackConfig);

var hotMiddleware = webpackHotMiddleware(compiler);

// 解决 index.html 修改后不刷新问题
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emit', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' })
    cb()
  })
})

app.use(hotMiddleware);

// 省略相关代码
```

更改 **package.json** 的 `dev` 脚本 为 `node build/dev-server.js` ，运行 `yarn run dev` 启动服务。

测试效果，实现了 HMR ，并解决 **webpack-dev-server** 下 **index.html** 修改不刷新的问题。

[源代码](https://github.com/jjandxa/webpack-demo/tree/webpack-dev-middleware)

# 第三方模块兼容

　　大多数情况下，我们还会依赖一些对全局变量有要求的第三方库。例如各类 Jquery 插件，依赖于全局变量 **$** 或 **jquery** 。对于这种情况，可以使用 ProvidePlugin 插件。参考一下情况：

```javascript
// webpack.config.js

var webpack = require("webpack")

module.exports = {
  // 省略相关代码
  plugins: [
    // 省略相关代码
    new webpack.ProvidePlugin({
      $: 'jquery',
      jQuery: 'jquery'
    })
  ]
}
```

main.js 中使用 jquery ：

```javascript
// main.js

require('./css/common.css')

$("#app").addClass("box")
```

测试后一切正常！

兼容第三方模块还有其他方式，通过 `imports-loader` 、 `exports-loader` 都是解决第三方依赖的方式，可以参考[官网相关资料](https://doc.webpack-china.org/guides/shimming/)。

[源代码](https://github.com/jjandxa/webpack-demo)
---
title: Webpack2 学习记录 — 第一章·最小可用配置
date: 2017-05-29 22:24:53
tags:
- JavaScript
- Webpack
- NodeJS
- 前端
categories:
- JavaScript
typora-root-url: ../../source
---

![logo](/images/webpack-guide-1/logo.svg)

<!-- more -->

> 在大前端的路上，总有那么些坑需要去踩。

　　相比于 14 年初刚出来工作，那时对前端的感觉远没有现在敏感，也不会想到现在的前端会发生天翻地覆的变化。细数自己对前端领域的技能，不懂的还有很多。NodeJs 也是后来粗略看了些，略微了解。前端自动化方面则一直没有时间去学习，例如 Gulp 、Grunt 都一窍不通。而前端模块化方面，之前略微了解 RequireJS ，Webpack 也曾手动写过配置。如今 Webpack 已升级到 2.0 + ，感觉是一个系统的学习并记录下过程的好时机。基于这个原因，就有了这个系列。— **Webpack2 学习记录**

# 基础

　　对于初次接触 Webpack 甚至 NodeJS 的同学，学习起来可能会有些困难。对于这种情况，建议还是先学习一些 NodeJS 的基础知识再学习 Webpack 会好些。Webpack 是一个模块化打包工具，它能将 JavaScript 、 CSS 、 JSX 、 图片等等文件进行模块化加载且打包。我们不在这里赘述模块化工程的重要性，因为这是越来越壮大的前端开发工作所不可避免的部分。Webpack 所主张的是 **一切皆模块** 的理解，我们就来看看它是怎么使用的吧！

## Hello Webpack!

```javascript
// lib/utils.js
export function show () {
  console.log('Hello Webpack!')
}

// main.js
var utils = require('./lib/utils')
utils.show()
```

> 以往，我们在 HTML 里对 JS 代码的程序依赖只能通过 script 标签的加载顺序控制。这样的坏处是程序的依赖关系不可见，代码里没有显式的表名程序需要的依赖。导致工程维护困难，这也正是前端模块化工程出现的原因之一。在上面的程序里，我们可以看到在 main.js 依赖与 utils.js 模块，优势是显而易见的。但我们现在要怎么使用它呢？接下来就是 Webpack 发挥作用的时候了。

```javascript
// webpack.config.js
module.exports = {
  entry: './src/main.js',
  output: {
    path: '/Users/aixiaoai/nodejs/webpack-demo/dist',
    filename: 'bundle.js'
  }
}
```

执行 `webpack` 就会开始进行构建。打包成功后，项目下的 dist 目录会多出 bundle.js 文件。这就是打包后的程序，我们可以试着运行看看，执行 `node dist/bundle.js`

```
webpack-demo$ node dist/bundle.js
Hello Webpack!
```

项目被成功打包了，我们可以把该 JS 文件引入 HTML 了，是不是很酷！接下来我们讲解一下以上是如何实现的。

## 讲解

1. 请尽量将项目作为 NodeJS 项目运行，因为在此基础上我们需要使用很多依赖于 NPM 包管理器的工具。相关命令 `npm init` 或 `yarn init`

2. 安装 webpack `npm install webpack --save-dev` 或 `yarn add webpack`

3. webpack 默认会寻找项目下命名为 webpack.config.js 的文件作为配置。所以我们新建一个名为 webpack.config.js 的文件，内容与上面的示例相同

   > 这里有一个需要注意的地方，output.path 的值需要是绝对路径

4. 运行 `webpack`

[源代码](https://github.com/jjandxa/webpack-demo/tree/demo1)

# 进阶

　　在上面的例子中我们讲到使用 Webpack 来打包一个 JavaScript 文件。有同学肯定会问，不是说 Webpack 的理念是**一切皆模块**吗？那其他文件怎么办？这里我们需要讲到 Webpack 的 **rules** ，在 Webpack 1.x 时代叫做 **loaders**。

## 模块化加载 CSS

　　Webpack 里针对不同文件转化为模块的做法，是通过相对应的模块加载器实现的。例如 ES6 可以用 babel-loader 来加载、图片文件可以用 url-loader 来加载、CSS 可以用 style-loader|css-loader 加载。拿 CSS 文件来举例：

安装 style-loader和css-loader `npm install style-loader css-loader --save-dev ` 或 `yarn add style-loader css-loader`

```css
/* css/common.css */
.box {
  background-color: #666666;
  width: 400px;
  height: 400px;
}
```

```javascript
// main.js
require('./css/common.css')

document.getElementById('app').className = 'box'

// webpack.config.js
module.exports = {
  entry: './src/main.js',
  output: {
    path: '/Users/aixiaoai/nodejs/webpack-demo/dist',
    filename: 'bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  }
}
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <div id="app"></div>
</body>
<script src="./dist/bundle.js"></script>
</html>
```

执行 `webpack`，如果没有报错即是打包成功。我们来看看效果：

![webpack-1](/images/webpack-guide-1/webpack-1.png)

成功应用了 CSS ，很酷吧！

## 模块化加载图片等资源文件

对于图片也是如此，我们加入 url-loader 试试。安装 url-loader `npm install url-loader --save-dev` 或 `yarn add url-loader`

```javascript
// webpack.config.js
// 省略相关代码
module.exports = {
  // 省略相关代码
  module: {
    rules: {
      // css-loader 省略相关代码
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
          use: {
            loader: 'url-loader',
            options: {
              // 如果超过大小限制，则不进行编码，输出到/img目录
              limit: 10000,
              name: '/img/[name].[hash:7].[ext]'
            }
          }
      }
    }
  }
}

// main.js
// 省略相关代码
document.getElementById('img').src = require('./img/logo.svg')
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <div id="app"><img src="" alt="" id="img"></div>
</body>
<script src="./dist/bundle.js"></script>
</html>
```

执行 `webpack`命令，查看效果：

![webpack-2](/images/webpack-guide-1/webpack-2.png)

666 ！你们说，到底 6 不 6 ！但是！有同学会说了，不想在 JS 中加载图片，想直接在 img 标签加在 src 属性里可以吗？当然可以！

## 基于 HTML 加载图片

　　我们发现需要在 JS 中加载图片，这在开发上难免会有些不便。这里需要使用 **html-loader** 来解决这个问题，安装`npm install html-loader --save-dev` 或 `yarn add html-loader`，加载器的配置也是老生常谈了：

```javascript
// webpack.config.js
module.exports = {
  // 省略相关代码
  module: {
    rules: {
      // 省略相关代码
      {
        test: /\.html$/,
        use: {
          loader: 'html-loader',
          options: {
            minimize: true
          }
        }
      }
    }
  }
}

// main.js
require('./css/common.css')

document.getElementById('app').className = 'box'
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Webpack-Demo</title>
</head>
<body>
  <div id="app"><img src="./src/img/logo.svg" alt="" id="img"></div>
</body>
<script src="./dist/bundle.js"></script>
</html>
```

效果与上一个例子是相同的，但我们可以在 img 标签的 src 属性指定图片了！

## Plugins(插件)

　　基于上面的示例，我们发现需要手动在 index.html 中引入 bundle.js 。这里要用到 Webpack Plugins 中的 **html-webpack-plugin** 插件，我们来对上面的项目进行一些改造。安装插件`npm install html-webpack-plugin --save-dev` 或 `yarn add html-webpack-plugin` ，然后我们开始配置插件：

```javascript
// webpack.config.js
var HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  // 省略相关代码
  plugins: [
    new HtmlWebpackPlguin({
      // 输出文件名
      filename: 'index.html',
      // 模板文件名
      template: 'index.html',
      // script 标签置于 body 底部
      inject: true
    })
  ]
}
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Webpack-Demo</title>
</head>
<body>
  <div id="app"><img src="./src/img/logo.svg" alt="" id="img"></div>
</body>
</html>
```

运行 `webpack` 开始构建，插件会将项目根目录下的 index.html 作为模板编译到 dist 目录下。

```
dist/
|_bundle.js
|_index.html
```

访问构建后的 index.html 查看效果。

![webpack-3](/images/webpack-guide-1/webpack-3.png)

大成功！

[源代码](https://github.com/jjandxa/webpack-demo/tree/demo2)
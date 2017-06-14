---
title: Webpack2 学习记录 — 第三章·开发/生产配置优化
date: 2017-06-14 10:24:40
tags:
- JavaScript
- Webpack
- NodeJS
- 前端
categories:
- JavaScript
typora-root-url: ../../source
---

![code-cleanup](/images/webpack-guide-3/code-cleanup.png)

<!-- more -->

　　上一章我们完成了  Webpack HMR 的配置，这也是在前端工程化开发后的标配。相信这些新的东西能让你体会到一些不一样的感觉。这一章我们主要把上一章的配置进行一些优化，并且添加一些打包压缩之类的配置。我们在第一章完成了基本的配置，能够打包输出静态文件。在第二章完成了 HMR 的配置，但是我们会发现在第二章里并不会输出静态文件。所以这里引出一个概念，在前端工程化的项目里，**开发**与**生产**并不一定会用同一套配置。**开发**时我们需要自带服务器、需要 HMR ，但是在**生产**环境并不需要这些，我们只需要一些静态文件，往 nginx 一丢就可以了。接下来主要就会讲这一部分！

# 配置优化

　　项目需要同时具备 **dev** 和 **build** 两套配置，所以我们抽离其中一些公用的配置，减少的样板代码的重复出现。仔细思考，我们会发现其中 **loaders** 部分是通用的，部分 **plugins** 也是通用的。所以我们可以在 build 目录建立一个 **base** 配置，内容与项目根目录下的 **webpack-config.js** 项目。然后去除一些不通用的配置：

```javascript
// webpack.base.js

module.exports = {
  entry: ['./src/main.js'],
  output: {
    path: '/Users/aixiaoai/nodejs/webpack-demo/dist',
    publicPath: '/',
    filename: 'bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.(png|jpe?g|git|svg)(\?.*)?$/,
        use: {
          loader: 'url-loader',
          options: {
            // 如果超过大小限制，则不进行编码，输出到/img目录
            limit: 10000,
            name: '/Users/aixiaoai/nodejs/webpack-demo/dist/img/[name].[hash:7].[ext]'
          }
        }
      },
      {
        test: /\.html$/,
        use: {
          loader: 'html-loader',
          options: {
            // 压缩 html 代码
            minimize: true
          }
        }
      }
    ]
  }
}
```

上面配置存在一些问题，例如存在一些绝对路径、多入口文件配置问题，我们可以使用 **NodeJS** 的 **path** 模块来获取正确的路径，并且对多入口文件进行修改：

```javascript
// webpack.base.conf.js

var path = require('path')

function resolve (dir) {
  return path.join(__dirname, '../', dir)
}

module.exports = {
  entry: {
    app: './src/main.js'
  },
  output: {
    path: resolve('dist'),
    publicPath: '/',
    filename: '[name].js'
  },
  module: {
    rules: [
      // 省略相关代码
      {
        test: /\.(png|jpe?g|git|svg)(\?.*)?$/,
        use: {
          loader: 'url-loader',
          options: {
            // 如果超过大小限制，则不进行编码，输出到/img目录
            limit: 10000,
            name: resolve('dist/img/[name].[hash:7].[ext]')
          }
        }
      }
      // 省略相关代码
    ]
  }
}
```

> __dirname：NodeJs Api ，获取当前目录
>
> path.join(__dirname, '../', dir)：拼接目录

但这样也不太好，因为**开发**与**生产**的构建过程我们最好通过一些环境变量来进行干预，例如一些路径我们可以通过环境不同而输出到不同路径。这里我们新建一个 **config** 目录，用来存放环境配置：

```javascript
// prod.env.js 生产环境变量

module.exports = {
  NODE_ENV: '"production"'
}
```

```javascript
// dev.env.js 开发环境变量

var merge = require('webpack-merge')
var prodEnv = require('./prod.env')

module.exports = merge(prodEnv, {
    NODE_ENV: '"development"'
})
```

> webpack-merge 模块用来合并对象，这里 dev.env.js 覆盖了 prod.env.js 的值

```javascript
// index.js 环境配置文件

var path = require('path')

module.exports = {
  build: {
    env: require('./prod.env'),
    index: path.resolve(__dirname, '../dist/index.html'),
    assetsRoot: path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',
    productionSourceMap: true,
    productionGzip: false,
    productionGzipExtensions: ['js', 'css'],
    bundleAnalyzerReport: process.env.npm_config_report
  },
  dev: {
    env: require('./dev.env'),
    port: 8080,
    autoOpenBrowser: true,
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',
    proxyTable: {},
    cssSourceMap: false
  }
}
```

接下来我们来改造 **webpack.base.conf.js**：

```javascript
// webpack.base.conf.js

var path = require('path')
var config = require('../config')

function resolve (dir) {
  return path.join(__dirname, '../', dir)
}

module.exports = {
  entry: {
    app: './src/main.js'
  },
  output: {
    path: config.build.assetsRoot,
    filename: '[name].js',
    publicPath: process.env.NODE_ENV === 'production'
      ? config.build.assetsPublicPath
      : config.dev.assetsPublicPath
  },
  resolve: {
    extensions: ['.js', '.json'],
    alias: {
      '@': resolve('src')
    }
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'eslint-loader',
        enforce: 'pre',
        include: [resolve('src'), resolve('test')],
        options: {
          formatter: require('eslint-friendly-formatter')
        }
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test')]
      },
      {
        test: /\.(png|jpe?g|git|svg)(\?.*)?$/,
        use: {
          loader: 'url-loader',
          options: {
            // 如果超过大小限制，则不进行编码，输出到/img目录
            limit: 10000,
            name: resolve('dist/img/[name].[hash:7].[ext]')
          }
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: resolve('dist/fonts/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.html$/,
        use: {
          loader: 'html-loader',
          options: {
            // 压缩 html 代码
            minimize: true
          }
        }
      }
    ]
  }
}
```

> 这里添加了 Babel 、字体相关配置。并且将 css-loader 去除，因为 css 相关 loader 比较多。需要追加一些 sass、less  等等。封装成一个工具函数比较好

> Babel：用来将 ES6 编译成 ES5 的工具

> ESLint 需要添加一下配置文件

```
// .eslintignore

build/*.js
config/*.js
```

```
// .eslintrc.js

module.exports = {
  root: true,
  parser: 'babel-eslint',
  parserOptions: {
    sourceType: 'module'
  },
  env: {
    browser: true,
  },
  extends: 'standard',
  // required to lint *.vue files
  plugins: [
    'html'
  ],
  // add your custom rules here
  'rules': {
    // allow paren-less arrow functions
    'arrow-parens': 0,
    // allow async-await
    'generator-star-spacing': 0,
    // allow debugger during development
    'no-debugger': process.env.NODE_ENV === 'production' ? 2 : 0
  }
}
```

新建一个 utils.js：

```javascript
// utils.js

var path = require('path')
var config = require('../config')
// 导出 css 的相关 plugin
var ExtractTextPlugin = require('extract-text-webpack-plugin')

exports.assetsPath = function (_path) {
  var assetsSubDirectory = process.env.NODE_ENV === 'production'
    ? config.build.assetsSubDirectory
    : config.dev.assetsPublicPath
  return path.posix.join(assetsSubDirectory, _path)
}

exports.cssLoaders = function (options) {
  options = options || {}

  var cssLoader = {
    loader: 'css-loader',
    options: {
      minimize: process.env.NODE_ENV === 'production',
      sourceMap: options.sourceMap
    }
  }

  function generateLoaders (loader, loaderOptions) {
    var loaders = [cssLoader]
    if (loader) {
      loaders.push({
        loader: loader + '-loader',
        options: Object.assign({}, loaderOptions, {
          sourceMap: options.sourceMap
        })
      })
    }

    if (options.extract) {
      return ExtractTextPlugin.extract({
        use: loaders,
        fallback: 'style-loader'
      })
    } else {
      return ['style-loader'].concat(loaders)
    }
  }

  return {
    css: generateLoaders(),
    less: generateLoaders('less'),
    sass: generateLoaders('sass', { indentedSyntax: true}),
    scss: generateLoaders('sass')
  }
}

exports.styleLoaders = function (options) {
  var output = []
  var loaders = exports.cssLoaders(options)
  for (var extension in loaders) {
    var loader = loaders[extension]
    output.push({
      test: new RegExp('\\.' + extension + '$'),
      use: loader
    })
  }
  return output
}
```

继续改造 **webpack.base.conf.js** ：

```javascript
// webpack.base.conf.js

var path = require('path')
var utils = require('./utils')
var config = require('../config')

// 省略相关代码

module.exports = {
  // 省略相关代码
  module: {
    rules: [
      // 省略相关代码
      {
        test: /\.(png|jpe?g|git|svg)(\?.*)?$/,
        use: {
          loader: 'url-loader',
          options: {
            // 如果超过大小限制，则不进行编码，输出到/img目录
            limit: 10000,
            name: utils.assetsPath('/img/[name].[hash:7].[ext]')
          }
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
      // 省略相关代码
    ]
  }
}
```

至此完成了 **webpack.base.conf.js** 的改造，通过封装工具函数、独立路径配置来达成高可用配置的效果。接下来完成 **dev** 环境的配置，我们新建一个 **webpack.dev.conf.js** 文件：

```javascript
// webpack.dev.conf.js

var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var HtmlWebpackPlugin = require('html-webpack-plugin')
// webpack 编译错误格式化为比较友好的格式的插件
var FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')

// 添加 webpack-hot-middleware 客户端
Object.keys(baseWebpackConfig.entry).forEach(function (name) {
  baseWebpackConfig.entry[name] = ['./build/dev-client'].concat(baseWebpackConfig.entry[name])
})

// 合并 base 配置
module.exports = merge(baseWebpackConfig, {
  module: {
    // 添加 css loaders
    rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap })
  },
  devtool: '#cheap-module-eval-source-map',
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.dev.env
    }),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoEmitOnErrorsPlugin(),
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    }),
    new FriendlyErrorsPlugin()
  ]
})
```

接下来修改 **dev-server.js** ：

```javascript
// dev-server.js

var config = require('../config')
if (!process.env.NODE_ENV) {
  process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)
}

// 用户自动打开浏览器
var opn = require('opn')
var path = require('path')
var express = require('express')
var webpack = require('webpack')
// 用户设置代理请求
var proxyMiddleware = require('http-proxy-middleware')
var webpackConfig = require('./webpack.dev.conf')
// 端口号
var port = process.env.PORT || config.dev.port
// 是否自动打开浏览器
var autoOpenBrowser = !!config.dev.autoOpenBrowser
// 请求转发表
var proxyTable = config.dev.proxyTable

var app = express()
var compiler = webpack(webpackConfig)

var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  // 格式化错误提示
  quiet: true
})

var hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: () => {}
})

compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emit', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' })
    cb()
  })
})

Object.keys(proxyTable).forEach(function (context) {
  var options = proxyTable[context]
  if (typeof options === 'string') {
    options = { target: options }
  }
  app.use(proxyMiddleware(options.filter || context, options))
})

app.use(require('connect-history-api-fallback')())

app.use(devMiddleware)

app.use(hotMiddleware)

var staticPath = path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory)
app.use(staticPath, express.static('./static'))

var uri = 'http://localhost:' + port

var _resolve
var readyPromise = new Promise(resolve => {
  _resolve = resolve
})

console.log('> 启动开发服务器...')
devMiddleware.waitUntilValid(() => {
  console.log('> 监听端口' + uri + '\n')
  if (autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
    opn(uri)
  }
  _resolve()
})

var server = app.listen(port)

module.exports = {
  ready: readyPromise,
  close: () => {
    server.close()
  }
}
```

现在完成了 **dev** 环境的配置，执行 `yarn run dev` 来测试一下吧！

![1](/images/webpack-guide-3/1.png)

![2](/images/webpack-guide-3/2.png)

很完美！不是吗！有了上面的基础，接下来编写 **build** 配置就稍微省些力气了，新建 **webpack.prod.conf.js** ：

```javascript
// webpack.prod.conf.js

var path = require('path')
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var CopyWebpackPlugin = require('copy-webpack-plugin')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')

var env = config.build.env

var webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  devtool: config.build.productionSourceMap ? "#source-map" : false,
  output: {
    path: config.build.assetsRoot,
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': env
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      sourceMap: true
    }),
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    new OptimizeCSSPlugin({
      cssProcessorOptions: {
        safe: true
      }
    }),
    new HtmlWebpackPlugin({
      filename: config.build.index,
      template: 'index.html',
      inject: true,
      chunksSortMode: 'dependency'
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module, count) {
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['vendor']
    }),
    // copy custom static assets
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ]
})

if (config.build.productionGzip) {
  var CompressionWebpackPlugin = require('compression-webpack-plugin')

  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',
      algorithm: 'gzip',
      test: new RegExp(
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      threshold: 10240,
      minRatio: 0.8
    })
  )
}

if (config.build.bundleAnalyzerReport) {
  var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}

module.exports = webpackConfig
```

创建 **build.js** 文件：

```javascript
// build.js

process.env.NODE_ENV = 'production'

var ora = require('ora')
var rm = require('rimraf')
var path = require('path')
var chalk = require('chalk')
var webpack = require('webpack')
var config = require('../config')
var webpackConfig = require('./webpack.prod.conf')

var spinner = ora('正在构建生产环境')
spinner.start()

rm(path.join(config.build.assetsRoot, config.build.assetsSubDirectory), err => {
  if (err) throw err
  webpack(webpackConfig, function (err, stats) {
    spinner.stop()
    if (err) throw err
    process.stdout.write(stats.toString({
      colors: true,
      modules: false,
      children: false,
      chunks: false,
      chunkModules: false
    }) + '\n\n')

    console.log(chalk.cyan('  构建完成.\n'))
  })
})
```

安装相关的依赖：

```
yarn add babel-core babel-eslint babel-loader babel-preset-env connect-history-api-fallback copy-webpack-plugin eslint eslint-config-standard eslint-friendly-formatter eslint-loader eslint-plugin-html eslint-plugin-import eslint-plugin-node eslint-plugin-promise eslint-plugin-standard extract-text-webpack-plugin friendly-errors-webpack-plugin http-proxy-middleware less less-loader node-sass opn optimize-css-assets-webpack-plugin ora sass-loader webpack-merge
```

执行 `yarn run build` 试一下吧！

![3](/images/webpack-guide-3/3.png)

大功告成！
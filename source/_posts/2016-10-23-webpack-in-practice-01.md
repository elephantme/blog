---
layout: post
title:  "webpack实战"
date:   2016-10-23
categories: webpack
---

> 本次示例主要讲解了代码分割、文件hash及commonschunk插件的使用。
> [示例代码](https://github.com/elephantme/share/tree/master/webpack/code-split-example) 

<!--more-->

## 1. webpack-dev-server

> The *webpack-dev-server* is a little Node.js [Express](http://expressjs.com/) server, which uses the *webpack-dev-middleware* to serve a *webpack bundle*. 
> It uses webpack to compile assets in-memory and serve them.

### 1.1 Automatic Refresh Mode

1. **Iframe mode**
   1. 不需要加任何参数，默认就是该方式
   2. `http://«host»:«port»/webpack-dev-server/«path»` 来访问
   3. 顶部会显示一个状态条
   4. 应用url的改变不会反应到浏览器上
2. **Inline mode**
   1. 命令行添加该参数 --inline
   2. `http://«host»:«port»/«path»` 来访问
   3. 状态信息显示在浏览器的console中
   4. 应用url的改变会反应到浏览器上

我们一般用inline模式就可以了。

### 1.2 命令行参数

1. `--inline` inline模式
2. `--hot` HMR(Hot Module Replacement)
3. `--host` hostname or IP. 0.0.0.0 binds to all hosts
4. `--port` 端口
5. `--history-api-fallbac` h5 history api

## 2. [Sourcemap](http://webpack.github.io/docs/configuration.html#devtool) & debug

> 配置情况比较多，可以参考[这里](https://segmentfault.com/a/1190000004280859)
>
> 开发模式下，可以考虑 `eval` 的方式，构建速度快

## 3. CodeSplitting

> 将打包大文件可以进行代码分割，实现按需加载
>
> 减少首页加载时间
>
> 配合浏览器缓存机制进行合理代码分割

### 3.1 实现方式

1. 在需要代码分割的位置进行埋点（webpack的require.ensure api），如路由触发时，某个事件回调里。
2. 只需要上面一步就可以实现代码分割了，但是一般还要搭配一些插件来进行优化，例如将公共部分提取到单独模块中。


## 4. Demo

场景：两个按钮各自单击时加载各自模块（module01和module02）, 他们都依赖common.module，而module02又额外依赖common_another.module。

代码结构如下

```javascript
// module01
var common = require('../common/common.module');
module.exports = {
  name: 'module01' + common.name
};

// module02
var common = require('../common/common.module');
var another = require('../common/another-common');

module.exports = {
  name: 'module02' + common.name
};

// 代码分割处(index.js)
btn01.addEventListener('click', function(){
  require.ensure([], function(require){
    var module01 = require('./module01/module01');
    content01.innerHTML = module01.name;
    location.hash = '#/module01';
  });
});

btn02.addEventListener('click', function(){
  require.ensure([], function(require){
    var module02 = require('./module02/module02');
    content02.innerHTML = module02.name;
    location.hash = '#/module02';
  });
});
```

1. 先只配置代码分割点，不使用commonchunk插件

   运行webpack进行build，module01和module02都是单独的bundle，但是代码中都含有common.module的代码。

   `2065d110a1dba417087943f06023d4fa3eb195be` (commit id)

2. 配置[commonchunk](https://github.com/webpack/docs/wiki/list-of-plugins#commonschunkplugin)插件，在webpack.config.js中添加如下配置

   ```javascript
   plugins: [
     new webpack.optimize.CommonsChunkPlugin({
       children: true
     })
   ]
   ```

   该例子定义了两个代码分割点，将一个app.js分成了三分，分别多了1.app.js和2.app.js。其中app.js为1.app.js和2.app.js的父亲。children为true就表示将app.js孩子中公共模块都放到我（父亲）这里。所以结果就是common.module的代码挪到了app.js中。注意：这样会有副作用，会加重初始化加载时间。

   `50e17d9071287bc1e54e92a5dfa4448f27967cea`

3. 在2的基础上，再增加另外一个配置 `async: true`

   ```javascript
   plugins: [
     new webpack.optimize.CommonsChunkPlugin({
       children: true,
       async: true
     })
   ]
   ```

   这个参数导致的结果就是：common.module被单独抽离到一个文件，在需要它的时候会并行加载进来。

   `0c67e470dca2ef388c6f3514d58ed64fd62a3767`


**再做另外一个需求**，将第三方(vender)也单独提取出来，方便浏览器缓存。webpack.config.js中配置如下：

```javascript
module.exports = {
  entry: {
    app: './src/index.js',
    vender: ['jquery']
  },
  output: {
    path: './dist',
    filename: 'app.js'
  },
  // devtool: 'cheap-eval-source-map',
  
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vender',
      filename: 'vender.js'
    }),
    new webpack.optimize.CommonsChunkPlugin({
      children: true,
      async: true
    })
  ]
};
```

1. 会新增一个vender的entry。
2. 增加commonchunk配置，其中name指的就是bundle name，filename是build后输出名称。

这样构建后就会多一个vender.js的bundle。

`73af9cf79f1a58539817ad6469e908d14308872b`

**build文件添加hash**

这个比较简单，难的是怎么在index.html中引入，多亏有`html-webpack-plugin` ，webpack.config.js为：

```javascript
var webpack = require('webpack');
var HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: {
    app: './src/index.js',
    vender: ['jquery']
  },
  output: {
    path: './dist',
    filename: 'app.[chunkhash].js'
  },
  // devtool: 'cheap-eval-source-map',
  
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vender',
      filename: 'vender.[chunkhash].js'
    }),
    new webpack.optimize.CommonsChunkPlugin({
      children: true,
      async: true
    }),

    new HtmlWebpackPlugin({
      chunks: ['app', 'vender'],
      filename: 'index.html',
      template: './index.html',
      inject: 'body'
    })
  ]
};
```

1. []是webpack的占位符，常用的有name、chunkhash、hash
2. HtmlWebpackPlugin 配置可以参考其文档。

`9b6127888203040d5f02f209d42155f64e5c4da4`

## 5. Analyzing Build Statistics

1. [The official analyse tool](http://webpack.github.io/analyse/)
2. [Webpack Visualizer](https://chrisbateman.github.io/webpack-visualizer/)
3. [Webpack Chart](https://alexkuz.github.io/webpack-chart/)
4. [robertknight/webpack-bundle-size-analyzer](https://github.com/robertknight/webpack-bundle-size-analyzer)

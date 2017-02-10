---
layout: post
title:  "angular+webpack+oclazyload实现代码分割"
date:   2016-08-03
categories: angular, webpack
---

## 1. 为什么要代码分割

### 1.1 目的

使相关页面按需加载资源，尽可能地减少加载时间。例如在Angular+RequireJs构建的应用中，在应用列表中，只需要加载相关的template、controller、service等合并压缩后的文件，其它资源可以在进入相关页面时再去加载，这样加快该页面的访问速度，尤其该页面是首页！

<!--more-->

## 2. [oclazyload](https://oclazyload.readme.io)简单介绍

angular在你的app启动的时候，就需要指定所依赖的module。如果要动态加载module，就需要想办法注入到angular中，这是就是oclazyload所干的事情。

## 3. [webpack代码分割功能](http://webpack.github.io/docs/code-splitting.html)

需要做代码分割的地方要埋点，可以通过`require.ensure`(CommonJs)或者`require`(AMD)API来实现。

```javascript
$stateProvider.state('dataAnalysis', {
  url: '/data/ananlysis',
  template: require('./data/analysis.html'),
  controller: 'DataController',
  resolve: ['$q', '$ocLazyLoad', function($q, $ocLazyLoad){
    return $q((resolve, reject) => {
      require.ensure([], function(require){
        var mod = require('./data');
        $ocLazyLoad.load({name: mod.name});
        resolve(mod.controller);
      });
    });
  }]
});
```
该模块是用CommonJs语法写的，所以只定义了一个代码分割点，就是加载data模块。

## 4. [完整项目](https://github.com/elephantme/angular-webpack-demo)

### 4.1 该项目支持的功能

1. 支持ES6源文件编译为ES5；
1. 使用webpack做代码分割，使首页加载更快速；
2. 构建中加入gulp，更灵活；
3. 加入web服务器，并且支持浏览器自动刷新，提高开发效率；
4. 支持sourcemap，方便调试错误；
5. 文件压缩，文件名支持hash；

### 4.2 项目结构

```
├── src
│   ├── assets
│   │   └── index.html  
│   ├── data 
│   │   ├── analysis.html
│   │   ├── chart.directive.js
│   │   ├── controller.js
│   │   ├── index.js
│   │   └── service.js
│   ├── user 
│   │   ├── center.html
│   │   ├── controller.js
│   │   ├── index.js
│   │   └── service.js
│   └── index.js
├── gulpfile.js
├── package.json
├── readme.md
└── webpack.config.js
```

data和user分别是不同的模块，可以通过require('./data')来直接引用。该项目最终会打包到dist文件夹下，运行后首页只是菜单导航，此时会加载angular相关包，及router的定义。当点击某个菜单时才去动态的加载该模块所依赖的文件，例如data模块他依赖了重量级的highcharts库，首页加载时就不需要加载他。

### 4.3 webpack配置文件

```javascript
var webpack = require('webpack');
var path = require('path');
var HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: {
    app: './src/index.js',
    vendor: ['angular', 'angular-ui-router', 'oclazyload']
  },
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: "[name].min.[chunkhash:8].js"
  },
  devtool: 'source-map',
  module: {
    loaders: [
      {
        test: /\.html$/,
        loader: 'html',
        query:{
          minimize: false
        }
      },
      {
        test: /\.js$/,
        loader: 'babel?presets[]=es2015',
        exclude: /node_modules/
      }
    ]
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin('vendor', '[name].min.[chunkhash:8].js'),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      mangle: {
        except: ['$q', '$ocLazyLoad'] //不加该参数，压缩js后会有错误
      }
    }),
    new HtmlWebpackPlugin({
      template: './src/assets/index.html',
      inject: 'body'
    })
  ]
};
```

1. `entry`定义了程序的入口，其中的两个属性app和vendor，就对应了两个chunk。
2. `output`定义了打包后输入的位置，其中`name`就为entry属性名，`chunkhash`为该chunk文件的hash值。
3. [`devtool`](http://webpack.github.io/docs/configuration.html#devtool) 指定sourcemap配置，用来调试程序。
4. `module`在这里可以配置loader。由于webpack可以加载一切静态资源，所以需要各种loader真正的来去加载解析编译这些资源。
5. `plugins`这里可以配置一些插件。资源输出时的一些处理，可以通过插件来完成，如js的压缩。

### 4.4 gulp配置文件

```javascript
var gulp = require('gulp');
var webpack = require('webpack-stream');
var clean = require('gulp-clean');
var browserSync = require('browser-sync').create();

// 清理dist目录
gulp.task('clean', function(){
  return gulp.src('./dist').pipe(clean());
});

// 启动web服务
gulp.task('webserver', ['webpack'], function() {
  browserSync.init({
    server: "./dist"
  });
});

// 使用webpack处理src中资源到dist目录下
gulp.task('webpack', function() {
  return gulp.src('src/index.js')
    .pipe(webpack(require('./webpack.config.js')))
    .pipe(gulp.dest('dist/'));
});

// 监听src中资源，一旦发生变动，则调用webpack打包资源并让浏览器自动刷新
gulp.task('watch', function () {
  gulp.watch('src/**/*', ['browserRefresh']);
});

// 确保webpack打包完成后执行
gulp.task('browserRefresh', ['webpack'], function(done){
  browserSync.reload();
  done();
});

// 开发环境任务
gulp.task('dev', ['clean'], function(){
  gulp.start('webserver', 'watch');
});

```

运行`gulp dev`命令构建成功后，会自动打开浏览器。如果文件做修改保存后，会自动部署并且浏览器自动更新。



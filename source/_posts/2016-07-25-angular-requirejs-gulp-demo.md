---
layout: post
title:  "使用Angular、RequireJs、Gulp构建单页面应用"
date:   2016-07-25 
categories: angular
tags: [angular, gulp]
---

## 1. 项目结构 
![angular项目结构](/images/angular-requirejs-demo.png)

<!--more-->

<a href="https://github.com/elephantme/angular-requirejs-demo" target="_blank">该项目可前往github上查看</a>

如果项目业务逻辑比较多，还是建议使用功能来划分，如user文件下会有controller、service、factory等。这样在编码时不需要到处找文件，后期维护也较容易一些。

## 2. 结构讲解

下面是程序的入口main.html

```html
<!DOCTYPE html>
<html>
<head>
  <title>Angular RequireJs Demo</title>
  <!-- build:css css/main.css -->
  <link rel="stylesheet" href="css/a.css">
  <link rel="stylesheet" href="css/b.css">
  <!-- endbuild -->
</head>
<body>
  <div>Angular+RequireJs+Gulp Demo</div>
  <header>
    <ul>
      <li><a href="#/view1">view1</a></li>
      <li><a href="#/view2">view2</a></li>
    </ul>
  </header>
  <div class="container" ng-view>
    
  </div>
  <script src="bower_components/requirejs/require.js" data-main="src/main"></script>
</body>
</html>
```

其中script标签中的data-main指定了RequireJs先要执行的文件，也就是src/main.js

```javascript
require.config({
  paths: {
    'angular': '../bower_components/angular/angular',
    'angular-route': '../bower_components/angular-route/angular-route'
  },
  shim: {
    'angular': {
      exports: 'angular'
    },
    'angular-route': {
      deps: ['angular']
    }
  },
  deps: ['./bootstrap']
});
```

这里是RequireJs的一些配置，其中paths就是定义别名。shim是处理那些不是AMD规范的包。deps是要执行的模块，即bootstrap.js。

```javascript
define([
  'require', 
  'angular',
  'app',
  'route'
], function(require, angular){
  angular.bootstrap(document, ['elephant']);
});
```

该模块需要依赖angular、app、route这些模块，他们都加载好以后，angular启动名为elephant的app，该app的作用域是整个document。

到这里我们知道必须要创建一个名为elephant的module，他是怎么和项目中的controller、service等建立起联系的呢？请看`angular.module('moduleName',[])`，该api中第二个参数的意思就是该module依赖的其他module，所以我们可以把所有的controller放到一个叫app.controllers的module中，service放到一个叫app.services的模块中，将他们依赖到elephant这个模块就可以了。 下面看一下app.js：

```javascript
define([
  'angular',
  'angular-route',
  'controller/all',
  'service/all'
], function(
  angular
){
  var app = angular.module('elephant', [
    'app.controllers',
    'app.services',
    'ngRoute'
  ]);

  return app;
});
```
可以看出该模块依赖controller/all、service/all，他们分别是angular单独的module——app.controllers和app.services，看一下service/all.js就明白了

```javascript
define(
  ['angular', './bookService', './userService'], 
  function(angular, bookService, userService){
    var all = angular.module('app.services', []);

    all.service({
      'BookService': bookService,
      'UserService': userService
    });
    
  }
)
```

通过这种方式，也可以将filter、directive等依赖到elephant中。

下面看一下route，本例中使用angular自带的angular-route插件，他需要单独下载。另外社区中ui-router也非常火热。

```javascript
define(['app'], function(app){
  app.config(['$routeProvider', function($routeProvider){
    $routeProvider.when('/view1', {
      templateUrl: 'template/view1.html',
      controller: 'Ctrl01'
    });

    $routeProvider.when('/view2', {
      templateUrl: 'template/view2.html',
      controller: 'Ctrl02'
    });
    
  }]);
});
```

他只依赖app，给app增加config来实现路由功能的，就是将我们的路由配置告诉$routeProvider这个provider。

到这里整个项目架构已完成，下面介绍下gulp这款构建工具。

## 3. gulp脚本

目前gulp已经完全取代了grunt，说明他有一定的优势。个人觉得gulp的优势有：

1. 代码优于配置 grunt是基于配置文件，而gulp是直接编码。Spring走的就是摒弃复杂的配置文件回归代码的路线。
2. 直观，gulp的任务更紧凑；而grunt只是配置了一个个子任务，需要看registerTask中任务的顺序才能明白要做的事情。

下面看一下gulpfile脚本吧

```javascript
/**
* 1.将css文件合并压缩后放到dist/css下
* 2.将template文件压缩后放到dist/template下
* 3.将src下js文件合并压缩后放到dist/js下
* 4.将main.html文件使用cssmin，jsmin替换后，并压缩，然后放到dist下
* 5.最后别忘了将requirejs源文件移过去
*/

var gulp = require('gulp'),
  concat = require('gulp-concat'),
  cleanCss = require('gulp-clean-css'),
  htmlmin = require('gulp-htmlmin'),
  uglify = require("gulp-uglify"),
  clean = require('gulp-clean'),
  usemin = require('gulp-usemin'),
  replace = require('gulp-regex-replace'),
  requirejsOptimize = require('gulp-requirejs-optimize');

gulp.task('clean', function(){
  return gulp.src('./dist')
    .pipe(clean());
});

// 执行第一步
gulp.task('css', function(){
  return gulp.src('./css/*.css')
    .pipe(concat('main.css'))
    .pipe(cleanCss())
    .pipe(gulp.dest('./dist/css/'));
});

// 执行第二步
gulp.task('template', function(){
  return gulp.src('./template/*.html')
    .pipe(htmlmin({collapseWhitespace: true}))
    .pipe(gulp.dest('./dist/template/'));
});

// 合并压缩js文件
gulp.task('js', function(){
  return gulp.src('./src/main.js')
    .pipe(requirejsOptimize(function(){
      return {
        mainConfigFile: './src/main.js'
      }
    }))
    .pipe(uglify())
    .pipe(concat("main.min.js"))
    .pipe(gulp.dest('./dist/js'));
});

// 处理main.html
gulp.task('main', function(){
  return gulp.src('./main.html')
    .pipe(usemin())
    .pipe(replace({
      regex: 'src="bower_components/requirejs/require.js" data-main="src/main"',
      replace: 'src="js/require.js" data-main="js/main.min"'
    }))
    .pipe(htmlmin())
    .pipe(gulp.dest('./dist'))
});

// 处理requirejs
gulp.task('requirejs', function(){
  return gulp.src('./bower_components/requirejs/require.js')
    .pipe(uglify())
    .pipe(gulp.dest('./dist/js'))
});

gulp.task('default', ['clean'], function(){
  gulp.start('css', 'template', 'js', 'main', 'requirejs');
});

```

gulp中src是输入，然后pipe可以接各种处理脚本，最后dest是输出。是不是很类似shell脚本，如：`grep src | clean | concat > dest`。

每一个task都是集中处理一个任务，通过gulp.start()来指定任务执行的顺序。可以看看grunt任务的配置，很明显没有gulp直观。

```javascript
grunt.registerTask('prod',['clean', 'copy', 'usemin', 'regex-replace', 'htmlmin', 'requirejs']);
```





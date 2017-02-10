---
layout: post
title:  "初识webpack"
date:   2016-10-22
categories: webpack
---

![webpack](/images/webpack.png)

<!--more-->

## 1. Features

1. Hot Module Replacement
2. Bundle Splitting
3. Asset Hashing
4. Loaders and Plugins

## 2. Hello World 示例

> 主要目的是搭建webpack环境，以输出helloworld为例

1. mkdir helloworld
2. helloworld下运行npm init，按照提示回车下去
3. npm install --save-dev webpack
4. 在根路径下增加webpack.config.js,这是webpack默认配置文件。

   ```javascript
   module.exports = {
    entry: {
      app: './src/index.js',
    },
    output: {
      path: './dist',
      filename: '[name].js'
    }
   };
   ```
5. 根路径下运行webpack，然后ll，可以看到多了一个dist文件夹。
6. 最后运行node dist/app.js，会输出hello world。

## 3. webpack.config 文件结构

```javascript
{
  context: '', // 配置basePath
  entry: {}, // 依赖遍历入口文件
  output: { // 打包输出配置
        path: '', 
        filename: ''
  }, 
  resolve: { // 用来配置路径
    alias: {}, // 模块别名
    root: [] // 查找模块时从这里开始，配置绝对路径
  },
  module: { // 最关键的配置，加载器
    loaders: []
  },
  plugins: [] // 插件
}
```

## 4. [Loader](http://webpack.github.io/docs/using-loaders.html)

> loader是处理、加工源码的加载器。从源码的最初加载到最终输出可以已管道的方式配置多个loader，类似gulp的流式处理。

### 4.1 用法

1. 显示地在 `require` 中添加。

   > 在require语句中（或者 define, require.ensure等）你可以指定装载机。只需要用`！`将资源和loader分开。每一部分会相对于当前文件夹来resolve。

   ```javascript
   // 使用当前路径下loader.js来装载dir文件夹下的file.txt
   require("./loader!./dir/file.txt");

   // 使用jade-loader来装载template.jade
   require("jade!./template.jade");

   // 先使用less-loader来装载bootstrap/less文件夹下的bootstrap.less，然后将处理结果再由css-loader来处理，其结果会交给style-loader,最终输出
   require("!style!css!less!bootstrap/less/bootstrap.less");
   ```

2. 在配置文件中 配置。

   ```javascript
   {
       module: {
           loaders: [
               { test: /\.jade$/, loader: "jade" },
               { test: /\.css$/, loader: "style!css" },
               // 还可以这么写
               { test: /\.css$/, loaders: ["style", "css"] },
           ]
       }
   }
   ```

3. 在命令行配置。

   ```shell
   webpack --module-bind jade --module-bind 'css=style!css'
   ```

**参数**

loader可以传入query字符作为参数（像浏览器中那样），query字符串跟在 Loader后面。例如 `url-loader?mimetype=image/png`。

**[list webpack loaders](https://webpack.github.io/docs/list-of-loaders.html)**

## 5. [Plugin](http://webpack.github.io/docs/using-plugins.html)

> 在webpack中通常使用插件添加与 bundles相关的功能。例如使用 `extract-text-webpack-plugin` 插件可以将style-loader输出的bundle从html文件抽取到单独的css文件中。

**常用的plugin**

1. [html-webpack-plugin](https://github.com/ampedandwired/html-webpack-plugin) 单页应用程序首页可以用它来动态生成，会自动引入程序入口js文件。
2. [extract-text-webpack-plugin](https://github.com/webpack/extract-text-webpack-plugin/blob/webpack-1/README.md) 将css提取到单独文件
3. [copy-webpack-plugin](https://github.com/kevlened/copy-webpack-plugin) 复制源文件到构建路径下
4. [string-replace-webpack-plugin](https://www.npmjs.com/package/string-replace-webpack-plugin) 正则替换
5. CommonsChunkPlugin webpack内置插件，将公共文件抽取出来
6. [webpack-svgstore-plugin](https://www.npmjs.com/package/webpack-svgstore-plugin) svg 合并
7. UglifyJsPlugin js压缩，webpack内置
8. [purifycss-webpack-plugin](https://www.npmjs.com/package/purifycss-webpack-plugin) Remove unused CSS. Also works with single-page apps



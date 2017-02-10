---
layout: post
title:  使用markdown自动生成Restfull风格的文档
date:   2017-02-10
---

## 1. markdown

[markdown](http://wowubuntu.com/markdown/) 语法超级简单，分分钟就看会了。特别适合程序员写文档，写blog。目前有好多静态博客都采用markdown，如[jekyll](http://jekyllcn.com/)、[Hexo](https://hexo.io)。

<!--more-->

## 2. slate & mkdocs

### 2.1 [slate](https://github.com/lord/slate)

这个项目已有1W多star，基于ruby开发，特点是界面特别专业和美观，[点击这里](https://github.com/lord/slate/wiki/Slate-in-the-Wild) 可以看到有好多OpenApi示例，都是使用slate来搭建的。原本打算用它来着，[结果对中文不支持](https://github.com/lord/slate/issues/573)，非常遗憾。

### 2.2 [mkdocs](http://www.mkdocs.org/)

这个环境搭建很简单，下面会介绍。官方文档介绍也比较详细，也有[Material](https://material.io/guidelines/)风格的主题，最终选择了它。

## 3. mkdocs环境搭建

### 3.1 Python & pip

pip是Python的包管理器，类似于nodejs中的npm、ruby的gem。
安装都比较简单，windows下需要配置环境变量。

### 3.2 mkdocs

1. pip install mkdocs
2. pip install mkdocs-material
	[mkdocs-material](https://github.com/squidfunk/mkdocs-material) 是一款主题

### 3.3 启动

在docs项目根路径下运行`mkdocs serve`命令，mkdocs内置server会将markdown编译成html并部署到8000端口，直接用http://localhost:8000访问就可以了。


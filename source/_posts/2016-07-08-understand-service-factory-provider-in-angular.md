---
layout: post
title:  理解angular中的service、factory、provider
date:   2016-07-08 
categories: angular
tags: angular
---

> angular的依赖注入特性确实很强大，其核心就是$injector，他注入的所有服务，底层都是通过provider来提供的。而所谓的Service、Factory、Value、Constant只是provider封装出来的API方法而已，这些方法又大都暴露给angular.module，所以才会出现我们常见的module.service等方法。

<!--more-->

$provide常见的API有：

```javascript
//provider提供两种传参方式，object和function。前者必须包括$get方法，后者通过$injector.instantiate()得到一个实例对象。
provider(name, provider);

//它是$provide.provider(name, {$get: $getFn})的简写
factory(name, $getFn);

//内部实现类似这样$provide.provider(name, {$get: $injector.instantiate(constructor)})
service(name, constructor);

//它是$provide.provider(name, {$get: value})的简写
//他不可以注入到config中，但是他的值是可以被修改的。
value(name, value);

//他可以注入到config中，但是他的值是不可以被修改的。
constant(name, value);

//没错，和设计模式中的装饰者模式的概念一样。后面再详细说吧
decorator(name, decorator);
```

下面举例来说明一下

## 1. Value

```javascript
var app = angular.module('app', []);
app.provider('value', {
  $get:function(){
    return 'this is a string value'
  }
});
//等价于
app.value('value', 'this is a string value');
```

## 2. Factory

```javascript
var app = angular.module('app', []);
app.provider('factory', {
  $get: function(){
    return {
      sayHello: function(){
        console.log('hello')
      }
    }
  }
});
//等价于
app.factory('factory', [function () {
  return {
    sayHello: function(){
      console.log('hello')
    }
  };
}])
```

## 3. Service

angular中的service有两个特性：1)延迟实例化 2)单例。另外Service与Factory注入的方式不一样，Service注入时需要创建对象实例。而Factory直接注入的就是一个对象实例。

```javascript
function User() {
  this.sayHello = function(){
    console.log('hello')
  }
}
var app = angular.module('app', []);
app.provider('service', function () {
  this.$get = [function() {
    return new User();
  }];
});
//等价于
app.service('service', User);
```

另外angular中的$http,$log,$filter等服务都提供了对应的provider，在注入这些服务之前，可以通过config来修改他们的配置。例如给$httpProvider.interceptors添加拦截器。




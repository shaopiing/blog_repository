---
title: "AngularJS 的常用特性 (四)"
date: 2016-09-14 23:21:10
category: 前端
tags: [Program,前端]
---
AngularJS常用特性
<!--more-->
**11、使用 Module(模块) 组织以来关系**

Angular 里面的模板，提供了一种方法，可以用来组织应用中一块功能区域的依赖关系；同时还提供了一种机制，可以自动解析依赖关系（又叫依赖注入），一般来说，我们把这些叫做依赖服务，因为它们会负责为应用提供特殊的服务。

例如，如果购物站点中的一个控制器需要从服务器中获取一个商品列表 Items，来处理从服务器获取商品的工作。进而，Items 对象就需要以某种方式与服务器上的数据库进行交互，可以通过 XHR 或者 WebSocket。

如果，在没有模块的情况下，那么代码看起来就像下面这样：

```javascript
function ItemsViewController($scope) {

    // 向服务器发起请求

    ...

    // 解析响应并放入 Item 对象

    ...

    // 把 Items 数组设置到 $scope 上，这样视图才能够显示它

    ...

}
```

虽然这么做可以运行，但是存在一些潜在的问题：

- 如果其他控制器也需要从服务器获取 Items，那么我们只能把这段代码重写一遍，代码维护工作增大难度；
- 加上其他的一些因素，例如服务器认证、解析数据的复杂性，会使控制器对象很难划分出一个合理的功能边界，并且代码阅读起来更加困难；
- 增加单元测试的难度，要测试这段代码，必须启动一个真正的服务器。

利用模块和模块内置的依赖注入功能，我们可以把控制器写得更加简单，示例如下：

```javascript
function ShoppingController($scope, Items) {

    $scope.items = Items.query();

}
```

这样会很简单而且方便，只需要把 Items 对象定义成了一个**服务**

服务都是单例的对象，Angular 内置了很多服务，例如 `$location` 服务，用来和浏览器的地址栏进行交互；`$route` 服务，用来根据 URL 地址的变化切换视图；还有 `$http` 服务，用来和服务器进行交互。

你也可以自定义自己的服务，但最好不要以 **$** 开头，因为 Angular 内置的服务是以 **$** 符号开头的，以免引起冲突。

你可以使用模型对象的 API 来定义服务，如下:

```
provider(name, Object OR constructor()) - 一个可配置的服务，如果你传递了一个 Object 作为参数，那么该对象必须带有一个名为 $get 的函数，该函数需要返回服务的名称。否则，认为你传递的是一个构造函数，调用构造函数会返回服务实例对象。

factory(name, $getFunction()) - 一个不可配置的服务，需要你指定一个函数，当调用这个函数的时候，会返回服务的实例。可以用把它看成 provider (name, { $get: $getFunction()}) 的形式

service(name, constructor()) - 一个不可配置的服务，与上面 provider 函数的 constructor 参数类似，调用它可以创建服务实例

```

下面用 factory() 的方式编写服务：

```javascript
// 创建一个模型用来支撑我们的购物视图

var shoppingModule = angular.module('ShoppingModule', []);

// 设置好服务工厂，用来创建我们的 Items 接口，以便访问服务端数据库

shoppingModule.factory('Items', function () {

    var items = {};

    items.query = function () {

        return [

            {title: 'Paint', description: 'Pots full of paint', price: 3.95},

            {title: 'Polka', description: 'Dots with polka', price: 2.95},

            {title: 'Pebbles', description: 'Just little rocks', price: 6.95}

        ];

    };

    return items;

});
```

控制器就很简单了，但是需要注入 Items 对象：

```javascript
shoppingModule.controller('ShoppingController', function ($scope, Items) {

     $scope.items = Items.query();

});
```

页面可以写个很简单的如下：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-4-1.png)

效果如下：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-4-2.png)

**12、使用过滤器格式化数据**

你可以用过滤器来声明应该如何变换数据格式，然后再显示给用户，只要在模板中使用一个插值变量即可，过滤器的语法如下：

```
{{ expression | filterName : parameter1 : ... parameterN}}
```

Angular 的内置过滤器，比如 currency：

```
{{12.9 | currency}}
```

显示结果为：

```
$12.90
```

还有其他过滤器，比如 date、number、uppercase 等，可以连用：

```
{{12.9 | currency | number : 0}}
```

显示结果为：

```
$13
```

当然，我们经常需要自定义过滤器，比如我们需要为标题文字创建首字母大写的字符串，编写过滤器如下：

```javascript
var homeModule = angular.module('HomeModule', []);

homeModule.filter('titleCase', function () {

    return function (input) {

        var words = input.split(' ');

        for (var i = 0; i < words.length; i++) {

            words[i] = words[i].charAt(0).toUpperCase() + words[i].slice(1);

        }

        return words.join(' ');

    };

});

homeModule.controller('HomeController', function ($scope) {

    $scope.pageHeading = 'behold the majesty of your page title';

});
```

简单的模板页面：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-4-3.png)

显示结果如下：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-4-4.png)
---
title: "AngularJS 的常用特性 (三)"
date: 2016-08-04 23:21:10
category: 前端
tags: [Program,前端]
---
AngularJS常用特性
<!--more-->
**6、表达式**

在模板中使用表达式是为了以充分的灵活性在模板、业务逻辑和数据之间建立联系，同时又能避免让业务逻辑渗透到模板中。

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-3-1.png)

当然，对于第一个表达式，recompute() / 10，应该避免这种把业务逻辑放到模板中的方式，应该清晰地区分视图和控制器之间的职责，这样更便于测试。

虽然 Angular 里面的表达式比 Javascript 更严格，但是它们对 undefined 和 null 的**容错性**更好。如果遇到错误，模板只是简单地什么都不显示，而不会抛出一个 NullPointerException 错误，这样你就可以安全地使用未经初始化的模型值，而一旦它们被赋值以后就会立即显示出来。

**7、区分 UI 和控制器的职责**

在应用中控制器有三种职责：

- 为应用中的模型设置初始状态
- 通过 `$scope` 对象把数据模型和函数暴露给视图（UI 模版）
- 监视模型其余部分的变化，并采取相应的动作

建议为视图中的每一块功能区创建一个控制器，这样可以让控制器保持小巧和可管理的状态。

更为复杂的时候，你可以创建嵌套的控制器，通过内部的原型继承机制，父控制器对象上的 `$scope` 会被传递给内部嵌套控制器的 `$scope`

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-3-2.png)

上面的例子，ChildController 的 `$scope` 对象可以访问 ParentController 的 `$scope`对象上的所有属性（和函数）

**8、使用 `$watch` 监控数据模型的变化**

在 scope 内置的所有函数中，用到最多的可能就是 `$watch` 函数了，当你的数据模型中的某一部分发生变化时，`$watch` 可以向你发出通知，监控单个对象的属性，也可以监控需要经过计算的结果，只要能被当作属性访问到，或者可以当作一个 JavaScript 函数被计算出来，就可以被 `$watch` 函数监控。函数签名为：

　　`$watch(watchFn, watchAction, deepWatch)`

　　watchFn —— 该参数是一个带有 Angular 表达式或者函数的字符串，返回被监控的数据模型的当前值。它会被执行多次，所以要保证不会产生副作用；

　　watchAction —— 通常以函数的形式，接收到 watchFn 的新旧两个值，以及作用域对象的引用，函数签名为 function(newValue, oldValue, scope)

　　deepWatch —— 如果设置为 true，则会去检查被监控对象的每个属性是否发生了变化，一般监控一个数组或者多个对象时要设置为 true，但由于要遍历数组，所以运算负担会比较重。

 　  `$watch` 函数会返回一个属性，当你不再需要接受变更通知时，可以用这个返回的函数注销监控器。如果需要监控一个属性，然后注销监控，可以如下：

```javascript
...

var dereg = scope.watch('someModel.someProperty', callbackOnChange());

...

dereg();
```

一个购物车的例子，当用户添加到购物车中的商品价格超过 100 美元的时候，会有 10 美元的折扣。使用下面的模板：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-3-3.png)

控制器如下：

```javascript
var shoppingCart = angular.module('shoppingCart', []);

shoppingCart.controller('CartController', function ($scope) {

    $scope.bill = {};

    $scope.items = [
        {title: 'Paint pots', quantity: 8, price: 3.95},
        {title: 'Polka dots', quantity: 17, price: 12.95},
        {title: 'Pebbles', quantity: 5, price: 6.95}
    ];
    
    $scope.totalCart = function () {
        var total = 0;
        for (var i = 0, len = $scope.items.length; i < len; i++) {
            total = total + $scope.items[i].price * $scope.items[i].quantity;
        }
        return total;
    };
    
    function calculateDiscount(newValue, oldValue, scoope) {
        $scope.bill.discount = newValue > 100 ? 10 : 0;
    }
    
    $scope.subtotal = function () {
        return $scope.totalCart() - $scope.bill.discount;
    };
    
    $scope.$watch($scope.totalCart, calculateDiscount);

});
```

用户看到的效果如图：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-3-4.png)

**9、watch() 中的性能注意事项**

断点调试 totalCart() 的代码，你会发现渲染这个页面时，该函数被调用了 6 次。其中 3 次发生在每次调用它的时候：

```
- 模板 {{totalCart() | currency}}
- subtotal() 函数
- $watch() 函数 
```

然后 Angular 又把整个过程重复了一遍，这样的目的是：检测模型中的变更已经被完整地进行了传播，并且模型已经被设置好。Angular 的做法是，把所有被监控的属性都拷贝一份，然后把它们和当前的值进行比较，看看是否发生了变化。实际上，Angular 可能运行上面过程不止两遍，如果重复 10 遍发现属性还在变化，Angular 会报错并退出，这时候你需要解决循环依赖的问题了。

PS: 书里面说的是，由于 Angular 需要用 JavaScript 实现数据绑定，官方开发团队与 TC39 团队一起开发了一个叫做 `Object.observe()` 的底层本地化实现，这样可以让你的数据绑定操作就像本地化代码一样快速。

我们可以改成监控 items 数组的变化，然后重新计算 $scope 属性中的总价、折扣和小计值来减少函数调用次数。

模板修改如下：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-3-5.png)

控制器修改如下：

```javascript
var shoppingCart = angular.module('shoppingCart', []);

shoppingCart.controller('CartController', function ($scope) {

    $scope.bill = {};

    $scope.items = [
        {title: 'Paint pots', quantity: 8, price: 3.95},
        {title: 'Polka dots', quantity: 17, price: 12.95},
        {title: 'Pebbles', quantity: 5, price: 6.95}
    ];
    
    function calculateDiscount(newValue, oldValue, scoope) {
        var total = 0;
        for (var i = 0, len = $scope.items.length; i < len; i++) {
            total = total + $scope.items[i].price * $scope.items[i].quantity;
        }
        $scope.bill.totalCart = total;
        $scope.bill.discount = newValue > 100 ? 10 : 0;
        $scope.bill.subtotal = total - $scope.bill.discount;
    }
    
    $scope.$watch('items', calculateDiscount, true);

});
```



在调用 $watch 函数时把 items 写成了一个字符串，并且第三个参数设置为 true，可以监控整个 items 数组的变化，单需要制作一份数组的拷贝，用来进行比较操作。

所以，如果每次在 Angular 显示页面时只需要重新计算 bill 属性，那么性能会好很多。可以像下面这样重新计算属性值：

```javascript
scope.watch(function (newValue, oldValue, scope) {

    var total = 0;

    for (var i = 0, len = $scope.items.length; i < len; i++) {

        total = total + scope.items[i].price * scope.items[i].quantity;

    }

    $scope.bill.totalCart = total;

    $scope.bill.discount = newValue > 100 ? 10 : 0;

    scope.bill.subtotal = total - scope.bill.discount;

});
```

根据性能分析更是说明了这点（左边是第二种方式）：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-3-6.png)

当然可以使用上面说过的方法，利用 `$watch` 返回的函数，移除不必要的 `$watch`，参见破狼大神的例子：[Angular 移除不必要的 $watch](https://www.cnblogs.com/whitewolf/p/angularjs-remove-unused-watch.html)

**10、监控多个东西**

监控多个东西，有两种基本的选择：

- 监控把这些属性连接起来之后的值
- 把它们放到一个数组或者对象中，然后给 deepWatch 参数传递一个 true 值。


第二种情况上面已经介绍，第一种情况比较简单，比如在你的作用域中存在一个 things 对象，它带有两个属性 a 和 b，当这两个属性发生变化时都需要执行 callMe() 函数，你可以同时监控着两个属性， 示例如下：

```
scope.watch('things.a + things.b', callMe(...));
```

当然，a 和 b 也可以属于不同的对象，这个列表可以很长，不过如果需要监控的属性比较多， 不妨把这个列表放入一个函数中，返回连接的值。
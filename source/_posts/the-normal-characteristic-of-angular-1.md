---
title: "AngularJS 的常用特性 (一)"
date: 2016-06-15 00:05:10
category: 前端
tags: [Program,前端]
---
前言：AngularJS 是一款来自 Google 的前端 JS 框架，该框架已经被应用到了 Google 的多款产品中，这款框架最核心特性有：MVC、模块化、自动化双向数据绑定、语义化标签、依赖注入等待。而且，AngularJS 框架自身是通过 TDD（测试驱动）的方式开发的，从这个角度来看，AngularJS 是敏捷开发的一次成功实践。

引一段《AngularJS 深度剖析和最佳实践》中的话：

> Angular 的学习曲线大概是这样的：入门非常容易，中级的时候会发现需要深入理解很多概念，高级的时候需要掌握 Angular 的工作原理，而想成为专家则很难，需要经过很多工程实践的磨练。

既然入门比较容易，那就一起入门吧。

**1、调用 Angular**

为了使用 Angular，所有应用都必须首先做两件事情：

- 加载 angular.js 库
- 使用 ng-app 指令告诉 Angular 应该管理 DOM 中的哪一部分。

如果你正在构建一款纯 Angular 应用，则把 ng-app 指令写在 `<html>` 标签中，示例如下：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-1-1.png)

或者你也可以用 Angular 管理页面中的一部分，只有放在页面中的 `<div>` 之类的元素即可。

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-1-2.png)

**2、模板与数据绑定**

1）**数据绑定**是 Angular 的特色，即 “所见即所得”，像这样设置其中的文本：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-1-3.png)

也叫做双花括号插值语法，因为它可以把新的内容插入到现有的模板中。

控制器 (使用指令 ng-controller) 就是你所编写的类或者类型，它的作用是告诉 Angular 该模型是由哪些对象或者基本数据构成的，只要把这些对象或者基本数据设置到 `$scope` 对象上即可，`$scope` 对象会被传递给控制器：

```javascript
function TextController($scope) {

  $scope.someText = someText;  

}
```

一个简单的完整的例子：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-1-4.png)

加载到浏览器中，你会看到：

```
You have started your journey.
```

但是大多数应用，还是需要创建模型对象来容纳数据，而且并不建议在全局作用域创建 **TextController**，建议是定义成模块的一部分:

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-1-5.png)

记住：**把东西从全局命名空间中隔离开是一件非常好的事情**，模块机制起这个作用。

2）使用 `ng-bind` 指令，可以在 UI 中的任何地方显示并刷新文本，有两种等价形式：

一种是花括号形式：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-1-6.png)

一种是使用基于属性的指令：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-1-7.png)

花括号插值语法的目的是为了阅读自然，而且需要输入的内容更少，但是在页面加载过程中，Angular 会使用数据替换这些花括号，未被渲染好的模板可能会被用户看到。

造成这种现象的原因是，浏览器需要首先加载 HTML 页面，渲染它，然后 Angular 才有机会把它解释成你期望看到的内容。

其实，加载 HTML 页面一般只发生在首页，所以对于 `index.html` 页面中的数据绑定操作，建议使用 `ng-bind`，其他页面可以使用花括号，而且花括号更快一点。

利用 Chrome 浏览器的一个针对于 Angular 的扩展（**AngularJS Batarang(Stable)**）可以直观的看到两者消耗时间的差别：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-1-8.jpg)

3）表单输入

在 Angular 中使用表单元素非常方便，使用指令 ng-model 属性把元素绑定到你的模型属性上。

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-1-9.png)

对于输入元素来说，你可以使用 `ng-change` 属性来指定一个控制器方法，一旦用户修改了输入值，这个方法就会被调用。下面做一个简单的计算，帮助消费者计算一下需要付多少钱：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-1-10.png)

把输入文本框的值设置为用户估价的 10 倍即可，同时，默认值为 0：

```javascript
function StartUpController($scope) {

    $scope.funding = { startingEstimate : 0};

    $scope.computeNeeded = function () {
        $scope.funding.needed = $scope.funding.startingEstimate * 10;
    }

}
```

但是为了能够正确地刷新输入框，而不管它是通过何种途径进行刷新的，需要使用 `$scope` 中的 `$watch()` 函数：

```javascript
function StartUpController($scope) {

　　$scope.funding = { startingEstimate : 0};

   var computeNeeded = function () {

   　　scope.funding.needed = scope.funding.startingEstimate * 10;

   };

   scope.watch('funding.startingEstimate', computeNeeded);

}
```



请注意，需要监视的表达式位于引号中，这个字符串会被当作 Angular 表达式来操作。这时我们可以使用一个简单的模板：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-1-11.png)

4）浅谈非入侵式 JavaScript

对于大多数内联的时间监听器来说，Angular 有一种等价的形式 `ng-eventhandler = "expression"`，这里的 eventhandler 可以被替换成 click、mousedown、change 等。最重要的是，Angular 将会帮你屏蔽浏览器之间的差异性。

而且不会在全局命名空间中进行操作，你所指定的表达式只能访问元素控制器作用域范围内的函数和数据。

第二点举个例子就是，不同的 **Controller**，可以定义相同名字的数据模型，你会创建一个导航栏和一个内容区域，当你在导航栏中选择不同的菜单项时，内容区域会发生相应的改变。

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-1-12.png)

当用户点击的时候，导航栏中的两个 `<li>` 和内容区域中的 `<div>` 都会调用 **doSomething()** 的函数，你可以在控制器代码中写好这些调用所引起的函数，实际上这两个函数可以是同一个，也可以不是同一个。

```javascript
function NavController($scope) {

    $scope.doSomething = doA;

}

function ContentAreaController($scope) {

    $scope.doSomething = doB;

}
```

请注意，到目前为止，我们所编写的所有控制器中，都没有在任何地方引用 DOM 或者 DOM 操作，所有定位元素和处理事件的工作都是在 Angular 内部完成的。

所以，你可以开心地使用 Angular 的声明式事件处理器，同时又能保持简单性和可读性，没有必要为违反最佳实践而感到内疚。
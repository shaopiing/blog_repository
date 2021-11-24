---
title: "AngularJS 的常用特性 (二)"
date: 2016-07-25 00:05:10
category: 前端
tags: [Program,前端]
---
3、列表、表格以及其他迭代型元素

ng-repeat 可能是最有用的 Angular 指令了，它可以根据集合中的项目一次创建一组元素的多份拷贝。

比如一个学生名册系统需要从服务器上获取学生信息，目前先把模型之间定义在 JavaScript 代码里面：

```javascript
var students = [{name: 'Mary', id: '1'},

                {name: 'Jack', id: '2'},

                {name: 'Jill', id: '3'}];

function StudentListController($scope) {

　　$scope.students = students;

}
```

为了显示这个学生列表，可以编写如下代码：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-2-1.png)

`ng-repeat` 将会生成标签内部所有 HTML 元素的一份拷贝，包括放指令的标签，显示结果如下：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-2-2.png)

根据具体情况分别链接到` /student/view/1`、`/student/view/2` 以及 `/student/view/3`。

**ng-repeat** 指令可以通过 `$index` 返回当前引用的元素序号（类似 struts2 中 iterator 标签中的 index），还可以通过 `$first`、`$middle` 及 `$last`，`ng-repeat` 指令返回布尔值。

4、隐藏和显示

对于菜单、上下文敏感的工具以及很多其他情况来说，显示和隐藏元素是一项核心功能。

ng-show 和 ng-hide 指令的功能是等价的，但是运行效果正好相反。

ng-show 在表达式为 true 时将会显示元素，为 false 时将会隐藏元素；而 ng-hide 则恰好相反。

工作原理：根据实际情况把元素的样式设置为 display : block 来显示元素；设置为 display : none 来隐藏元素。

5、CSS 类和样式

目前你已经可以在应用中动态地设置 CSS 类和样式了，只有使用两对大括号 插值语法把它们进行数据绑定即可，甚至可以在模板中构造 CSS 类名的部分匹配方式。

```javascript
.menu-disabled-true {

    color: gray;

}
```

使用以下模板，可以将功能显示成禁用状态：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-2-3.png)

这样通过控制器来设置 isDisabled 属性：

```javascript
function MenuController($scope) {

    $scope.isDisabled = false;

    $scope.disabledIt = function() {
        $scope.isDisabled = true;
    }

}
```

这样，只有 isDisabled 为 true 时，拼接出来的才是 `menu-disabled-true`，CSS 样式才会起作用。

当使用插值的方式绑定内联样式的时候，同样适用，例如 `style = "{{some.expression}}"`。

但是由于这种方式并不灵活，后期维护困难，所以 Angular 中推荐使用 `ng-class` 和 `ng-style` 指令。
　　
这两个指令都可以接受一个表达式，表达式执行的结果可能是如下取值之一：

- 表示 CSS 类名的字符串，以空格分隔
- CSS 类名数组
- CSS 类名到布尔值的映射

我们可以如下实现显示错误和警告信息的功能：

```css
.error {

    background-color: red;

}

.warning {

    background-color: yellow;

}
```

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-2-4.png)

```javascript
function HeaderController($scope) {

    $scope.isError = false;

    $scope.isWarning = false;

    

    $scope.showError = function () {
        $scope.messageText = 'This is an error';
        $scope.isError = true;
        $scope.isWarning = false;
    };
    
    $scope.showWarning = function () {
        $scope.messageText = 'Just a warning, Please carry on.';
        $scope.isWarning = true;
        $scope.isError = false;
    }

}
```

你会发现这样实现的很优雅，而且容易维护，下面还可以做一个更炫的事情，例如，把表格中被选中的行进行高亮显示。

首先，在 CSS 中设置一个样式：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-2-5.png)

创建控制器：

```javascript
var app = angular.module('app', []);

app.controller('RestaurantTableController', function ($scope) {

    $scope.directory = [{name: 'The Handsome Heifer', cuisine: 'BBQ'},

        {name: 'Green\'s Green Greens', cuisine: 'Salads'},

        {name: 'House of Fine Fish', cuisine: 'Seafood'}];

});

$scope.selectRestaurant = function (row) {

    $scope.selectedRow = row;

}
```

显示效果如图：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-2-6.png)


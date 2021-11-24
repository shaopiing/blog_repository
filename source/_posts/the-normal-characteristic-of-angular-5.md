---
title: "AngularJS 的常用特性 (五)"
date: 2016-10-21 00:01:10
category: 前端
tags: [Program,前端]
---
**13、使用路由和 $location 切换视图**

　　对于一些单页面应用来说，有时候需要为用户展示或者隐藏一些子页面视图，可以利用 Angular 的 $route 服务来管理这种场景。

　　你可以利用路由服务来定义这样的一种东西：对于浏览器所指向的特定 URL，Angular 将会加载并显示一个模板，并实例化一个控制器来为模板提供内容。

　　在应用中可以调用 $routeProvider 服务上的函数来创建路由，把需要创建的路由当成一个配置块传给这些函数即可。伪代码如下：

```javascript
var someModule = anguler.module('someModule', [...module dependencies...]);

someModule.config(function($routeProvider) {

    $routeProvider.

        when('url', {controller : aController, templateUrl : '/path/to/template'}).

        when(...other mappings for your app...).

        ...

        otherwise(...what to do if nothing else matches);

});
```

以上代码中，当浏览器中的 URL 变成指定的取值时，Angular 将会加载 /path/to/template 路径下的模板，然后把这个模板中的根元素关联到 aController 上，最后一行中的 otherwise 调用可以告诉路由，在没有匹配到任何东西时跳转到这里。

下面构建一个小例子，但是需要把代码放到 web 服务器上中。

index.html

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-5-1.png)
　　
list.html

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-5-2.png)

detail.html

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-5-3.png)

controller.js

```javascript
//为核心的 AMail 服务创建模块

var aMailServices = angular.module('AMail', ['ngRoute']);

//在 URL，模板和控制器之间建立映射关系

aMailServices.config(function($routeProvider) {

    $routeProvider.

    when('/', {

        controller: ListController,     // 全局的 function 方式去找 Controller

        templateUrl: 'list.html'

    }).

    //注意，为了创建详情视图，在 id 前面加了一个冒号，从而指定了一个参数化的 URL 组件

    when('/view/:id', {

        controller: 'DetailController', // 用注册的方式去找 Controller

        templateUrl: 'detail.html'

    }).

    otherwise({

        redirectTo: '/'

    });

});

//一些虚拟邮件

messages = [{

        id: 0, sender: 'jean@somecompany.com', subject: 'Hi there, old friend',

        date: 'Dec 7, 2013 12:32:00', recipients: ['greg@somecompany.com'],

        message: 'Hey, we should get together for lunch sometime and catch up.'

        + 'There are many things we should collaborate on this year.'

    }, {

        id: 1, sender: 'maria@somecompany.com', subject: 'Where did you leave my laptop?',

        date: 'Dec 7, 2013 8:15:12', recipients: ['greg@somecompany.com'],

        message: 'I thought you were going to put it in my desk drawer.'

        + 'But it does not seem to be there.'

    }, {

        id: 2, sender: 'bill@somecompany.com', subject: 'Lost python',

        date: 'Dec 6, 2013 20:35:02', recipients: ['greg@somecompany.com'],

        message: "Nobody panic, but my pet python is missing from her cage."

        + "She doesn't move too fast, so just call me if you see her."

    }];

    

//把邮件发布给邮件列表模板，注意两种方式，建议使用下面注册的方式，避免全局的 function 污染

function ListController($scope) {

    $scope.messages = messages;

}

aMailServices.controller('DetailController', function(scope, routeParams) {

    scope.messages = messages[routeParams.id];

});
```

效果如下：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-5-4.png)

**14、与服务器交互**

　　真正的应用需要和真实的服务器进行交互，Angular 中提供了一个叫做 $http 的服务。它提供了一个可扩展的抽象方法列表，使得与服务器的交互更加容易。它支持 HTTP、JSONP 和 CORS 方式。它还包含了安全性支持，避免 JSON 格式的脆弱性和 XSRF。可以让你轻松地转换请求和响应数据，甚至还实现了简单的缓存。

　　返回的响应数据示例如下：

```javascript
[

    {

        "id" : 0,

        "title" : "Paint pots",

        "description" : "Pots full of paint",

        "price" : 3.95

    },

    {

        "id" : 1,

        "title" : "Polka dots",

        "description" : "Dots with that polka groove",

        "price" : 12.95

    },

    {

        "id" : 2,

        "title" : "Pebbles",

        "description" : "Just little rocks, really",

        "price" : 6.95

    }

]
```

我们可以像下面这样编写查询代码：

```javascript
function ShoopingController(scope, http) {

	$http.get('/products').success(function (data, status, headers, config) {

   		$scope.items = data;

    });

}
```

然后就可以应用到模板中了。

**15、使用指令修改 DOM**

　　指令扩展了 HTML 语法，同时它也是使用自定义的元素和属性把行为和 DOM 转换关联到一起的方式。通过这些指令，可以创建可复用的 UI 组件，配置你的应用，并且可以做到你能想象到的几乎所有事情，这些事情都可以在你的 UI 模板中实现。所以说，自定义指令是 Angular 的精髓。

　　与服务一样，你可以通过模块对象的 API 来定义指令，只要调用模块实例的 directive() 函数即可，其中 directiveFunction 是一个工厂函数，用来定义指令的特性。

```javascript
var appModule = angular.module('appModule', [...]);

appModule.directive('directiveName', directiveFunction);
```

指令的东西很多，以后详解，这里先举个例子，感受下指令的魅力。

编写一个非常简单的指令：一个 <hello> 元素，它会把自己替换成 <div>Hi there</div>。

先看指令：

```javascript
var appModule = angular.module('app', []);

appModule.directive('hello', function () {

    return {

        restrict: 'E',

        template: '<div>Hi there</div>',

        replace: true

    };

});
```

这里，restrict 属性表示描述指令的风格，E 表示允许使用元素的形式；template 属性表示需要替换的内容；replace 属性设置为 true 表示会用 HTML 内容来替换模板。

可以在模板中这样使用它：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-5-5.png)

把以上内容加载到浏览器中，就会显示

```
Hi there
```

**16、校验用户输入**

Angular 自动为 "form" 元素增加了一些好用的特性，使其更适合单页面应用。其中一个特性是，Angular 允许你为表单中的输入元素定义一个合法的状态，并且只有当所有元素都是合法状态时才允许提交表单。

控制器如下：

```javascript
var addUser = angular.module('AddUserModule', []);

addUser.controller('AddUserController', function ($scope) {

    $scope.message = '';

    $scope.addUser = function () {
        // 这里把 user 真正保存到数据库中
        $scope.message = 'Thanks, ' + $scope.user.first + ', we added you!';
    }

});
```

模板如下：

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-5-6.png)

在控制器中，可以通过 $valid 属性获取表单的校验状态，当表单中的所有输入项都合法时，Angular 将会把这个属性设置为 true，否则没有输入完成时禁用了 Submit 按钮。

![](http://p8bc1hri5.bkt.clouddn.com/the-normal-characteristic-of-angular-5-7.png)

PS：到目前为止，已经介绍了 Angular 框架的几乎所有常用特性，掌握了这个《常用特性》系列的文章，就算是入门了，加油！
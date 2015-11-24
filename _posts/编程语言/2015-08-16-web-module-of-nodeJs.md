---
layout: post
category : 编程语言
tagline: 
tags : [nodeJs, modules, node_modules, exports]
excerpt : 
title_cn: node学习第二课——node的模块 
description: Node模块打包代码是为了重用，但它们不会改变全局作用域，避免了对全局作用域的污染，从而也就避免了命名冲突，并简化了代码的重用。Node模块允许你从被引入文件中选择要暴露给程序的函数和变量。如果模块返回的函数或变量不止一个，那它可以通过设定`exports`对象的属性来指明它们。但如果模块只返回一个函数或变量，则可以设定`module.exports`属性。
---
{% include JB/setup %}

Node模块打包代码是为了重用，但它们不会改变全局作用域，避免了对全局作用域的污染，从而也就避免了命名冲突，并简化了代码的重用。Node模块允许你从被引入文件中选择要暴露给程序的函数和变量。如果模块返回的函数或变量不止一个，那它可以通过设定`exports`对象的属性来指明它们。但如果模块只返回一个函数或变量，则可以设定`module.exports`属性。

## 一、创建模块

可以是文件或包含文件的目录，如果为后者，默认以`index.js`文件作为模块入口，该文件可以在模块目录下的`package.json`文件的`main`键中定义以取代默认值。模块文件是包含`exports`对象属性定义的文件，例如：
{% highlight javascript %}
// 加元转美元
exports.canadian2US = function (canadian) {
    return roundTwoDecimal(canadian * canadianDollar);
}
{% endhighlight %}

## 二、引入模块

node使用`require`函数引入模块，该函数为同步函数，一般在文件顶端调用来引入模块，应该避免在I/O密集的地方使用该函数，以避免性能问题。模块调用例子如下：
{% highlight javascript %}
// 引入自定义模块，路径相对于当前文件
var currency = require('./lib/currency');
console.log("Canadian2Us - 100 -> us : " + currency.canadian2US(100));
console.assert(currency.canadian2US(100) == 91);
{% endhighlight %}
注意，模块引入的路径应该为相对当前文件的相对路径。

## 三、使用`module.exports`创建模块

有时候，`exports`创建模块不能完全满足需求，例如创建一个根据参数变化的动态模块，此时需要使用`module.exports`函数。例如，创建一个模块，返回一个对象，而node不允许重写exports函数，如果将对象赋值给exports函数，node会提示错误：
{% highlight javascript %}
var Currency = function (canadianDollar) {
    this.canadianDollar = canadianDollar;
}
...
// 错误，node不允许重写exports函数
// exports = Currency;
// 正确
module.exports = exports = Currency;
{% endhighlight %}
调用时：
{% highlight javascript %}
var Currency = require('./lib/currency');
var canadianDollar = 0.91;
var currency = new Currency(canadianDollar);
console.log("Canadian2Us - 100 -> us : " + currency.canadian2US(100));
console.assert(currency.canadian2US(100) == 91);
console.log("Us2Canadian - 91 -> canadian : " + currency.us2Canadian(91));
console.assert(currency.us2Canadian(91) == 100);
{% endhighlight %}
**为什么要使用`module.exports = exports`？**原来，`exports`只是对`module.exports`的一个全局引用，如果写成这样：
{% highlight javascript %}
module.exports = Currency;
{% endhighlight %}
那么，`exports`与`module.exports`的引用关系被打破，那么这个模块的`exports`函数会失效，例如，在这个模块在定义一个`plus`函数：
{% highlight javascript %}
var Currency = function (canadianDollar) {
    this.canadianDollar = canadianDollar;
}
...
module.exports = Currency;
exports.plus = function (a, b) {
    return a + b;
}
{% endhighlight %}
调用：
{% highlight javascript %}
var Currency = require('./lib/currency');
...
console.log(Currency.plus(1, 4));
{% endhighlight %}
程序会抛出异常：`Currency.plus`方法未定义。

**因此，最好采用`module.exports = exports = Currency`这种方式，保持`exports`对`module.exports`的全局引用，以避免其他问题。**

## 四、使用node_modules重用模块

不需要知道相对路径的模块（大多为第三方开发好的程序直接使用的模块，例如`socket.io`模块），放在`node_modules`目录中，引入时无须关心模块的相对路径。例如`var socketIo = require('socket.io')`，node在查找模块时，按照以下流程：
* node的核心模块直接使用，如`var http = require('http')`；
* 非核心模块，先在`node_modules`目录中查找，没有则在父级目录的`node_modules`中查找，没有父级目录则在系统环境变量`NODE_PATH`定义的目录中查找。

`node_modules`下的模块，如果模块为一个目录，那么模块默认的入口为模块根目录下的`index.js`文件，然而，可以在模块根目录下配置`package.json`文件的main值来覆盖默认模块入口，例如，`mime`模块的默认入口为`mime.js`，其`package.json`文件定义如下：
{% highlight javascript %}
{
  "author": {
    "name": "Robert Kieffer",
    "email": "robert@broofa.com",
    "url": "http://github.com/broofa"
  },
  ...
  "main": "mime.js", // 定义模块的入口为mime.js，而非默认的index.js
  "name": "mime",
  "repository": {
    "url": "https://github.com/broofa/node-mime",
    "type": "git"
  }
  ...
}
{% endhighlight %}

源码地址：[https://github.com/sunfuchang/nodeJs-study/tree/master/lesson3](https://github.com/sunfuchang/nodeJs-study/tree/master/lesson3)
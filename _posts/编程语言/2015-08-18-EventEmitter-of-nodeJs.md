---
layout: post
category : 编程语言
tagline: 
tags : [nodeJs, EventEmitter]
excerpt : 
title_cn: node学习第三课——node的事件触发器EventEmitter
description: 我们说`node`是基于事件驱动的、异步的、单线程的平台，因此，事件驱动在`node`中处于非常核心的位置。在`node`中，有一个核心的模块叫做`events`，该模块提供了一个核心的类：`EventEmiiter`。
---
{% include JB/setup %}

## 一、核心事件模块`events`

我们说`node`是基于事件驱动的、异步的、单线程的平台，因此，事件驱动在`node`中处于非常核心的位置。在`node`中，有一个核心的模块叫做`events`，该模块提供了一个核心的类：`EventEmiiter`。

引入该模块：
{% highlight javascript %}
var events = require("events");
{% endhighlight %}

## 二、事件发射器`EventEmitter`

`EventEmitter`（事件发射器）是`node`的核心部分，所有能够发射事件的类都继承于它，同时，也可以通过继承`EventEmitter`开发自己的事件发射器。

创建`EventMitter`：
{% highlight javascript %}
var EventEmitter= require('events').EventEmitter;
var channel = new EventEmitter();
{% endhighlight %}
或者：
{% highlight javascript %}
var events = require('events');
var channel = new events.EventEmitter();
{% endhighlight %}

### 1、监听器`listeners`

绑定到对象上的方法，在事件发射时执行，那么这种方法被称为做监听器(`listeners`)。在监听器方法内部，`this`关键字指向的对象为该监听器关联的`EventListener`。

### 2、监听器的添加

#### 2.1 重复性事件

每次触发事件，监听都重复执行，这样的事件称为“重复性事件”。

{% highlight javascript %}
emitter.addListener(event, listener)
{% endhighlight %}
或者更简单的方式：
{% highlight javascript %}
emitter.on(event, listener)
{% endhighlight %}

例如，以下代码给`server`对象添加一个`connection`事件的监听器，仅仅打印了一句话：
{% highlight javascript %}
server.on('connection', function (stream) {
  console.log('someone connected!');
});
{% endhighlight %}

但是，这种方式添加的监听函数，在每次事件被触发的时候都会执行一次，这种事件称为“重复性事件”，接下来我们看看一次性事件监听。

#### 2.2 一次性事件

当第一次触发事件时，事件监听执行一次，然后立即被移出，再次触发事件时不会在被执行，这种事件成为“一次性事件”。

添加一次性事件监听的api：
{% highlight javascript %}
emitter.once(event, listener)
{% endhighlight %}

例如，下边的代码，仅在第一次`connect`事件触发时执行一次监听函数：
{% highlight javascript %}
server.once('connection', function (stream) {
  console.log('Ah, we have our first user!');
});
{% endhighlight %}

### 3、监听器的移除

{% highlight javascript %}
emitter.removeListener(event, listener)
{% endhighlight %}

### 4、写自己的监听器

在node中，可以通过继承`EventEmitter`类来定义自己的监听器。

例如，定义一个文件监听器：

{% highlight javascript %}

...

// 定义文件监听器
function Watcher (srcDir, destDir) {
	this.srcDir = srcDir;
	this.destDir = destDir;
}

util.inherits(Watcher, events.EventEmitter);

// watch方法，文件监听
Watcher.prototype.watch = function () {
	var watcher = this;
	fs.readdir(watcher.srcDir, function (error, files) {
		if (error) throw error;
		for (var index in files) {
			console.log('Reading file : ' + files[index]);
			// 发射process事件
			watcher.emit('process', files[index]);
		}
	});
}

// 启动文件监听方法
Watcher.prototype.start = function () {
	var watcher = this;
	// 监听文件
	fs.watchFile(watcher.srcDir, function () {
		watcher.watch();
	});
}

...

// 讲源文件移动到目标文件，且文件名改为小写
watcher.on('process', function (file) {
	// 源文件
	var srcDirFile = this.srcDir + '/' + file;
	// 目标文件，名称转为小写
	var destDirFile = this.destDir + '/' + file.toLowerCase();
	// 重命名并移动文件
	fs.rename(srcDirFile, destDirFile, function (error) {
		if (error) throw error;
		console.log('Coping and renaming file : ' + srcDirFile + ' -> ' + destDirFile);
	});
});

...

{% endhighlight %}

这里定义了一个文件监听器，用于监听源目录文件的变化，讲源目录文件移动到目标文件夹并将文件名改为小写。主要用到`fs.readdir`方法和`fs.watchFile`，读取文件时触发`process`事件，来处理业务逻辑。

### 5、其他api：

* 移除全部listner：`emitter.removeAllListeners([event])`
* 设置最大监听器数量：`emitter.setMaxListeners(n)`
默认不超过10个，否则会警告，设置为0表示无限制。
* 查看某一个对象在某一个事件的监听器：`emitter.listeners(event)`
* 触发事件，执行监听器：`emitter.emit(event[, arg1][, arg2][, ...])`
* 获取对象的给定事件的监听数量：`Class Method: EventEmitter.listenerCount(emitter, event)`
* `Event: 'newListener'`：当添加监听器时触发该事件，当该事件触发时，监听器可能还没有添加完成
* `Event: 'removeListener'`：当移除监听器时触发该事件，当该事件触发时，监听器可能还没有移除完成

示例代码：

基于telnet的简单聊天室——[https://github.com/sunfuchang/nodeJs-study/blob/master/lesson4-asynProgramming/chatServer.js](https://github.com/sunfuchang/nodeJs-study/blob/master/lesson4-asynProgramming/chatServer.js)

文件监听器-[https://github.com/sunfuchang/nodeJs-study/blob/master/lesson4-asynProgramming/fileWatcher/fileWatcher.js](https://github.com/sunfuchang/nodeJs-study/blob/master/lesson4-asynProgramming/fileWatcher/fileWatcher.js)
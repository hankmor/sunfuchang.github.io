---
layout: post
category : 技术摘录
tagline: 
tags : [jekyll, 安装, windows, gem, ruby]
excerpt : 
title_cn: win7上安装jekyll
description: 上一篇讲了在github上构建个人站点的基本步骤，现在来详细说下win7上怎么安装jekyll。
---
{% include JB/setup %}

上一篇讲了在github上构建个人站点的基本步骤，现在来详细说下win7上怎么安装jekyll。

## 1、安装windows版本你的ruby：
[http://rubyinstaller.org/downloads/](http://rubyinstaller.org/downloads/)<br>
根据操作系统下载对应的版本，这里我下载的是2.0.0 64位版本。

## 2、安装ruby，记住勾上添加到执行目录到path选项：  
<img src="/assets/images/article_imgs/technology/2015/05/04/1.png" alt="图1" align="center"/>  
安装完成后，确定ruby是否正确安装：  
<img src="/assets/images/article_imgs/technology/2015/05/04/2.png" alt="图2" align="center"/>  
看到如图信息，则说明安装完成。

## 3、下载DevKit：
[http://rubyinstaller.org/downloads/](http://rubyinstaller.org/downloads/)<br>
这里我下载的64位的版本：*DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe*
 
## 4、安装DevKit：
点击下载的exe程序，解压到*D:DevKit*目录
## 5、配置ruby使用DevKit：  
<pre>cd D:\DevKit</pre>

执行*ruby dk.rb init*命令：  
<img src="/assets/images/article_imgs/technology/2015/05/04/3.png" alt="图3" align="center"/>  
如图所示，提示配置*config.yml*文件，暂时不理睬，稍后说明解决办法。  
执行*ruby dk.rb install*命令：  
<img src="/assets/images/article_imgs/technology/2015/05/04/4.png" alt="图4" align="center"/>  
看到报错，原因是64位操作系统需要配置ruby安装的绝对路径，打开*config.yml*文件，最后添加你的ruby安装位置，如图：  
<img src="/assets/images/article_imgs/technology/2015/05/04/5.png" alt="图5" align="center"/>  
注意斜杠，最好按照规范使用左斜杠。  
再次执行安装命令，可以看到安装成功提示：  
<img src="/assets/images/article_imgs/technology/2015/05/04/6.png" alt="图6" align="center"/>  
## 6、安装jekyll：
执行*gem install jekyll*命令，出现一个跟字符相关的错误：  
<pre>ERROR:  While executing gem ... (ArgumentError) invalid byte sequence in UTF-8</pre>  
网上查了下，大概是ruby版本的问题，下载安装ruby2.0版本即可，只是2.1版本为什么有这个问题，我还没搞懂，知情的朋友请告知，多谢指教。   
再次执行如下命令：  
<img src="/assets/images/article_imgs/technology/2015/05/04/7.png" alt="图7" align="center"/>  
可以看到，又出现了一个跟权限有关的错误信息，解决方式如下：  
打开[http://ruby.taobao.org/](http://ruby.taobao.org/)，可以看到如下信息：  
<pre>
$ gem sources --remove https://rubygems.org/  
$ gem sources -a https://ruby.taobao.org/  
$ gem sources -l  
*** CURRENT SOURCES ***   
# 请确保只有 ruby.taobao.org  
$ gem install rails
</pre>

执行对应的命令：  
<img src="/assets/images/article_imgs/technology/2015/05/04/8.png" alt="图8" align="center"/>   
等待rails安装完成，会显示如下信息：  
<img src="/assets/images/article_imgs/technology/2015/05/04/9.png" alt="图9" align="center"/>   
然后安装jekyll：  
<img src="/assets/images/article_imgs/technology/2015/05/04/10.png" alt="图10" align="center"/>   
看到如图所示的信息，说明jekyll安装成功。

**参考文档：**[http://jekyll-windows.juthilo.com/2-jekyll-gem/](http://jekyll-windows.juthilo.com/2-jekyll-gem/)
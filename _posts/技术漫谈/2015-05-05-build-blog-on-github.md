---
layout: post
category : technology
tagline: 
tags : [jekyll, github, 博客]
excerpt : 
title_cn: 使用github和jekyll搭建个人站点系列——总览 
description: 最近在使用github的pages功能，结合jekyll在windowns平台上搭建个人博客系统，经过自己不断的探索和尝试，终于搭建成功，遂记录下来，以供参考和备用。
---
{% include JB/setup %}

最近在使用github的pages功能，结合jekyll在windowns平台上搭建个人博客系统，经过自己不断的探索和尝试，终于搭建成功，遂记录下来，以供参考和备用。

## 前提

1、会用git  
2、懂html和css  
3、有较强的学习能力  
具备以上能力，你就具备了在github上搭建个人博客的基本条件。

## 前言

最近研究了下github的pages功能，发现是一个比较理想的搭建个人博客的方案，主要因为，github的pages功能能够托管我们的站点，而且是免费的，既简单又节约成本，但是个人站点的页面都是静态的，也就是说你博客上的页面都是静态html。当然，github支持jekyll，我们可以集成jekyll来解析我们的静态站点，更加方便，而且功能更强大，写博客就可以直接使用markdown或者textile语法。jekyll官方帮助文档：  
[https://help.github.com/categories/github-pages-basics/](https://help.github.com/categories/github-pages-basics/)。

github的page分为两种：个人/组织站点和项目站点，一般来说，较好的托管于github的开源框架都有帮助页面，例如mybatis的网站为：[http://mybatis.github.io/mybatis-3/](http://mybatis.github.io/mybatis-3/)，如果用来搭建个人站点，推荐用个人站点的方式，大概步骤如下：

### 一、注册github账号
注意你的用户名，因为注册后的站点的域名必须遵循github规定，格式为：username.github.io，所以最好取一个你喜欢的github用户名。注册什么的就不多说了，会上网的都明白。

### 二、安装git
最好下载有命令行的git，推荐git官方版本，下载地址：[http://git-scm.com/download/](http://git-scm.com/download/)，我的操作系统是win7，安装步骤也非常简单，不多说，安装完成后，在git bash或者cmd中输入*git --version*，如果能正确打印出git版本信息，这说明安装成功。

### 三、使用jekyllbootstrap项目作为模板，创建个人站点
当然，你也可以直接迁出别人的博客，修改样式和内容，从而构建自己的博客，前提是你必须安装了git。  

1、创建版本库  

登录github，创建一个以*username.git.io*为名称的版本库，username为你的git用户名，不是邮箱，例如，我的为：[sunfuchang.github.io](sunfuchang.github.io)。  

2、基于jekyllbootstrap迁出你的版本库代码到本地
<pre>
git clone https://github.com/plusjade/jekyll-bootstrap.git USERNAME.github.com
cd USERNAME.github.com
git remote set-url origin git@github.com:USERNAME/USERNAME.github.com.git
git push origin master
</pre>

3、查看你的站点  

代码push到github上后，稍等片刻，你就可以访问你的默认站点了，地址为：http://username.github.io。
当然，此时的站点是jekyllboostrap默认的，你需要修改为你想要的内容，比如修改模板、修改站点配置等等……  

为了能够在本地浏览站点，方便调试、预览，我们还需要安装jekyll——静态页面解析工具。

### 四、安装jekyll
由于我的操作系统是win7 64位旗舰版，在windows上安装比较复杂，而且步骤繁多，请跟随我一步一步安装，稍有不慎，可能导致各种错误，所以定要仔细，详细安装教程见：  
[http://sunfuchang.github.io/technology/2015/05/04/install-jekyll-on-windows/](http://sunfuchang.github.io/technology/2015/05/04/install-jekyll-on-windows/)。

### 五、使用jekyll搭建个人博客
jekyll安装完成后，如果在git bash或者cmd中，输入ruby -v，jekyll -v，gem -v，都能正确显示版本信息，说明jekyll安装成功，接下来就可以使用jekyll编译、预览你的站点了，详细教程稍后补上。

总结，以上内容是基于jekyllbootstrap项目为模板来搭建自己的站点，仅仅讲述了整体的步骤，具体教程稍后补上。当然，你可以直接clone我已经搭建好的站点，然后修修改改，就可以直接构建为你的站点了：[http://sunfuchang.github.io](http://sunfuchang.github.io)。

### 参考网站：
[https://help.github.com/categories/github-pages-basics/](https://help.github.com/categories/github-pages-basics/)  
[http://jekyllbootstrap.com/usage/jekyll-quick-start.html](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)  
[http://jekyllrb.com/docs/home/](http://jekyllrb.com/docs/home/)  
[http://jekyll-windows.juthilo.com/2-jekyll-gem/](http://jekyll-windows.juthilo.com/2-jekyll-gem/)  
[http://ruby.taobao.org/](http://ruby.taobao.org/)
---
layout: post
category : 技术摘录
tagline: 
tags : [nexus, maven, 私服, 搭建]
excerpt : 
title_cn:windows下使用nexus搭建maven私服的流程和说明
description: maven如今已经是非常流行的项目构建工具，开发人员不仅需要使用maven，还需要认识、使用甚至自己搭建maven私服，以便管理jar包、开发、发布jar包等等，甚至实现自动化部署、持续集成都需要用到maven和私服。
---
{% include JB/setup %}

maven如今已经是非常流行的项目构建工具，开发人员不仅需要使用maven，还需要认识、使用甚至自己搭建maven私服，以便管理jar包、开发、发布jar包等等，甚至实现自动化部署、持续集成都需要用到maven和私服。

maven私服有很多开源软件，最常用的就是本文需要介绍的nexus。

## 1、下载nexus：

没什么难的，下载一个oss版本，即`open source`开源版。下载地址：http://www.sonatype.org/nexus/go

如下图所示：

![2463997d-c06d-46e5-84f8-69ea9d70757c.jpg](_posts/技术摘录/img/2015/2463997d-c06d-46e5-84f8-69ea9d70757c.jpg "nexus下载")

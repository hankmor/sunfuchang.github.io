---
layout: post
category : 技术摘录
tagline: 
tags : [nexus, maven, 私服, 搭建]
excerpt : 
title_cn: windows下使用nexus搭建maven私服的流程和说明
description: maven如今已经是非常流行的项目构建工具，开发人员不仅需要使用maven，还需要认识、使用甚至自己搭建maven私服，以便管理jar包、开发、发布jar包等等，甚至实现自动化部署、持续集成都需要用到maven和私服。
---
{% include JB/setup %}

maven如今已经是非常流行的项目构建工具，开发人员不仅需要使用maven，还需要认识、使用甚至自己搭建maven私服，以便管理jar包、开发、发布jar包等等，甚至实现自动化部署、持续集成都需要用到maven和私服。

maven私服有很多开源软件，最常用的就是本文需要介绍的nexus。

## 1、下载nexus：

没什么难的，下载一个oss版本，即open source开源版。下载地址：http://www.sonatype.org/nexus/go

如下图所示：

![2463997d-c06d-46e5-84f8-69ea9d70757c.jpg](/assets/images/article_imgs/technology/2015/12/24/img/2015/2463997d-c06d-46e5-84f8-69ea9d70757c.jpg "nexus下载")

## 2、将下载的zip包或者tgz包解压出来：

![861b0cb9-8f70-4235-aa0f-bf8cfcd394e4.png](/assets/images/article_imgs/technology/2015/12/24/861b0cb9-8f70-4235-aa0f-bf8cfcd394e4.png "")

如图所示，nexus默认的工作目录为统计目录下的sonatype-work目录，可以在`%nexus_home%/conf/nexus.properties`中进行修改。
工作目录的作用：最主要是存储，所有的仓库索引文件、插件索引文件都存在工作目录中(storage、plugin-repository目录)；另外就是存储系统日志（log文件夹）。因此，工作目录应该设置为磁盘空间较大的目录。

## 3、启动nexus：

![b9351649-b0e3-4bb0-a953-7c4980393208.png](/assets/images/article_imgs/technology/2015/12/24/b9351649-b0e3-4bb0-a953-7c4980393208.png "")

`%nexus_home%/bin/js/`下找到操作系统对应的脚本`console-nexus.bat`，运行即可。

nexus默认使用内置的jetty服务器，文件夹中其他的脚本，根据名称很容易理解，无非是安装成windows服务，其他服务、停止服务、卸载服务等。

## 4、登录并修改密码：

nexus默认的管理员账号为`admin`，密码`admin123`，可以通过左侧菜单`security-user`来修改密码。

## 5、仓库管理：

点击菜单栏的view/repostories-repostories菜单，进入仓库界面：
![469f9086-9013-4a2f-a62b-ec34fbbad560.png](/assets/images/article_imgs/technology/2015/12/24/469f9086-9013-4a2f-a62b-ec34fbbad560.png "")

### （1）仓库类型：

1. **hosted**：宿主仓库，其实就是本地的仓库
2. **proxy**：代理仓库，就是当前私服代理了其他的第三方仓库或Apache的中央仓库
3. **virtual**：虚拟仓库
4. **group**：多个仓库可以组成一个组，使用组就相当于在使用组内的仓库成员的资源

### （2）说明：

如图的界面所示，nexus默认有一个仓库组（`public repositories`），其配置可以通过它的`configuration`子标签页查看：
![a1d832cd-782a-4314-9105-66ea4ef51936.png](/assets/images/article_imgs/technology/2015/12/24/a1d832cd-782a-4314-9105-66ea4ef51936.png "")

可以看到，这个组里边默认有本地的`releases`、`snapsots`、`3rd party`库，同时还有`central`库，这些仓库的顺序决定了查找资源的顺序，所以最好将本地的放在前边。

* **releases**库，系统默认的库，存放本地部署的release版包；
* **snapshots**库，系统默认库，存放本地的snapshot版包；
* **3rd party**库，系统默认库，存放第三方包；
* **central**库，代理Apache中间仓库。

到这里，我们的私服已经可以使用了。

## 6、使用仓库：

###（1）修改`maven`的配置文件：

找到maven的配置文件（这里我直接修改`%M2_HOME%/conf/setting.xml`文件），找到`<mirrors>`节点，添加一个镜像节点：

```
<mirror>
    <id>nexus</id>
    <mirrorOf>*</mirrorOf>
    <url>http://localhost:8081/nexus/content/groups/public</url>
</mirror>
```

* **id**：镜像的位唯一标示
* **mirrorOf**：代理哪些仓库，*为所有的资源都从本`maven`私服获取
* **url**：及新搭建的私服的默认仓库组的url地址（通过页面可以查看）
同样，在`<profile>`节点配置一个`<repository>`节点和`<pluginRepostory>`节点。

```
<repository>
    <id>dc-chengdu</id>
    <name>dc-chengdu</name>
    <url>http://192.168.1.223:8081/nexus/content/repositories/central</url>
    <releases>
        <enabled>true</enabled>
    </releases>
    <snapshots>
        <enabled>true</enabled>
    </snapshots>
</repository>
```

ok，配置完成，接下来，我们可以在项目的`pom.xml`中配置需要的jar包，如果本地没有，则会到我们搭建的私服中找其索引文件，并并下载到本地，如果私服没有，则会去仓库组中找（确切的说是仓库组配置的Apache中央仓库去找），找到并将索引文件保存到私服中，将jar包下载到本地仓库中。

因此，我们没必要将中央仓库的所有索引下下来，而是使用到的时候会自动下载。

## 7、如何发布本地的jar包到私服中？
###（1）配置授权：
在`maven`的配置文件（这里我直接修改`%M2_HOME%/conf/setting.xml`文件）中，找到`<servers>`节点，添加两个`<server>`配置：

```
    <server>
        <id>releases</id>
        <username>deployment</username>
        <password>123456</password>
    </server>

    <server>
        <id>snapshots</id>
        <username>deployment</username>
        <password>123456</password>
    </server>
```

* **id**：必须与项目的`pom`中配置`<distributionManagement>`的中的id相同，唯一标示，这里的`release`表示发布`release`版本的包到`release`仓库，而`snapshot`表示发布`snapshot`版本的包到`snapshot`仓库；
* **username**：具有私服发布包权限的用户的`User ID`，具体见私服的权限和用户说明
* **password**：当然是用户的密码。

![a545c31b-39b3-4f7a-90f9-272b8830040d.png](/assets/images/article_imgs/technology/2015/12/24/a545c31b-39b3-4f7a-90f9-272b8830040d.png "")

###（2）配置发布的地址信息
在项目的`pom.xml`配置文件中，配置发布的地址信息：
![d9a6c058-9ca7-4306-95a7-0f38601b8660.png](/assets/images/article_imgs/technology/2015/12/24/d9a6c058-9ca7-4306-95a7-0f38601b8660.png "")

* **id**：与（1）中配置授权时的id一致；
* **url**：私服对应的仓库的url地址。

其实，这部分信息在私服仓库子标签页`summary`可以查看：
![ae52277c-0c53-492b-a5bf-6cb5f22b823a.png](/assets/images/article_imgs/technology/2015/12/24/ae52277c-0c53-492b-a5bf-6cb5f22b823a.png "")

###（3）配置完成:

可以发布你的jar包到私服中了，如果项目的pom.xml中，<version>属性有SNAPSHOT表示为快照版，则会发布到snapshot仓库中，否则，发布到release仓库中。

## 注意事项总结：

1. 私服不会下载中央仓库的所有jar包，而是下载其索引文件，最终的jar包还是来自中央仓库或第三方仓库；
2. 不需要一开始就下载中央仓库的索引文件，该文件很大，而是使用过程中逐渐下载；
3. 注意仓库组的概念，一般来说默认的组已经完全够用了，按照需要，可以自己建立需要的仓库。
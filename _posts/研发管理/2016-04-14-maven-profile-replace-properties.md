﻿---
layout : post
category : 研发管理
tagline : 
tags : [maven,profile]
excerpt : 
title_cn: maven的web工程多个Profile无法替换property属性
description: 最近开发一个web项目，用的maven构建，建立多个profile，对应不同环境，分别包含不同的配置，在打包的时候发现，xml和properties配置文件没有被替换为profile下定义的property属性值。

---
{% include JB/setup %}

## 场景
最近开发一个web项目，用的<code>maven</code>构建，建立多个<code>profile</code>，对应不同环境，分别包含不同的配置，在打包的时候发现，xml和properties配置文件没有被替换为profile下定义的<code>property</code>属性值。

## 解决方案：

WEB工程需要<code>war</code>插件，启用web资源目录的<code>filter</code>功能，而普通jar不需要。步骤如下：

1、定义profile和各个property属性值
2、定义maven-war插件，并确认webresource文件目录
3、开启filter功能
4、执行clean package -P(profile-id) 命令，可以正常替换web资源目录下的形如<code>${}</code>的变量


## 示例

### 1、定义profile

定义dev和uat两个profile，分别对应开发环境和uat环境：
{% highlight xml%}
<profiles>
        <profile>
            <id>dev</id>
            <properties>
                <!-- 数据库相关配置-->
                <mysql.jdbc.url>192.168.1.224:3306/cd_pro</mysql.jdbc.url>
                <mysql.jdbc.username>root</mysql.jdbc.username>
                <mysql.jdbc.password>xxxxxx</mysql.jdbc.password>
                <!-- 环境配置 -->
                <application.context.ip>http://192.168.1.224/</application.context.ip>
                <site.id>510100</site.id>
                <!-- 支付相关配置-->
                <revisa.pay.service.id>123456</revisa.pay.service.id>
                <!--LOG4J日志级别-->
                <log4j.log.level>debug</log4j.log.level>
                <log4j.logfile.path>D:/logs/cd_pro.log</log4j.logfile.path>
                <memorycache.url>192.168.1.224:11211</memorycache.url>
            </properties>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <profile>
            <id>uat</id>
            <properties>
                <!-- 数据库相关配置-->
                <mysql.jdbc.url>10.150.39.21:3306/cd_pro</mysql.jdbc.url>
                <mysql.jdbc.username>root</mysql.jdbc.username>
                <mysql.jdbc.password>xxxxxx</mysql.jdbc.password>
                <!-- 环境配置 -->
                <application.context.ip>http://uat.zaichengdu.com/</application.context.ip>
                <revisa.pay.site.id>510100</revisa.pay.site.id>
                <!-- 支付相关配置-->
                <revisa.pay.service.id>123456</revisa.pay.service.id>
                <!--LOG4J日志级别-->
                <log4j.log.level>info</log4j.log.level>
                <log4j.logfile.path>/data/logs/cd_pro.log</log4j.logfile.path>
                <memorycache.url>10.150.38.106:11211</memorycache.url>
            </properties>
        </profile>
    </profiles>
{% endhighlight %}

### 2、war插件配置和web资源目录定义

{% highlight xml%}
           <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <configuration>
                    <webResources>
                        <resource>
                            <!-- this is relative to the pom.xml directory -->
                            <directory>src/main/resources</directory>
                            <targetPath>WEB-INF/classes/</targetPath>
                            <filtering>true</filtering>
                        </resource>
                    </webResources>
                </configuration>
            </plugin>
{% endhighlight %}

关键是里边filtering配置，设置为true则开启资源过滤，能够将,<code>${}</code>的配置替换为profile里边对应的property属性值。

### 3、修改properties配置文件

我的mysql文件配置
{% highlight xml%}
mysql.jdbc.url=${mysql.jdbc.url}
mysql.jdbc.username=${mysql.jdbc.username}
mysql.jdbc.password=${mysql.jdbc.password}
{% endhighlight %}

### 4、执行命令
{% highlight xml%}
mvn clean package -Puat -maven.test.skip=true
{% endhighlight %}

执行如上命令，然后配置文件会被替换为profile中定义的值：
{% highlight xml%}
mysql.jdbc.url=10.150.39.21:3306/cd_pro
mysql.jdbc.username=root
mysql.jdbc.password=xxxxxx
{% endhighlight %}

## 总结

WEB工程一定要配置war插件，制定web资源目录，并开启过滤功能，文件夹下的文本文件(xml、property)就能够替换profile定义的值。
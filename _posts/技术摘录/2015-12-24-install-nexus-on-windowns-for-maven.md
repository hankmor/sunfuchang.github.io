---
layout: post
category : ����ժ¼
tagline: 
tags : [nexus, maven, ˽��, �]
excerpt : 
title_cn:windows��ʹ��nexus�maven˽�������̺�˵��
description: maven����Ѿ��Ƿǳ����е���Ŀ�������ߣ�������Ա������Ҫʹ��maven������Ҫ��ʶ��ʹ�������Լ��maven˽�����Ա����jar��������������jar���ȵȣ�����ʵ���Զ������𡢳������ɶ���Ҫ�õ�maven��˽����
---
{% include JB/setup %}

maven����Ѿ��Ƿǳ����е���Ŀ�������ߣ�������Ա������Ҫʹ��maven������Ҫ��ʶ��ʹ�������Լ��maven˽�����Ա����jar��������������jar���ȵȣ�����ʵ���Զ������𡢳������ɶ���Ҫ�õ�maven��˽����

maven˽���кܶ࿪Դ�������õľ��Ǳ�����Ҫ���ܵ�nexus��

## 1������nexus��

ûʲô�ѵģ�����һ��oss�汾����open source��Դ�档���ص�ַ��http://www.sonatype.org/nexus/go

����ͼ��ʾ��

![2463997d-c06d-46e5-84f8-69ea9d70757c.jpg](_posts/����ժ¼/img/2015/2463997d-c06d-46e5-84f8-69ea9d70757c.jpg "nexus����")

## 2�������ص�zip������tgz����ѹ������

![861b0cb9-8f70-4235-aa0f-bf8cfcd394e4.png](_posts/����ժ¼/img/2015/861b0cb9-8f70-4235-aa0f-bf8cfcd394e4.png "")

��ͼ��ʾ��nexusĬ�ϵĹ���Ŀ¼Ϊͳ��Ŀ¼�µ�sonatype-workĿ¼��������`%nexus_home%/conf/nexus.properties`�н����޸ġ�
����Ŀ¼�����ã�����Ҫ�Ǵ洢�����еĲֿ������ļ�����������ļ������ڹ���Ŀ¼��(storage��plugin-repositoryĿ¼)��������Ǵ洢ϵͳ��־��log�ļ��У�����ˣ�����Ŀ¼Ӧ������Ϊ���̿ռ�ϴ��Ŀ¼��

## 3������nexus��

![b9351649-b0e3-4bb0-a953-7c4980393208.png](_posts/����ժ¼/img/2015/b9351649-b0e3-4bb0-a953-7c4980393208.png "")

`%nexus_home%/bin/js/`���ҵ�����ϵͳ��Ӧ�Ľű�`console-nexus.bat`�����м��ɡ�

nexusĬ��ʹ�����õ�jetty���������ļ����������Ľű����������ƺ�������⣬�޷��ǰ�װ��windows������������ֹͣ����ж�ط���ȡ�

## 4����¼���޸����룺

nexusĬ�ϵĹ���Ա�˺�Ϊ`admin`������`admin123`������ͨ�����˵�`security-user`���޸����롣

## 5���ֿ����

����˵�����view/repostories-repostories�˵�������ֿ���棺
![469f9086-9013-4a2f-a62b-ec34fbbad560.png](_posts/����ժ¼/img/2015/469f9086-9013-4a2f-a62b-ec34fbbad560.png "")

### ��1���ֿ����ͣ�

1. **hosted**�������ֿ⣬��ʵ���Ǳ��صĲֿ�
2. **proxy**������ֿ⣬���ǵ�ǰ˽�������������ĵ������ֿ��Apache������ֿ�
3. **virtual**������ֿ�
4. **group**������ֿ�������һ���飬ʹ������൱����ʹ�����ڵĲֿ��Ա����Դ

### ��2��˵����

��ͼ�Ľ�����ʾ��nexusĬ����һ���ֿ��飨`public repositories`���������ÿ���ͨ������`configuration`�ӱ�ǩҳ�鿴��
![a1d832cd-782a-4314-9105-66ea4ef51936.png](_posts/����ժ¼/img/2015/a1d832cd-782a-4314-9105-66ea4ef51936.png "")

���Կ�������������Ĭ���б��ص�`releases`��`snapsots`��`3rd party`�⣬ͬʱ����`central`�⣬��Щ�ֿ��˳������˲�����Դ��˳��������ý����صķ���ǰ�ߡ�

* **releases**�⣬ϵͳĬ�ϵĿ⣬��ű��ز����release�����
* **snapshots**�⣬ϵͳĬ�Ͽ⣬��ű��ص�snapshot�����
* **3rd party**�⣬ϵͳĬ�Ͽ⣬��ŵ���������
* **central**�⣬����Apache�м�ֿ⡣

��������ǵ�˽���Ѿ�����ʹ���ˡ�

## 6��ʹ�òֿ⣺

###��1���޸�`maven`�������ļ���

�ҵ�maven�������ļ���������ֱ���޸�`%M2_HOME%/conf/setting.xml`�ļ������ҵ�`<mirrors>`�ڵ㣬���һ������ڵ㣺

```xml
<mirror>
    <id>nexus</id>
    <mirrorOf>*</mirrorOf>
    <url>http://localhost:8081/nexus/content/groups/public</url>
</mirror>
```

* **id**�������λΨһ��ʾ
* **mirrorOf**��������Щ�ֿ⣬*Ϊ���е���Դ���ӱ�`maven`˽����ȡ
* **url**�����´��˽����Ĭ�ϲֿ����url��ַ��ͨ��ҳ����Բ鿴��
ͬ������`<profile>`�ڵ�����һ��`<repository>`�ڵ��`<pluginRepostory>`�ڵ㡣

```xml
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

ok��������ɣ������������ǿ�������Ŀ��`pom.xml`��������Ҫ��jar�����������û�У���ᵽ���Ǵ��˽�������������ļ����������ص����أ����˽��û�У����ȥ�ֿ������ң�ȷ�е�˵�ǲֿ������õ�Apache����ֿ�ȥ�ң����ҵ����������ļ����浽˽���У���jar�����ص����زֿ��С�

��ˣ�����û��Ҫ������ֿ����������������������ʹ�õ���ʱ����Զ����ء�

## 7����η������ص�jar����˽���У�
###��1��������Ȩ��
��`maven`�������ļ���������ֱ���޸�`%M2_HOME%/conf/setting.xml`�ļ����У��ҵ�`<servers>`�ڵ㣬�������`<server>`���ã�

```xml
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

* **id**����������Ŀ��`pom`������`<distributionManagement>`���е�id��ͬ��Ψһ��ʾ�������`release`��ʾ����`release`�汾�İ���`release`�ֿ⣬��`snapshot`��ʾ����`snapshot`�汾�İ���`snapshot`�ֿ⣻
* **username**������˽��������Ȩ�޵��û���`User ID`�������˽����Ȩ�޺��û�˵��
* **password**����Ȼ���û������롣

![a545c31b-39b3-4f7a-90f9-272b8830040d.png](_posts/����ժ¼/img/2015/a545c31b-39b3-4f7a-90f9-272b8830040d.png "")

###��2�����÷����ĵ�ַ��Ϣ
����Ŀ��`pom.xml`�����ļ��У����÷����ĵ�ַ��Ϣ��
![d9a6c058-9ca7-4306-95a7-0f38601b8660.png](_posts/����ժ¼/img/2015/d9a6c058-9ca7-4306-95a7-0f38601b8660.png "")

* **id**���루1����������Ȩʱ��idһ�£�
* **url**��˽����Ӧ�Ĳֿ��url��ַ��

��ʵ���ⲿ����Ϣ��˽���ֿ��ӱ�ǩҳ`summary`���Բ鿴��
![ae52277c-0c53-492b-a5bf-6cb5f22b823a.png](_posts/����ժ¼/img/2015/ae52277c-0c53-492b-a5bf-6cb5f22b823a.png "")

###��3���������:

���Է������jar����˽�����ˣ������Ŀ��pom.xml�У�<version>������SNAPSHOT��ʾΪ���հ棬��ᷢ����snapshot�ֿ��У����򣬷�����release�ֿ��С�

## ע�������ܽ᣺

1. ˽��������������ֿ������jar�������������������ļ������յ�jar��������������ֿ��������ֿ⣻
2. ����Ҫһ��ʼ����������ֿ�������ļ������ļ��ܴ󣬶���ʹ�ù����������أ�
3. ע��ֿ���ĸ��һ����˵Ĭ�ϵ����Ѿ���ȫ�����ˣ�������Ҫ�������Լ�������Ҫ�Ĳֿ⡣
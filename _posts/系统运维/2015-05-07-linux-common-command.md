---
layout: post
category : system_management
tagline: 
tags : [linux, command]
excerpt : 
title_cn: linux常见操作整理
description: Linux是一套免费使用和自由传播的类Unix操作系统，是一个基于POSIX和UNIX的多用户、多任务、支持多线程和多CPU的操作系统。它能运行主要的UNIX工具软件、应用程序和网络协议。它支持32位和64位硬件。Linux继承了Unix以网络为核心的设计思想，是一个性能稳定的多用户网络操作系统。本文记录了Linux常用的一些命令，并不断完善中，以备查阅。
---
{% include JB/setup %}

Linux是一套免费使用和自由传播的类Unix操作系统，是一个基于POSIX和UNIX的多用户、多任务、支持多线程和多CPU的操作系统。它能运行主要的UNIX工具软件、应用程序和网络协议。它支持32位和64位硬件。Linux继承了Unix以网络为核心的设计思想，是一个性能稳定的多用户网络操作系统。本文记录了Linux常用的一些命令，并不断完善中，以备查阅。

## 一、基本操作

1、删除文件命令：
<pre>rm -rf目录循环删除目录</pre>
2、打开/关闭fpt命令：
<pre>service sftpd stop/start</pre>
3、ftp开启命令：
<pre>
   /etc/rc.d/init.d/xinetd stop
   /etc/rc.d/init.d/vsftpd start
   /etc/rc.d/init.d/xinetd start
</pre>
4、<code>svn</code>开启命令： 
<pre>svnserve -d -r /usr/local/svn/</pre>
5、<code>jetty</code>启动：
<pre>
root@ubuntu-server1:/usr/local/jetty-mvn/jetty9/bin# ./jetty.sh start
      Starting Jetty: . . . OK Tue Sep 10 12:30:45 CST 2013
</pre>
6、解压文件到指定目录：
<pre>tar xxx.gz -C /usr/local/</pre>
7、查看权限命令
查看目录的相关权限可以采用命令ls -lD，或者直接用ls -la如:
<pre>ls -l www.jb51.net  //这里表示查看www.jb51.net目录</pre>
8、查看<code>linux</code>机器是32位还是64位的方法：
<pre>file /sbin/init 或者 file /bin/ls</pre>
9、找不到网卡解决：
<pre>vi /etc/udev/rules.d/70-persistent-net.rules</pre>
将最新的<code>eth2</code>网卡改为<code>eth0</code>，并拷贝其ATTR(即最新的mac地址)值，然后编辑<code>eth0</code>的配置文件，将<code>HWADDR</code>改为该值，重启网络服务，问题解决。

10、查看磁盘空间：
<pre>df -hl</pre>
 
11、查看系统用户和组：
<pre>
<code>groups</code>查看当前登录用户的组内成员  
<code>groups gliethttp</code>查看<code>gliethttp</code>用户所在的组,以及组内成员  
<code>whoami</code>查看当前登录用户名 
<code>/etc/group</code>文件包含所有组  
<code>/etc/shadow</code>和<code>/etc/passwd</code>系统存在的所有用户名
</pre>

12、设置环境变量：
<pre>vi /etc/profile</pre>
最后添加
<pre>export PATH=/usr/loca/mysql/bin:$PATH</pre>
 
保存退出执行：
<pre>source /etc/profile</pre>
然后查看：
<pre>echo $PATH</pre>

 13、防火墙：  
（1）、关闭/开启：
<pre>service iptables stop/start</pre>
（2）、添加端口过滤：
<pre>vi /etc/sysconfig/iptables</pre>

## 二、修改权限命令

<pre>
chmod 777 文件名
1.chmod 577 /home/stuser -R
2.umask -p 0200
3.chown XXXX YYYY (XXXX 为用户名 YYYY为文件名）
</pre>

权限列表
<pre>
-rw-------   (600) 只有所有者才有读和写的权限
-rw-r--r--   (644) 只有所有者才有读和写的权限，组群和其他人只有读的权限
-rwx------   (700) 只有所有者才有读，写，执行的权限
-rwxr-xr-x   (755) 只有所有者才有读，写，执行的权限，组群和其他人只有读和执行的权限
-rwx--x--x   (711) 只有所有者才有读，写，执行的权限，组群和其他人只有执行的权限
-rw-rw-rw-   (666) 每个人都有读写的权限
-rwxrwxrwx   (777) 每个人都有读写和执行的权限
</pre>

## 三、查看系统信息命令

<pre>
# uname -a # 查看内核/操作系统/CPU信息 
# head -n 1 /etc/issue # 查看操作系统版本 
# cat /proc/cpuinfo # 查看CPU信息 
# hostname # 查看计算机名 
# lspci -tv # 列出所有PCI设备 
# lsusb -tv # 列出所有USB设备 
# lsmod # 列出加载的内核模块 
# env # 查看环境变量资源 
# free -m # 查看内存使用量和交换区使用量 
# df -h # 查看各分区使用情况 
# du -sh &lt;目录名&gt; # 查看指定目录的大小 
# grep MemTotal /proc/meminfo # 查看内存总量 
# grep MemFree /proc/meminfo # 查看空闲内存量 
# uptime # 查看系统运行时间、用户数、负载 
# cat /proc/loadavg # 查看系统负载磁盘和分区 
# mount | column -t # 查看挂接的分区状态 
# fdisk -l # 查看所有分区 
# swapon -s # 查看所有交换分区 
# hdparm -i /dev/hda # 查看磁盘参数(仅适用于IDE设备) 
# dmesg | grep IDE # 查看启动时IDE设备检测状况网络 
# ifconfig # 查看所有网络接口的属性 
# iptables -L # 查看防火墙设置 
# route -n # 查看路由表 
# netstat -lntp # 查看所有监听端口 
# netstat -antp # 查看所有已经建立的连接 
# netstat -s # 查看网络统计信息进程 
# ps -ef # 查看所有进程 
# top # 实时显示进程状态用户 
# w # 查看活动用户 
# id &lt;用户名&gt; # 查看指定用户信息 
# last # 查看用户登录日志 
# cut -d: -f1 /etc/passwd # 查看系统所有用户 
# cut -d: -f1 /etc/group # 查看系统所有组 
# crontab -l # 查看当前用户的计划任务服务 
# chkconfig –list # 列出所有系统服务 
# chkconfig –list | grep on # 列出所有启动的系统服务程序 
# rpm -qa # 查看所有安装的软件包
</pre>

## 四、svn创建版本库配置
<pre>
cd /home/svn　　
mkdir ityizhan
svnadmin create ityizhan
cd ityizhan/conf
vi passwd

[users]
ituser = itpassword
vi svnserve.conf
password-db = passwd
authz-db = authz
vi authz

[groups]
ityizhan = ituser
[ityizhan:/]
* =
@ityizhan = rw
</pre>

## 五、perl安装

1、在官方网站下载新版本的源码包：  
[http://www.perl.org/get.html](http://www.perl.org/get.html)，版本自己选择，我下载的是*perl-5.12.2.tar.gz*

2、解压<code>/usr/local/src</code>下的*perl-5.12.2.tar.gz*
<pre># tar zxvf perl-5.12.2.tar.gz</pre>

3、建立文件目录，以供安装时使用
<pre>
  # mkdir /usr/local/perl
</pre>                                                                     

4、设置源码  
进入*perl-5.12.2.tar.gz*的解压目录，执行：
<pre>
  # ./Configure --help
</pre>
提示如下: 查看过后，使用这个指令来设置源码：
<pre>
 # ./Configure -des -Dprefix=/usr/local/perl -Dusethreads -Uversiononly
</pre>

5、编译
<pre>
 # make  //这个过程会比较久，因为源码文件有那么大，我的这个有14M。
 # make install
</pre>
等待这个命令完成后，基本安装就完成了。

6、替换掉旧的<code>perl</code>命令
<pre>
 # cd /usr/bin
 # mv perl perl.old       //把原来的perl更名为perl.old，弃用。
 # ln ls /usr/local/perl/bin/perl /usr/bin/perl  //做一个软链接，使用新的perl
</pre> 

7、完成
 <pre># perl -version</pre>使用这个命令查看perl的版本，可以看到，已经更新到5.12.2版本了.


## 六、glibc的安装

有些软件可能要求系统的<code>Glibc</code>高于某个版本才可以正常运行，如果您的<code>Glibc</code>低于要求的版本，为了运行这些软件，您就不得不升级您的<code>Glibc</code>了。比如：
<pre>
qq: error while loading shared libraries: requires glibc 2.5 or later dynamic linker
</pre>

您可以寻找已经编译好的<code>rpm</code>包或者使用源代码的方式升级<code>Glibc</code>。

### RPM包方式安装glibc

<code>RPM</code>虽然比较容易安装，但就是依赖问题不好解决。给出一个下载地址：
[http://mirrors.jtlnet.com/centos/5.5/os/i386/CentOS/](http://mirrors.jtlnet.com/centos/5.5/os/i386/CentOS/)
<pre>
$ rpm –ivh glibc-2.5-49.i386.rpm
</pre>

不过我用的是<code>CentOS</code> 4.8，貌似不能兼容:
<pre>
error: Failed dependencies:
glibc-common = 2.5-49 is needed by glibc-2.5-49.i386
glibc > 2.3.4 conflicts with glibc-common-2.3.4-2.43.el4_8.3.i386
</pre>

安装完成后，可以查看是否已升级：
<pre>
$ ls -l /lib/libc.so.6
lrwxrwxrwx 1 root root 11 10-08 22:08 /lib/libc.so.6 -> libc-2.5.so
</pre>

### 编译安装glibc

下载glibc
<pre>
[root@localhost test]# pwd
/test
[root@localhost test]# wget http://ftp.gnu.org/gnu/glibc/glibc-2.9.tar.bz2
</pre>

下载glibc-linuxthreads
<pre>
[root@localhost test]# wget http://ftp.gnu.org/gnu/glibc/glibc-linuxthreads-2.5.tar.bz2
</pre>

解压
<pre>
[root@localhost test]# tar -jvxf glibc-2.9.tar.bz2
[root@localhost test]# cd glibc-2.9
[root@localhost glibc-2.9]# tar -jvxf ../glibc-linuxthreads-2.5.tar.bz2
</pre>

配置
<pre>
[root@localhost glibc-2.9]# cd ..
[root@localhost test]# export CFLAGS="-g -O2 -march=i486"
[root@localhost test]# mkdir glibc-build
[root@localhost test]# cd glibc-build
[root@localhost glibc-build]# ../glibc-2.9/configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
</pre>

安装
<pre>
[root@localhost glibc-build]# make
[root@localhost glibc-build]# make install
</pre>

安装编译过程中需要注意三点：  
1、要将<code>glibc-linuxthreads</code>解压到<code>glibc</code>目录下。  
2、不能在<code>glibc</code>当前目录下运行<code>configure</code>。  
3、否则如果出现错误：error "glibc cannot be compiled without   optimization"，需要加上优化开关：
<pre>
[root@localhost test]# export CFLAGS="-g -O2 -march=i486"
</pre>

## 七、CentOS修改DNS

1、<code>CentOS</code>修改<code>DNS</code>

修改对应网卡的<code>DNS</code>的配置文件
<pre>
# vi /etc/resolv.conf 
</pre>
修改以下内容
<pre>
nameserver 8.8.8.8 #google域名服务器
nameserver 8.8.4.4 #google域名服务器
</pre>

2、<code>CentOS</code>修改网关 

修改对应网卡的网关的配置文件
<pre>
[root@centos]# vi /etc/sysconfig/network
</pre>

修改以下内容
<pre>
NETWORKING=yes(表示系统是否使用网络，一般设置为yes。如果设为no，则不能使用网络，而且很多系统服务程序将无法启动)
HOSTNAME=centos(设置本机的主机名，这里设置的主机名要和/etc/hosts中设置的主机名对应)
GATEWAY=192.168.1.1(设置本机连接的网关的IP地址。例如，网关为10.0.0.2)
</pre>

3、<code>CentOS</code>修改IP地址

修改对应网卡的IP地址的配置文件
<pre>
# vi /etc/sysconfig/network-scripts/ifcfg-eth0
</pre>

修改以下内容
<pre>
DEVICE=eth0 #描述网卡对应的设备别名，例如ifcfg-eth0的文件中它为eth0
BOOTPROTO=static #设置网卡获得ip地址的方式，可能的选项为static，dhcp或bootp，分别对应静态指定的 ip地址，通过dhcp协议获得的ip地址，通过bootp协议获得的ip地址
BROADCAST=192.168.0.255 #对应的子网广播地址
HWADDR=00:07:E9:05:E8:B4 #对应的网卡物理地址
IPADDR=12.168.1.2 #如果设置网卡获得 ip地址的方式为静态指定，此字段就指定了网卡对应的ip地址
IPV6INIT=no
IPV6_AUTOCONF=no
NETMASK=255.255.255.0 #网卡对应的网络掩码
NETWORK=192.168.1.0 #网卡对应的网络地址
ONBOOT=yes #系统启动时是否设置此网络接口，设置为yes时，系统启动时激活此设备
</pre>

4、重新启动网络配置
<pre>
# service network restart 
或
# /etc/init.d/network restart
</pre>

修改 IP 地址，即时生效:
<pre>
# ifconfig eth0 192.168.0.2 netmask 255.255.255.0 
</pre>
启动生效，修改：
<pre>
/etc/sysconfig/network-scripts/ifcfg-eth0
</pre>

修改网关<code>Default Gateway</code>，即时生效:
<pre>
# route add default gw 192.168.0.1 dev eth0 
</pre>
启动生效，修改：
<pre>
/etc/sysconfig/network
</pre>

修改<code>DNS</code>，修改
<pre>
/etc/resolv.conf
</pre>
修改后可即时生效，启动同样有效。

修改<code>host name</code>，即时生效:
<pre>
# hostname centos1 
</pre>
启动生效，修改：
<pre>
/etc/sysconfig/network
</pre>

手动更改<code>centos</code>为静态IP

1，先搜索了一下，得到以下解释：
<pre>
IP IP地址
Netmark 子网掩码
Gateway 默认网关
HostName 主机名称
DomainName 域名
DNS DNS的IP
</pre>

2，需要修改的文件常有
<pre>
/etc/sysconfig/network
/etc/sysconfig/network-scripts/ifcfg-eth0
/etc/resolv.conf
/etc/hosts
</pre>

**出现’connect:Network is unreachable error’问题，VirtualBox采用的是Bridged Adapter的方式连接。**

通过修改
<pre>
/etc/sysconfig/network-scripts/ifcfg-eth0
</pre>
修改虚拟机的IP地址、network和netmask。
发现能ping同network和netmask，于是断定应该是虚拟机操作系统的路由配置问题，尝试直接修改系统文件：
<pre>
/etc/sysconfig /network-scripts/route-eth0
</pre>
添加
<pre>’defult via 192.168.0.1′（192.168.0.1是我的路由器的IP地址，可以根据自身情况修改）
</pre>
到<code>/etc/sysconfig/network-scripts/</code>目录下发现压根儿没有<code>route-eth0</code>这个文件，于是自己创建了一个，将<code>defult via 192.168.0.1</code>添加到文件中。

运行
<pre>/etc/init.d/network restart</pre>
重启network，一切ok！
---
title: 开发环境初始化
tags:
  - Gradle
  - JDK
  - Maven
  - ssh直连
  - STS
  - SwitchSharp
  - VirtualBox
  - 免密登录
  - 自动挂载
id: '30'
categories:
  - - 开发环境
date: 2015-10-18 08:51:27
---

_从虚拟机安装到基础的开发环境配置完成。_ **基础开发环境：** VirtualBox+Ubuntu emacs+免密码登录配置 chrome+SwithySharp JDK+STS+Git+Maven+Gradle

### **1\. VirtualBox**

VirtualBox普通的安装就不介绍了，本段介绍下移动VirtualBox的Virtual Disk Image和手动扩大磁盘大小的方法。 **移动VirtualBox存储位置** 背景：默认的Virtual Disk Image 存放在 `C:\Users\{UserName}\VirtualBox VMs` ，随着C盘空间越来越小，需要移动Virtual Disk Image的位置。 操作步骤：

1.  将原始XXX.vdi文件移动到D盘。
2.  删除Virtual Box图形界面程序中删除旧的虚拟电脑。
3.  新建虚拟电脑，名称必须与旧的虚拟电脑名称一样，在第3部‘虚拟硬盘’，选择‘使用已有的虚拟硬盘文件’，然后选择在第一部中移动到D盘的XXX.vdi文件即可。

**手动扩大磁盘大小** 背景：随着虚拟机内容的增多，很可能遇到最初配置的硬盘大小不够用，需要扩大硬盘大小。 操作步骤：

1.  打开cmd
2.  进入VirtualBox安装目录 `cd C:\Program Files\Oracle\VirtualBox`
3.  调整虚拟磁盘大小 `VBoxManage modifyhd “D:\virtual-host\dev.vdi” –-resize 10240` (单位M)
4.  进入虚拟机，打开console，输入lsblk查看设备块信息 ，发现sda盘多了10G空间：
    
    zman@zman:/data/zman$ lsblk
    NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda      8:0    0  29.3G  0 disk 
    ├─sda1   8:1    0   5.5G  0 part /
    ├─sda2   8:2    0     1K  0 part 
    └─sda5   8:5    0   2.5G  0 part \[SWAP\]
    sr0     11:0    1  1024M  0 rom
    
5.  使用gparted分配空间，使用方式与windows下一样，安装命令`sudo apt-get install gparted`
6.  查看新分区sda3的UUID
    
    zman@zman:/data/zman$ sudo blkid /dev/sda3
    /dev/sda3: LABEL="data" UUID="4aba2e9c-6347-4386-bdbf-b0362604222a" TYPE="ext4"
    
7.  设置sda3分区自动挂载，修改配置文件/etc/fstab添加如下一行：`UUID=4aba2e9c-6347-4386-bdbf-b0362604222a /data ext4 defaults 0 0` UUID为在第6步获取得。
8.  我习惯将/home目录下的用户空间软链到/data目录下 `ln -s /data/zman /home`。

### 2\. Ubuntu

**分区配置**

/home/zman/apps            #常用工具安装目录
/home/zman/tmp             #临时文件存放目录
/home/zman/workspace       #开发工作空间
/home/lib/                 #公用库安装目录

### 3\. Emacs

sudo apt-get install emacs

### 4. 免密码登录配置

1\. 修改~/.ssh/config，添加目标机器信息：

Host aliHost
  HostName 122.56.112.1
  User root

2\. 在本机生成公私钥 `ssh-keygen -t rsa -P ''` ：id\_rsa和id\_rsa.pub。 3. 将id\_rsa.pub传至目标机器 `scp ~/.ssh/id_rsa.pub aliHost:~/tmp` 4. 在目标机器上，将id\_rsa.pub内容添加到auththorized\_keys文件中 `cat ~/tmp/id_rsa.pub >> ~/.ssh/authorized_keys` 5. 在本机修改.bashrc，添加alias命令 `alias aliHost='ssh aliHost'` 6. 在本机console中输入aliHost即可连接目标机器 **\*\*  工作中经常遇到本地机器不能直连目标机器，需经过跳板机，直连配置步骤与上面相同，但需要修改~/.ssh/config对目标机器开始代理透传**

Host proxyHost
  HostName 123.123.155.155
  User zman
Host targetHost
  HostName 134.134.155.155
  User zman
  ProxyComand ssh proxyHost nc -w 120 %h 22

### 5. chrome+SwithySharp

我习惯使用chrome+SwithySharp搭配，只要墙外有台机器，本地ssh连接时开始socks代理即可。 chrome可以在google.cn下载，SwithSharp可以在http://chrome-extension-downloader.com/搜索‘https://chrome.google.com/webstore/detail/proxy-switchysharp/dpplabbmogkhghncfbfdeeokoefdjegm?utm\_source=chrome-ntp-icon’下载。

### 6\. JDK

cd ~/downloads  #进入下载目录
wget http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-linux-x64.tar.gz --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" #下载
tar zxvf jdk-8u60-linux-x64.tar.gz -C /usr/lib/jvm/ #解压
emacs -nw ~/.bashrc #添加如下内容

#java                                                                                    
export JAVA\_HOME=/usr/lib/jvm/jdk1.8.0\_60
export JRE\_HOME=${JAVA\_HOME}/jre
export PATH=${JAVA\_HOME}/bin:$PATH

**下载地址：http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html**

### 7\. STS（已经转战IntelliJ了）

cd ~/downloads
wget http://dist.springsource.com/release/STS/3.7.1.RELEASE/dist/e4.5/spring-tool-suite-3.7.1.RELEASE-e4.5.1-linux-gtk-x86\_64.tar.gz
tar zxvf spring-tool-suite-3.7.1.RELEASE-e4.5.1-linux-gtk-x86\_64.tar.gz -C ~/apps

自定义配置：

1.  安装主题 http://marketplace.eclipse.org/content/eclipse-color-theme，
2.  General -> Appearance -> Theme：选择Dark。
3.  General -> Appearance -> Color Theme：选择Solarized Dark。
4.  General -> Appearance -> Colors and Fonts -> Text Font: 选择Courier 10。
5.  General -> Keys -> Scheme: Emacs
6.  General -> Editors -> Text Editors: 选中'Insert spaces for tabs'。
7.  General -> Java -> Code Style -> Formatter: 使用Java Conventions，并编辑：Indentation->Tab: Space Only, Off/On Tags: 选择Enable。
8.  General -> Network Connections 配置SOCKS到本地开出的SOCKS端口，我的是：Host: 127.0.0.1，Port: 1080。

或者引入旧配置：

Import -> Preferences -> All (选择导出的配置文件\*.epf)

**下载地址：http://spring.io/tools/sts/all** _国内下载特别慢，可以先在ec2下载好，再scp到本地。_

### 8\. Git

sudo apt-get install git

### 9\. Maven

cd ~/downloads
wget http://mirror.bit.edu.cn/apache/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.tar.gz
tar zxvf apache-maven-3.3.3-bin.tar.gz -C ~/apps
emacs -nw ~/.bashrc #添加如下内容：

#maven                                                                                   
export PATH=/home/zman/apps/apache-maven-3.3.3/bin:$PATH

添加oschina镜像：

mkdir ~/.m2
emacs -nw ~/.m2/settings.xml 内容如下：

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0                                                                                                                                             
                          http://maven.apache.org/xsd/settings-1.0.0.xsd">
      <localRepository/>
      <interactiveMode/>
      <usePluginRegistry/>
      <offline/>
      <pluginGroups/>
      <servers/>
      <mirrors>
        <mirror>
          <id>nexus-osc</id>
          <mirrorOf>\*</mirrorOf>
          <name>Nexus osc</name>
          <url>http://maven.oschina.net/content/groups/public/</url>
        </mirror>
      </mirrors>
      <proxies/>
      <profiles/>
      <activeProfiles/>
</settings>

配置STS： Preferences -> Maven -> Installations: Add Maven的安装目录。 **下载地址：https://maven.apache.org/download.cgi**

### 10\. Gradle

cd ~/downloads
wget https://services.gradle.org/distributions/gradle-2.7-all.zip
unzip gradle-2.7-all.zip -d ~/apps
emacs -nw ~/.bashrc 添加如下内容

#gradle                                                                                  
export PATH=/home/zman/apps/gradle-2.7/bin:$PATH

配置STS： Preferences -> Gradle -> Gradle Distribution -> Folder: /data/zman/apps/gradle-2.7 Preferences -> Gradle -> Gradle User Home -> Directory: /data/zman **下载地址：http://gradle.org/**
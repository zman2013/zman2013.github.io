---
title: WordPress安装步骤
tags:
  - WordPress
id: '4'
categories:
  - - WordPress
date: 2015-10-13 07:29:24
---

_安装WordPress的详细步骤。_ 在阿里云租用了一台ECS，用Ubuntu 14.04 64位系统。 WordPress是一个php网站，因此只需搭建一个php+apache+mysql服务器环境，即可正常使用。 **1\. 更新ubuntu的源列表**

[http://wiki.ubuntu.org.cn/%E6%BA%90%E5%88%97%E8%A1%A8](http://wiki.ubuntu.org.cn/%E6%BA%90%E5%88%97%E8%A1%A8) （我选择了清华服务器）

**2\. 安装php**

sudo apt-get install php5-cli

**3\. 安装apache及php模块**

sudo apt-get install apache2

sudo apt-get install libapache2-mod-php5

**4\. 安装mysql及php模块**

wget http://dev.mysql.com/get/mysql-apt-config\_0.3.7-1ubuntu14.04\_all.deb
sudo dpkg -i mysql-apt-config\_0.3.7-1ubuntu14.04\_all.deb
sudo apt-get update
sudo apt-get install mysql-server

(mysql官方安装文档: http://dev.mysql.com/doc/mysql-apt-repo-quick-guide/en/index.html#apt-repo-fresh-install)

#创建wordpress专用数据库
mysql -uroot -p
#sql: create database wordpress
#安装php mysql拓展
sudo apt-get install php5-mysql php5-curl php5-gd php5-idn php-pear php5-imagick php5-imap php5-mcrypt php5-memcache php5-ming php5-ps php5-pspell php5-recode php5-snmp php5-sqlite php5-tidy php5-xmlrpc php5-xsl

**5\. 下载并安装WordPress** 5.1 下载4.3.1 wget https://wordpress.org/wordpress-4.3.1.zip 5.2 解压到/var/www目录下 sudo unzip wordpress-4.3.1-zh\_CN.zip -d /var/www 5.3 修改apache2的配置，编辑文件/etc/apache2/sites-available/000-default.conf如下：

#DocumentRoot /var/www/html
DocumentRoot /var/www/wordpress

5.4 重启apache2

apachectl restart

5.5 在浏览器输入阿里云ECS的IP即可访问WordPress。 5.6 第一次访问WordPress默认进入数据库配置页面，输入如下信息：

数据库地址

用户名/密码

数据库名称（在步骤4创建的名为wordpress数据库)

5.7 点击完成 **6\. 至此为WordPress的全部安装过程，开始第一篇blog吧。**   Q：安装插件时FTP问题 A：通过web安装插件时，wordpress有可能要求输入ftp凭证。 这是由权限问题引起的：apache的运行用户与wordpress目录的用户不同，将wordpress目录（包含子目录）的拥有者修改apache的运行用户即可。（apache默认用户为www-data） Q：禁止googleapi fonts A：后台搜索‘disable google fonts’，安装、激活。
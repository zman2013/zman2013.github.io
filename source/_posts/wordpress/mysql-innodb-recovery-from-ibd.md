---
title: MySQL从.ibd文件恢复数据
tags:
  - .ibd
  - MySQL数据恢复
id: '115'
categories:
  - - Database
date: 2015-11-22 15:24:53
---

_但愿你没有遇到这样糟糕的事情_ **最简单的情况需要四步** 1. 创建一个表确证与原始表结构一致: `CREATE TABLE <table_name> ...;` 2. 删除新建的表空间: `ALTER TABLE <table_name> DISCARD TABLESPACE;` 3. 复制待恢复的<table\_name>.ibd文件到目标数据库文件夹下面并修改权限:

cp <tablename>.ibd /var/lib/mysql/<database\_name>
cd /var/lib/mysql/<database\_name>
chown mysql:mysql <tablename>

4\. 导入表空间即<table\_name>.ibd: `ALTER TABLE <table_name> IMPORT TABLESPACE;` 5. 如果一切顺利，数据恢复至此完成: `SELECT * FROM <table_name> LIMIT 1;` **也可能是更糟糕的情况** 同样执行上面1 ~ 4步，但第4步失败，并提示如下:

InnoDB: Error: tablespace id in file 
'<table\_name>.ibd' is 30, but in the InnoDB
InnoDB: data dictionary it is 1.

OK，表空间的ID不匹配：新建的<table\_name>对应表空间ID=1，而<table\_name>.ibd对应表空间ID=30。需要继续执行以下步骤： 5. 删除新建的表: `DROP TABLE <table_name>;` 6. 新建28(=30-1)个表，`CREATE TABLE <tmp_table_name_x> ... ;` 7\. 新建<table\_name>, 此表将被分配的表空间ID为30，`CREATE TABLE <table_name> ... ;` 8. 重复步骤2 ~ 4:

mysql> ALTER TABLE <table\_name> DISCARD TABLESPACE;
<-- 复制<table\_name>.ibd文件并修改权限 -->
mysql> ALTER TABLE <table\_name> IMPORT TABLESPACE;

9\. 完成! 测试: `SELECT * FROM <table_name> LIMIT 1;` **随笔** Q：表空间是什么？ A：表中数据存储的地方。以MySQL为例，在独立表空间模式下，一个表对应一个表空间，即<table\_name>.ibd。 Q：数据存储目录在哪里？ A：默认位置/usr/lib/mysql。可以通过SQL命令查看： `show variables like 'datadir';` Q：查看所有数据库？ A：`SHOW DATABASES;` Q：查看所有表？ A：`SHOW TABLES;` Q：导出表数据命令？ A：`mysqldump -u<username> -p<password> <database> <table_name> > <table_name>.sql` Q：导入表数据命令？ A：`mysql -u<username> -p<password> <database> < <table_name>.sql` **[LINK](https://www.zmannotes.com/index.php/2015/11/22/mysql-innodb-recovery-from-ibd/)**

1.  [MySQL InnoDB lost tables but files exist](http://superuser.com/questions/675445/mysql-innodb-lost-tables-but-files-exist)
2.  [Recovering an InnoDB table from only an .ibd file](http://www.chriscalender.com/tag/innodb-error-tablespace-id-in-file/)
3.  [InnoDB数据表空间文件迁移](http://imysql.cn/2008_12_17_migrate_innodb_tablespace_smoothly)
4.  [tablespace](https://en.wikipedia.org/wiki/Tablespace)
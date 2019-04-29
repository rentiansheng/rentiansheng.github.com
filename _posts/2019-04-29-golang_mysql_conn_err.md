---
layout: post
title:  mysql this authentication plugin is not supported
category: [golang, mysql]
tags: [golang,mysql]
---

出现这个是因为你使用MySql 5.7及以上的版本。如果不是就要看了。这篇文章对你没有帮助。

出现问题原因：
mysql5.7中user表的password字段已被取消，取而代之的事 authentication_string 字段。
mysql.user 表中的plugin字段值不对。如果用户对应plugin=caching_sha2_password. 我们遇到的问题是一样的。

确认方式如下：
``` sh
mysql> use mysql;
mysql> select host,user,plugin from mysql.user;
+-----------+------------------+-----------------------+
| host      | user             | plugin                |
+-----------+------------------+-----------------------+
| %         | root             | caching_sha2_password |
| localhost | mysql.infoschema | caching_sha2_password |
| localhost | mysql.session    | caching_sha2_password |
| localhost | mysql.sys        | caching_sha2_password |
+-----------+------------------+-----------------------+
``` 

修复问题方式如下：
``` sh 
mysql> use mysql;
mysql> ALTER USER 'user name'@'host' IDENTIFIED WITH mysql_native_password BY 'user new password';
mysql> flush privileges; 

``` 


确认方案：

修改样例（我修改的root用户）：
```
mysql> use mysql;
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'newpassword';
mysql> flush privileges; 

```
mysql.user 表中用户对应的plugin 字段是否改变为mysql_native_password

``` sh 
mysql> use mysql;
mysql> select host,user,plugin from mysql.user;
+-----------+------------------+-----------------------+
| host      | user             | plugin                |
+-----------+------------------+-----------------------+
| %         | root             | mysql_native_password |
| localhost | mysql.infoschema | caching_sha2_password |
| localhost | mysql.session    | caching_sha2_password |
| localhost | mysql.sys        | caching_sha2_password |
+-----------+------------------+-----------------------+

```

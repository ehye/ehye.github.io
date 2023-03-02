---
title: 在 Ubuntu 上安装和配置 LAMP
date: 2018-05-26 10:34:46
categories: Note
tags:
    - LAMP
    - Apache
    - Ubuntu
    - PHP
    - MySQL
---

Set up and install a LAMP (Linux-Apache-MySQL-PHP) server in Ubuntu 18.04, including Apache 2, PHP 7 and MySQL 5.7. <!--more-->

---

# Installing Apache 2
```
sudo apt install apache2
```

## 改变默认站点位置
**这步可以在最后 LAMP 所有组件都测试通过后再调整**

1. 复制站点配置文件进行备份
```
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/mysite.conf 
```
2. 编辑新的可用站点配置文件
```
sudo vim /etc/apache2/sites-available/mysite.conf
```
    将 DocumentRoot 设置为 `/home/<user>/public_html`
    
3. 改变探测路径
```
sudo vim /etc/apache2/apache2.conf
```
    将 `<Directory /var/www/>` 替换为 `<Directory /home/<user>/public_html/>`

3. 使旧站点失效，新站点生效
```
sudo a2dissite 000-default && sudo a2ensite mysite
```
4. 重启 Apache
```
sudo /etc/init.d/apache2 restart
```



## 测试
建立一个测试页面
```
echo '<b>Hello! It is working!</b>' > /home/<user>/public_html/index.html
```

访问进行测试 `http://your_server_ip/`

---

# Installing MYSQL
安装
```
sudo apt install mysql-server
```
进行安全配置
```
sudo mysql_secure_installation
```

- 跳过安装密码检验插件
> 使用弱密码和自动配置 MySQL 用户凭据的软件 (如 phpMyAdmin)会导致问题。禁用验证是安全的， 但应该始终为数据库凭据使用强而独特的密码
- 禁止 root 远程登录、删除测试数据库等
- 输入 root 密码

---

# Installing PHP

安装 PHP 并重启 apache 
```bash
sudo apt install php libapache2-mod-php php-mysql
sudo systemctl restart apache2
sudo systemctl status apache2
```
看到`active`表明成功
```bash
● apache2.service - The Apache HTTP Server
   Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
  Drop-In: /lib/systemd/system/apache2.service.d
           └─apache2-systemd.conf
   Active: active (running) since Fri 2018-05-25 23:09:25 EDT; 3min 21s ago
  Process: 13308 ExecStop=/usr/sbin/apachectl stop (code=exited, status=0/SUCCESS)
  Process: 13313 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCESS)
 Main PID: 13317 (apache2)
    Tasks: 7 (limit: 571)
   CGroup: /system.slice/apache2.service
           ├─13317 /usr/sbin/apache2 -k start
           ├─13318 /usr/sbin/apache2 -k start
            . . .
```
## Testing PHP 
在 apache 默认站点或者你更改后的自定义位置创建一个测试页面
```
sudo vim /var/www/html/test.php
```
将 php 环境信息显示出来
```php
<?php
    phpinfo();
?>
```
访问页面进行测试
`http://your_server_ip/test.php`

测试成功后出于安全考虑，删除这个文件
```
sudo rm /var/www/html/test.php
```

---

# Installing phpMyAdmin
```bash
sudo apt install phpmyadmin php-mbstring php-gettext
```
 
- 服务器选择`apache2`
> 注意这里比较坑的一点，“apache2"默认是高亮的，但并未选中，因此要按`空格键`然后`tab`再`回车`来选择 apache 作为服务器
- 当询问是否用 dbconfig-common 来设置数据库时选择`Yes`
- 输入一个让 phpMyAdmin 来访问 MySQL 的密码，为空则生成一段随机密码

启动模块
```
sudo phpenmod mbstring
```

## 更改验证方式

目前还不能让 phpMyAdmin 用刚刚的 root 用户登录到 MySQL 数据库。在运行 MySQL 5.7 (和更高版本) 的 Ubuntu 系统中， root 用户默认使用 auth_socket 插件进行身份验证， 而不是密码。这允许在许多情况下获得更大的安全性和可用性, 但是当需要允许外部程序 (如 phpMyAdmin) 访问用户时，它也会使事情复杂化。为了作为 root 用户登录到 phpMyAdmin， 需要将其身份验证方法从 auth_socket 切换到 mysql_native_password

登录数据库
```bash
sudo mysql -u root
```
查看验证方式 
```sql
SELECT user,authentication_string,plugin,host FROM mysql.user;
```

```
+------------------+-------------------------------------------+-----------------------+-----------+
| user             | authentication_string                     | plugin                | host      |
+------------------+-------------------------------------------+-----------------------+-----------+
| root             |                                           | auth_socket           | localhost |
| mysql.session    | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
| mysql.sys        | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
| debian-sys-maint | *C484B7E390A676F6D82EB057C57176B66A34333B | mysql_native_password | localhost |
| phpmyadmin       | *5ED9F84DCC6ACB1D0B6C78366D77C83FA88E4BD9 | mysql_native_password | localhost |
+------------------+-------------------------------------------+-----------------------+-----------+
5 rows in set (0.00 sec)
```
可以看到 root 认证方式是`auth_socket`并且密码为空；
phpMyAdmin 默认不允许空密码登录，下面更改认证方式并添加一个较强的密码**`password`**
```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '<password>';
```

刷新权限
```
mysql> FLUSH PRIVILEGES;
```

再次查看一下，这时就是 `mysql_native_password`

```
+------------------+-------------------------------------------+-----------------------+-----------+
| user             | authentication_string                     | plugin                | host      |
+------------------+-------------------------------------------+-----------------------+-----------+
| root             | *4ACFE3202A5FF5CF467898FC58AAB1D615029441 | mysql_native_password | localhost |
| mysql.session    | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
| mysql.sys        | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
| debian-sys-maint | *C484B7E390A676F6D82EB057C57176B66A34333B | mysql_native_password | localhost |
| phpmyadmin       | *5ED9F84DCC6ACB1D0B6C78366D77C83FA88E4BD9 | mysql_native_password | localhost |
+------------------+-------------------------------------------+-----------------------+-----------+
5 rows in set (0.00 sec)
```

以后在命令行就需要提供密码来登录数据库
```
sudo mysql -u root -p
```

## 解决兼容问题
通过软件源安装的 phpMyAdmin 和 php7.2 不兼容，会有查询错误，解决方法是手动更新 phpMyAdmin
1. 到官网下载最新的 phpMyAdmin

2. 删除或重命名（备份）`/usr/share/phpmyadmin/` 的文件夹

3. 将最新的 phpMyAdmin 解压到 `/usr/share/phpmyadmin/`

4. 编辑文件 `./libraries/vendor_config.php` 将 `define('CONFIG_DIR', '');` 
改为
`define('CONFIG_DIR', '/etc/phpmyadmin/');`

5. 重启 apache

## 测试
`https://your_domain_or_IP/phpmyadmin`
使用 root 和密码应可成功登录


## 增加 http 基本验证
```
sudo vim /etc/apache2/conf-available/phpmyadmin.conf
```
在 <Directory /usr/share/phpmyadmin> 内加一行 `AllowOverride All`，效果如下
```bash
<Directory /usr/share/phpmyadmin>
    Options FollowSymLinks
    DirectoryIndex index.php
    AllowOverride All
    . . .
```

保存并关闭文件，重启 Apache

```bash
sudo systemctl restart apache2
```

编辑 Apache 配置文件, 启用 .htaccess 文件重写
新建一个 .htaccess 文件
```bash
sudo vim /usr/share/phpmyadmin/.htaccess
```
文件内容如下
```
AuthType Basic
AuthName "Restricted Files"
AuthUserFile /etc/phpmyadmin/.htpasswd
Require valid-user
```
设置用户名和密码
```bash
sudo htpasswd -c /etc/phpmyadmin/.htpasswd <username>
```
访问测试
`http://domain_name_or_IP/phpmyadmin`

---
Reference
- https://help.ubuntu.com/community/ApacheMySQLPHP
- https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-ubuntu-18-04
- https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-ubuntu-18-04
- https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql
- https://stackoverflow.com/questions/48001569/phpmyadmin-count-parameter-must-be-an-array-or-an-object-that-implements-co
---
layout:  post
title:		wordpress网站部署
subtitle:		企业门户网站
date:     2018-08-31
author:   Francis-wu
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - wordpress
    - php
---

>最近很扯淡的要弄一个企业门户网站，在github上找了半天之后，找了个spring做的网站，跑起来文章数据显示异常，看了半天没有头绪，想起来干脆用wordprss装个主题用一下好了，将部署过程记在这里，留作参考。（任何问题欢迎邮件:[admin@fraciswu.top](admin@fraciswu.top)）

# 又是一篇部署备忘 ~~划划水 字数补丁~~

## 基本环境
* 1 安装LAMP

    采用yum方式进行安装httpd、MariaDB、php、php-mysql，php-mysql用来进行php和MariaDB数据库的连接。

```
    yum install  httpd  mariadb-server  php php-mysql -y
```
    这样安装的是php的默认版本，需要装高版本的需要用`rpm`安装。
* 2 安装MariaDB
    ```
    yum install mariadb mariadb-client mariadb mariadb-server
    ```

* 3 安装扩展 数据库连接
    ```
    yum install php-mysql
    yum install php-pdo
    ```

* 4 启动服务 设置开机启动
    ```
    systemctl start httpd.service # httpd服务
    systemctl enable httpd.service # 开机启动
    systemctl start mariadb.service #mariadb服务
    systemctl enable mariadb.service #开机启动
    ```

* 5 测试环境
    在/var/www/html目录下新建一个test.php，在里面写下面的代码：
    ```
    <?php phpinfo(); ?>
    ```
    在浏览器访问：`http://ip/test.php`
    出现php版本信息的页面，说明配置成功了。 

* 6 安装phpmyadmin
    根据实际需求看，安装完成后需要修改phpMyAdmin的httpd设置，配置文件为**/etc/httpd/conf.d/phpMyAdmin.conf**

* 首先安装EPEL库：

    ```
    yum install -y epel-release
    ```
    然后安装phpMyAdmin
    ```
    yum install -y phpmyadmin
    ```

## 创建数据库
   ```
   #登录数据库
    mysql -u root -p
    #创建数据库
    CREATE DATABASE wordpress;
    #创建数据库用户和密码
    CREATE USER wordpressuser@localhost IDENTIFIED BY 'wordress_password';
    #设置wordpressuser访问wordpress数据库权限
    GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost IDENTIFIED BY 'wordress_password';
    #刷新数据库设置
    FLUSH PRIVILEGES;
    #退出数据库
    exit
   ```

## 安装wordpress
   ```
   wget http://wordpress.org/latest.tar.gz
   ```
然后解压出来，拷贝到/var/www/html/wordpress目录：
```
# 解压wordpress
tar xzvf latest.tar.gz
# 拷贝到/var/www/html/wordpress目录
sudo rsync -avP ~/wordpress/ /var/www/html/wordpress/


```
然后编辑wp-config.php文件：
```
# 切换到wordpress目录
cd /var/www/html/wordpress
# 复制wp-config.php文件
cp wp-config-sample.php wp-config.php
# 编辑wp-config.php文件
sudo vim wp-config.php


```
然后在配置文件里设置正确的值：
```

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'database_name_here');
/** MySQL database username */
define('DB_USER', 'username_here');
/** MySQL database password */
define('DB_PASSWORD', 'password_here');
/** MySQL hostname */
define('DB_HOST', 'localhost');

```

## 问题解决
### 1、在线安装主题的时候会提示ftp登录，只需要将下面的代码加入WordPress根目录下的wp-config.php文件最后面

```
define('FS_METHOD','direct');
```
这个时候会提示没有权限
```
chmod -R  777 /wordpress(wp安装目录)
```
还不行的话加上下面这个修改分组
```
chown -R www:www /wordpress(wp安装目录)
```

### 2、maria数据库崩溃后不会重启（根本原因没有找到，先设置自动重启）

一般方法：

```shell
#首先确保此文件存在 /etc/systemd/system/multi-user.target.wants/mariadb.service
vi /etc/systemd/system/multi-user.target.wants/mariadb.service

#在[Service]下面加入以下行
Restart=always

#保存退出
:wq

#重新加载系统守护程序

sudo systemctl daemon-reload
sudo systemctl restart mariadb.service
```

如果不行再试试这个方法:

```shell
#让崩溃后的数据库服务器自动重启，需要配置/etc/inittab文件，首先备份一个，编辑此文件一定要非常小心
sudo cp /etc/inittab /etc/inittab.orig
vi /etc/inittab

#结尾处增加一行，在/ etc / inittab文件中放置一个命令，以在mysqld_safe进程崩溃时重新生成mysqld_safe进程。 它有四个字段，每个字段与冒号（:)分隔开

ms:2345:respawn:/bin/sh /usr/bin/mysqld_safe
:wq

#保存后重启服务
systemctl restart mariadb.service 
```



## 使用域名直接访问wordpress网站配置方法

* dashboard设置网站根目录为xxxx.com/wordpress，网站目录为xxxx.com这个时候会报错先不管。

* 登录服务器，在安装wordpress的目录下，执行命令：

  ```shell
  cp index.php ..
  cd ..
  vi index.php
  ```

  然后更改index.php的内容

```php
require( dirname( __FILE__ ) . '/wp-blog-header.php' );1
```

​	改为

```php
require('./wordpress/wp-blog-header.php');
```

* 如果还有adaptive images报错需要把目录下的.hetcss文件也拷贝至上层目录（和上面步骤一样），设置权限777
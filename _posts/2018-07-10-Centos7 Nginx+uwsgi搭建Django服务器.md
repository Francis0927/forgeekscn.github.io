---
layout:  post
title:		Centos7 Nginx+uwsgi搭建Django服务器
subtitle:		部署备忘
date:     2018-07-10
author:   Francis-wu
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Nginx
    - Django
---

# 部署备忘

## 1、基本环境

centos7自带python2.7环境和web.py一致，如果要使用其他环境，推荐使用**virtualenv**进行管理

安装过程能不解压安装就不使用解压安装

换为阿里云的yum源

    cd /etc/yum.repos.d

    sudo mv CentOS-Base.repo CentOS-Base.repo.bak

    sudo wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

    sudo wget -P /etc/yum.repos.d/ http://mirrors.aliyun.com/repo/epel-7.repo 

    yum clean all

    yum makecache

**easy_install**
```bash
yum install python-setuptools
```

**pip**
```bash
yum -y install epel-release
yum -y install python-pip
```

**virtualenv**  不一定要
```bash 
pip install virtualenv
```

**Nginx**
```bash
yum install nginx
```

**gcc**
```bash
yum install gcc
```

**uwsgi环境**
```bash
yum install python-devel mysql-devel zlib-devel openssl-devel
```

**uwsgi**
```bash
pip install uwsgi
```

**yum清理**
```bash
yum clean all
```

## 2、uwsgi Nginx配置文件修改 参考 https://www.tuicool.com/articles/qEVrYn
uwsgi 的配置文件 可支持xml yaml ini等格式。这里使用ini格式的配置文件。默认路径为/etc/uwsgi.ini
```bash
[uwsgi] 
#使用动态端口，启动后将端口号写入以下文件中
socket = /tmp/uwsgi_vhosts.sock
#也可以指定使用固定的端口
#socket=127.0.0.1:9031 
pidfile=/var/run/uwsgi.pid 
daemonize=/var/log/uwsgi.log 
chdir=你的www目录
wsgi-file=启动wsgi的py

master=true 
vhost=true 
gid=nginx 
uid=nginx 

#性能相关的一些参数，具体内容查看官网文档
workers=50
max-requests=5000 
limit-as=512
```

创建uwsgi开机自启动脚本，便于进行系统管理
vi /etc/init.d/uwsgi，内容如下：
```bash
#! /bin/sh  
# chkconfig: 2345 55 25  
# Description: Startup script for uwsgi webserver on Debian. Place in /etc/init.d and  
# run 'update-rc.d -f uwsgi defaults', or use the appropriate command on your  
# distro. For CentOS/Redhat run: 'chkconfig --add uwsgi'  

### BEGIN INIT INFO  
# Provides:			 uwsgi  
# Required-Start:	 $all  
# Required-Stop:	  $all  
# Default-Start:	  2 3 4 5  
# Default-Stop:		0 1 6  
# Short-Description: starts the uwsgi web server  
# Description:		 starts uwsgi using start-stop-daemon  
### END INIT INFO  

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin  
DESC="uwsgi daemon"  
NAME=uwsgi  
DAEMON=/usr/bin/uwsgi  
CONFIGFILE=/etc/$NAME.ini  
PIDFILE=/var/run/$NAME.pid  
SCRIPTNAME=/etc/init.d/$NAME  

set -e  
[ -x "$DAEMON" ] || exit 0  

do_start() {  
	 $DAEMON $CONFIGFILE || echo -n "uwsgi already running"  
}  

do_stop() {  
	 $DAEMON --stop $PIDFILE || echo -n "uwsgi not running"  
	 rm -f $PIDFILE  
	 echo "$DAEMON STOPED."  
}  

do_reload() {  
	 $DAEMON --reload $PIDFILE || echo -n "uwsgi can't reload"  
}  

do_status() {  
	 ps aux|grep $DAEMON  
}  

case "$1" in  
 status)  
	 echo -en "Status $NAME: \n"  
	 do_status  
 ;;  
 start)  
	 echo -en "Starting $NAME: \n"  
	 do_start  
 ;;  
 stop)  
	 echo -en "Stopping $NAME: \n"  
	 do_stop  
 ;;  
 reload|graceful)  
	 echo -en "Reloading $NAME: \n"  
	 do_reload  
 ;;  
 *)  
	 echo "Usage: $SCRIPTNAME {start|stop|reload}" >&2  
	 exit 3  
 ;;  
esac  

exit 0
```

修改脚本属性为可执行
```bash
chmod 755 /etc/init.d/uwsgi 
```

启用开机自动启动
```bash
chkconfig uwsgi on 
```

启动uwsgi服务
```bash
service uwsgi start 
```

配置nginx下的uwsgi站点
例如新增以下一个站点mysite。
```bash
vi /etc/nginx/conf.d/mysite.conf
```
  
```bash
server { 
	listen  9091; 
	server_name  localhost; 
	root /www/mysite; --你的www目录
	index index.html index.htm; 
	access_log /var/log/nginx/mysite_access.log; 
	error_log /var/log/nginx/mysite_error.log; 
	location / { 
		#使用动态端口
		uwsgi_pass unix:///tmp/uwsgi_vhosts.sock;
		#uwsgi_pass 127.0.0.1:9031; 

		include uwsgi_params; 
		uwsgi_param UWSGI_SCRIPT uwsgi;   
		uwsgi_param UWSGI_PYHOME $document_root; 
		uwsgi_param UWSGI_CHDIR  $document_root; 
	} 
}
```

启动Nginx服务
```bash
service nginx start 
chkconfig nginx on 
```

### 访问一直是attempt to write a readonly database django，修改数据库文件用户组无效的情况下，需要修改数据库文件的用户组为nginx
    chown -R nginx:nginx /home/jack/

### 查询当前nginx配置文件位置的方法
`
nginx -t
`
### 查看端口占用以及解除
```bash
netstat -lpnp
```
```bash
kill -9 pid
```
### 服务器重启
nginx
```bash
ngnix -s reload
```

### python包位置
```bash
/usr/lib/python2.x(3.x)/site-packages/
```



### pip easy_install出现'frozenset' object is not callable错误删除包中的.egg












    





---
title: LNMP环境搭建
date: 2021-01-17 14:17:00
description: LNMP环境搭建
slug: lnmp-start
image:
categories:
    - PHP
tags: ["PHP","Laravel"]

---

最近几天没事折腾了下自己的测试服务器，在服务器中搭建lnmp环境，=-=废话不说了，来看文章的你肯定是想知道怎么去搭建lnmp环境，以下我便会把我所知道的教你（有句话是这样说的，你告诉我的，我都忘记了。你教会我的，我都记得），教程实践过程可能会遇到不同的错误，但也不要为此烦恼，尽量去解决就可以了，直到你发出wow的一声。

### 简介

L：linux	N:nginx	M:mysql	P:php

<!--more-->

### 一、通过一键安装包安装

网址：https://lnmp.org/

这个网站上有关于LNMP环境的安装，操作起来简单易上手，适合新手。

### 二、源码编译安装

#### 约定

```
软件源代码包存放位置：/root/lnmp
源码包编译安装位置：/usr/local/软件名
数据库数据文件存储路径/data/mysql
```

#### 安装编译工具和库文件

```sh
# 用yum安装
yum install -y make apr* autoconf automake curl curl-devel gcc gcc-c++  cmake  gtk+-devel zlib-devel openssl openssl-devel pcre-devel gd kernel keyutils patch perl kernel-headers compat* cpp glibc libgomp libstdc++-devel keyutils-libs-devel  libarchive   libsepol-devel libselinux-devel krb5-devel libXpm* freetype freetype-devel freetype* fontconfig fontconfig-devel libjpeg* libpng* php-common php-gd gettext gettext-devel ncurses* libtool* libxml2 libxml2-devel patch policycoreutils bison
```

#### 安装cmake

下载地址啊；https://cmake.org/download/

cmake是跨平台项目管理工具，主要生成CMakeLists.txt文件（相当于configure），主要用于后面编译mysql

更详细的介绍可参考：https://blog.csdn.net/caowei880123/article/details/52497550

```shell
tar -zxvf cmake-2.8.7.tar.gz
cd cmake-2.8.7
./configure --prefix=/usr/local/cmake
make #编译
make install #安装
vim /etc/profile 在path路径中增加cmake执行文件路径
export PATH=$PATH:/usr/local/cmake/bin
source /etc/profile使配置立即生效
```

### 安装pcre

nginx的Rewrite模块和HTTP核心模块会使用到PCRE正则表达式语法。

因此nginx编译需要使用PCRE(Perl Compatible Regular Expression)库，需要安装pcre和pcre-devel，pcre提供编译版本的库，pcre-devel提供开发阶段的头文件和编译项目的源代码。 

下载：https://ftp.pcre.org/pub/pcre/

```shell
tar -zxvf pcre-8.39.tar.gz
cd pcre-8.39
./configure --prefix=/usr/local/pcre 
make && make install
```

#### 安装libmcrypt

```shell
tar -zxvf libmcrypt-2.5.8.tar.gz
cd libmcrypt-2.5.8
./configure #配置
make #编译
make install #安装
```

#### 安装gd库

```shell
tar -zxvf gd-2.0.36RC1.tar.gz
cd gd-2.0.36RC1
./configure --enable-m4_pattern_allow --prefix=/usr/local/gd --with-jpeg=/usr/lib --with-png=/usr/lib --with-xpm=/usr/lib --with-freetype=/usr/lib --with-fontconfig=/usr/lib 
make #编译
make install #安装
```

### 安装Mysql

```shell
groupadd mysql #添加mysql组
useradd -g mysql mysql -s /sbin/nologin #创建用户mysql并加入到mysql组，不允许mysql用户直接登录系统
mkdir -p /var/mysql/data #创建MySQL数据库存放目录
chown -R mysql:mysql /var/mysql/data #设置MySQL数据库目录权限

tar -zxvf mysql-5.5.28.tar.gz #解压

cd mysql-5.5.28 # 进入解压后的目录

# 执行以下命令
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_MEMORY_STORAGE_ENGINE=1 -DWITH_READLINE=1 -DENABLED_LOCAL_INFILE=1 -DMYSQL_DATADIR=/var/mysql/data -DMYSQL_USER=mysql -DMYSQL_TCP_PORT=3306

# 编译和安装
make
make install

cp ./support-files/my-huge.cnf /etc/my.cnf #拷贝配置文件（注意：如果/etc目录下面默认有一个my.cnf，直接覆盖即可）
vi /etc/my.cnf #编辑配置文件,在 [mysqld] 部分增加
datadir = /var/mysql/data #添加MySQL数据库路径
log-error       = /var/mysql/log/error
tmpdir          = /var/tmp

cd /usr/local/mysql 
./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql/ --datadir=/var/mysql/data/ &

# 如果mysql是5.7版本的执行下面这句：
bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql/ --datadir=/var/mysql/data/ & 

#生成系统数据库   这一步真重要     
cd /root/lnmp/mysql-5.5.28
cp ./support-files/mysql.server /etc/rc.d/init.d/mysqld #把Mysql加入系统启动
chmod 755 /etc/init.d/mysqld #增加执行权限
chkconfig mysqld on #加入开机启动
# init.d  就是  /etc/rc.d/init.d  软链接
vi /etc/rc.d/init.d/mysqld #编辑
basedir=/usr/local/mysql #MySQL程序安装路径
datadir=/var/mysql/data #MySQl数据库存放目录

service mysqld start #启动,可能无法写入pid文件，注意将mysql用户权限加入至/usr/local/mysql
chown -R mysql:mysql /usr/local/mysql

# 关于pid的解决方案：
### 1
#执行 
cat /etc/hosts 
vim /etc/hosts 

在
[root@root etc]# cat /etc/hosts 
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 
::1             localhost localhost.localdomain localhost6 localhost6.localdomain6 
下方添加一条
10.36.138.xxx（自己的ip） root（主机名 ）
然后进入
cd /usr/local/mysql   执行
[root@localhost mysql]# ./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/var/mysql/data & 
再启动service mysqld start

### 2
2.Backup your MySQL configuration first.
mv /etc/my.cnf /etc/my.cnf.backup
And restart the MySQL server again:
/usr/local/share/mysql/mysql.server start
Hopefully you will see the following message:
Starting MySQL. SUCCESS!

mkdir /var/lib/mysql #创建目录
ln -s /tmp/mysql.sock /var/lib/mysql/mysql.sock #添加软链接
#先启动服务 service mysqld start
mysql_secure_installation #设置Mysql密码，根据提示按Y 回车输入2次密码
/usr/local/mysql/bin/mysqladmin -u root -p password "123456" #或者直接修改密码

# 到此，mysql安装完成！
```

### 安装Nginx

```shell
tar -zxvf nginx-1.11.5.tar.gz
groupadd www #添加www组
useradd -g www www -s /sbin/nologin #创建nginx运行账户www并加入到www组，不允许www用户直接登录系统
openssl-1.1.0b.tar.gz   #仅仅解压就好 其它不要操作   
cd nginx-1.11.5
./configure --prefix=/usr/local/nginx --without-http_memcached_module --user=www --group=www   --with-http_stub_status_module --with-openssl=/root/lnmp/openssl-1.1.0b --with-pcre=/root/lnmp/pcre-8.39   --with-http_ssl_module

注意:--with-pcre=/lnmp/src/pcre-8.39指向的是源码包解压的路径，而不是安装的路径，否则会报错
```

```shell
make
make install
/usr/local/nginx/sbin/nginx #启动nginx
设置nginx开启启动
vi /etc/rc.d/init.d/nginx #编辑启动文件添加下面内容
=======================================================
#!/bin/bash
# nginx Startup script for the Nginx HTTP Server
# it is v.0.0.2 version.
# chkconfig: - 85 15
# description: Nginx is a high-performance web and proxy server.
# It has a lot of features, but it's not for everyone.
# processname: nginx
# pidfile: /var/run/nginx.pid
# config: /usr/local/nginx/conf/nginx.conf
nginxd=/usr/local/nginx/sbin/nginx
nginx_config=/usr/local/nginx/conf/nginx.conf
nginx_pid=/usr/local/nginx/logs/nginx.pid
RETVAL=0
prog="nginx"
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ ${NETWORKING} = "no" ] && exit 0
[ -x $nginxd ] || exit 0
# Start nginx daemons functions.
start() {
if [ -e $nginx_pid ];then
echo "nginx already running...."
exit 1
fi
echo -n $"Starting $prog: "
daemon $nginxd -c ${nginx_config}
RETVAL=$?
echo
[ $RETVAL = 0 ] && touch /var/lock/subsys/nginx
return $RETVAL
}
# Stop nginxc daemons functions.
stop() {
echo -n $"Stopping $prog: "
killproc $nginxd
RETVAL=$?
echo
[ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /usr/local/nginx/logs/nginx.pid
}
reload() {
echo -n $"Reloading $prog: "
#kill -HUP `cat ${nginx_pid}`
killproc $nginxd -HUP
RETVAL=$?
echo
}
# See how we were called.
case "$1" in
start)
start
;;
stop)
stop
;;
reload)
reload
;;
restart)
stop
start
;;
status)
status $prog
RETVAL=$?
;;
*)
echo $"Usage: $prog {start|stop|restart|reload|status|help}"
exit 1
esac
exit $RETVAL
=======================================================
:wq! #保存退出
chmod 775 /etc/rc.d/init.d/nginx #赋予文件执行权限
chkconfig nginx on #设置开机启动
/etc/rc.d/init.d/nginx restart #重新启动Nginx
service nginx restart
=======================================================
```

### 安装PHP

```shell
tar -jxvf php-7.0.7.tar.bz2	
cd php-7.0.7
./configure --prefix=/usr/local/php7 --with-config-file-path=/usr/local/php7/etc  --with-mysqli=/usr/local/mysql/bin/mysql_config --enable-mysqlnd --with-mysql-sock=/usr/local/mysql/mysql.sock --with-gd --with-iconv --with-zlib --enable-xml --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --enable-mbregex --enable-fpm --enable-mbstring --enable-ftp --enable-gd-native-ttf --with-openssl --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --without-pear --with-gettext --enable-session --with-mcrypt --with-curl --with-jpeg-dir --with-freetype-dir   --with-pdo-mysql=/usr/local/mysql/

make #编译,，若遇到make: *** [ext/fileinfo/libmagic/apprentice.lo] 错误 ，这加参数–-disable-fileinfo
make install #安装

cd /root/lnmp/php-7.0.7
cp php.ini-production /usr/local/php7/etc/php.ini #复制php配置文件到安装目录
rm -rf /etc/php.ini #删除系统自带配置文件
ln -s /usr/local/php7/etc/php.ini /etc/php.ini #添加软链接

cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
cp /usr/local/php7/etc/php-fpm.conf.default /usr/local/php7/etc/php-fpm.conf #拷贝模板文件为php-fpm配置文件

cp /usr/local/php7/etc/php-fpm.d/www.conf.default /usr/local/php7/etc/php-fpm.d/www.conf  

vi /usr/local/php7/etc/php-fpm.d/www.conf  #编辑

user = www #设置php-fpm运行账号为www
group = www #设置php-fpm运行组为www

vim /usr/local/php7/etc/php-fpm.conf
pid = run/php-fpm.pid #取消前面的分号


# 加入服务并开机启动 ，设置 php-fpm开机启动
cp /lnmp/src/php-7.0.7/sapi/fpm/init.d.php-fpm /etc/rc.d/init.d/php-fpm #拷贝php-fpm到启动目录
chmod +x /etc/rc.d/init.d/php-fpm #添加执行权限
chkconfig php-fpm on #设置开机启动

vi /usr/local/php7/etc/php.ini #编辑配置文件

```

```php
这里暂时不给禁用
找到：disable_functions =
修改为：disable_functions = passthru,exec,system,chroot,scandir,chgrp,chown,shell_exec,proc_open,proc_get_status,ini_alter,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server,escapeshellcmd,dll,popen,disk_free_space,checkdnsrr,checkdnsrr,getservbyname,getservbyport,disk_total_space,posix_ctermid,posix_get_last_error,posix_getcwd, posix_getegid,posix_geteuid,posix_getgid, posix_getgrgid,posix_getgrnam,posix_getgroups,posix_getlogin,posix_getpgid,posix_getpgrp,posix_getpid, posix_getppid,posix_getpwnam,posix_getpwuid, posix_getrlimit, posix_getsid,posix_getuid,posix_isatty, posix_kill,posix_mkfifo,posix_setegid,posix_seteuid,posix_setgid,posix_setpgid,posix_setsid,posix_setuid,posix_strerror,posix_times,posix_ttyname,posix_uname

```

列出PHP可以禁用的函数，如果某些程序需要用到这个函数，可以删除，取消s禁用

```
找到：;date.timezone =
修改为：date.timezone = PRC #设置时区
找到：expose_php = On
修改为：expose_php = OFF #禁止显示php版本的信息
找到：short_open_tag = Off
修改为：short_open_tag = On #支持php短标签

<?= ?>

```

### 配置nginx支持php

```nginx
vi /usr/local/nginx/conf/nginx.conf
修改/usr/local/nginx/conf/nginx.conf 配置文件,需做如下修改:

	user www www; #首行user去掉注释,修改Nginx运行组为www www；必须与/usr/local/php/etc/php-fpm.conf中的user,group配置相同，否则php运行出错

user www www;
	worker_processes 1;
	events {
	worker_connections 1024;
	}
	http {
		include mime.types;
		default_type application/octet-stream;
		sendfile on;
		keepalive_timeout 65;
		server {
			listen 80;
			server_name localhost;
				location / {
				root /data/www;
				index index.php index.html index.htm;
				}
				location ~ \.php$ {
				root /data/www;
				fastcgi_pass 127.0.0.1:9000;
				fastcgi_index index.php;
				fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
				include fastcgi_params;
				}
		}
	}
```

```shell
mkdir -p /data/www
chown www:www /data/www/ -R #设置目录所有者
chmod 700 /data/www -R #设置目录权限
```

#### 服务器相关操作命令

```shell
service nginx restart #重启nginx
service mysqld restart #重启mysql
/usr/local/php/sbin/php-fpm #启动php-fpm
/etc/rc.d/init.d/php-fpm restart #重启php-fpm
/etc/rc.d/init.d/php-fpm stop #停止php-fpm
/etc/rc.d/init.d/php-fpm start #启动php-fpm
```

### 编写自动化启动脚本

```shell
vim /sbin/lnmp  


#!/bin/bash 

if [ $1 == "start" ];then
/etc/init.d/nginx start
/etc/init.d/php-fpm start
/etc/init.d/mysqld start
echo 'success'
fi

if [ $1 == "stop" ];then
/etc/init.d/nginx stop
/etc/init.d/php-fpm stop
/etc/init.d/mysqld stop
echo 'success'
fi

if [ $1 == "restart" ];then
/etc/init.d/nginx restart
/etc/init.d/php-fpm restart
/etc/init.d/mysqld restart
echo 'success'
fi


chmod +x /sbin/lnmp 
```



### Nginx中的配置，创建vhost

```shell
# 编辑 nginx.conf 
vim /usr/local/nginx/conf/nginx.conf  
# 倒数第二行  写入  
include /usr/local/nginx/conf/vhost/*.conf; #把所有的.conf 文件包含进来 


cd /usr/local/nginx/conf/vhost   

# 创建一个网站的配置文件 例如：vim www.xxx.com.conf
vim www.xxx.com.conf  
# 写入以下内容，注意空格

server {
        listen       80;
        server_name  www.xxx.com xxx.com;  #多个vhost 只需要复制然后改这个  
		root /data/www/xxx; # 网站存放目录。 全局 这里不能跟   /usr/local/nginx/conf/nginx.conf 的root 一样 一定要新建一个   再就是改这个  即可  
        location / {
            index  index.php index.html index.htm;
        }

	location ~ \.php$ {
		fastcgi_pass 127.0.0.1:9000;
		fastcgi_index index.php;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include fastcgi_params;
	}
}


配置本地 hosts 文件  
别忘了重启 service  nginx  restart  或者 一键启动脚本   
service nginx restart
mkdir -p /data/www/tawan
cd /data/www/tawan
vim index.php
```
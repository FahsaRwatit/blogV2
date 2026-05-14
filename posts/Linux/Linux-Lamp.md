---
title: Linux中Lamp环境的搭建
description: Linux中Lamp环境的搭建
date: 2019-05-10 15:10:00
slug: Linux-Lamp
image:
categories:
    - Linux
tags: ["Linux"]

---

## yum 安装  lamp    

```shell
#先安装 apache 
yum -y install httpd 
#加入开机启动   
chkconfig httpd  on  
#启动apache    
service httpd start    service  httpd start|stop|restart 
#修改配置文件  
vim /etc/httpd/conf/httpd.conf 

第276行  取消 注释   改为   ServerName 127.0.0.1:80

service  httpd restart  
#关闭防火墙    
service  iptables stop    接下来到浏览器访问 即可            

#安装 apache 相应的扩展  
yum -y install httpd-manual（手册） mod_ssl mod_perl（加密链接） mod_auth_mysql（mysql 认证链接）

#安装 mysql   
yum -y install  mysql mysql-server mysql-devel   
mysql 客户端 
mysql-server 服务端 
mysql-devel mysql 开发工具包  
#加入开机启动  
chkconfig  mysqld on 
#启动mysql 服务   
service mysqld  start    

#修改密码  
/usr/bin/mysql_secure_installation  
根据提示输入即可   
#service mysqld restart  


#安装 PHP    
yum -y install php php-mysql
#安装PHP相应的扩展  
 yum -y install gd php-gd gd-devel php-xml php-mbstring php-pdo php-mysqli php-smtp php-imap php-common php-curl php-xmlrpc 
 
#重启apache  
service httpd restart  

根目录位于 /var/www/html  下    


```

## 编译安装   

```
机器只是识别二进制  我们希望机器能够识别我们的代码 所以我们就把代码变成可执行文件  这个过程就叫做编译    

把代码编程 二进制 也就是机器识别的文件 这个工具 叫做编译器   
软件 分为 GUI 还有命令行  
Linux下面的软件 全是 C 和 C++开发 的  


./configure  #配置过程   会有好多参数 
--prefix   #指定安装在哪个目录下  
--with  #依赖于什么服务     --without
---enable #启动某些参数     --disable  


make  #编译   
make install #安装    

make && make install  
```

## 编译安装  apache  

```
所有的软件是c c++ 写的  

http://mirrors.hust.edu.cn/apache  
 wget -c http://mirrors.hust.edu.cn/apache/apr/apr-1.6.3.tar.gz
 wget  -c http://mirrors.hust.edu.cn/apache/apr/apr-util-1.6.1.tar.gz
wget -c https://ftp.pcre.org/pub/pcre/pcre-8.42.tar.gz

1.安装 编译器  
yum -y install gcc gcc-c++ 
yum -y install expat-devel 
2.安装 apr   
tar -zxvf apr-1.6.3.tar.gz 
cd apr-1.6.3
./configure --prefix=/usr/local/apr #配置过程指定安装在哪个目录
make 
make install   

3.安装apr-util

tar -zxvf apr-util-1.6.1.tar.gz
进入目录
cd apr-util-1.6.1 
./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr   
make && make install  

4.安装pcre
tar -zxvf pcre-8.42.tar.gz

cd pcre-8.42

./configure --prefix=/usr/local/pcre     
make && make install   

5安装httpd 
http://mirrors.hust.edu.cn/apache/httpd/httpd-2.4.33.tar.gz 
tar -zxvf httpd-2.4.33.tar.gz 
cd   httpd-2.4.33
./configure --prefix=/usr/local/apache --with-apr-util=/usr/local/apr-util/  --with-pcre=/usr/local/pcre/
make 
make install 



cd  /usr/local/apache/bin
./apachectl start   启动服务  
cd /usr/local/apache/conf  
vim httpd.conf   
?ServerName  
ServerName 127.0.0.1:80  改为   

cd  /usr/local/apache/bin
./apachectl restart

关闭防火墙  
service  iptables stop  


根目录位于  
/usr/local/apache/htdocs     

```

## 防火墙     

> 0~127    是系统预留的端口     
>
> 1000 以下  需要 root权限       
>
> 端口开放   80 443 21 22 3306 25 110   由iptables 负责看管            

```
vim /etc/sysconfig/iptables  

service  iptables  restart|start|stop|save 
-A 增加一条规则 
-I 也是增加一条规则   
-p 协议名称 tcp  udp   ICMP  （ping 走的是这个协议）
--dport 跟 p 一起使用  制定 目标端口    
--sport 源端口   也是跟p 一起使用  
-s 源 IP  
-d 目标 ip 
-j 跟动作  accept  接受  drop丢弃掉   reject 拒绝包    
-i 制定网卡      
INPUT   入  
OUTPUT 出  
允许 ping 其它主机 不允许其它主机ping我     



iptables -I INPUT -s 10.36.138.17 -p tcp --dport 80 -j DROP   拒绝  ip访问80端口   
iptables -I INPUT -p icmp --icmp-type 8 -j DROP 只允许 ping 别人 不允许别人ping   
iptables -I OUTPUT -p tcp --dport 22 -d 10.36.138.244 -j DROP  拒绝 ssh 访问 10.36.138.244 



iptables  -nvL  查看规则  
iptables -F 清空规则   
service iptables save #保存规则   
iptables -nvL --line-numbers    
会有一个num 选项        
iptables -D OUTPUT|INPUT n   删除第几条规则   


iptables  限流方案    当请求数 达到10000 每分钟限流 200  
iptables -A INPUT -p tcp --dport 80 -m limit --limit 200/minute --limit-burst 10000 -j ACCEPT

 --limit-burst 10000 访问量达到10000  
 就开启限制  -m limit 
 每分钟 --limit  200/minute
```

## selinux 红帽阵营特有的软件   杀毒软件    

```
关闭selinux     
   vim /etc/selinux/config   
       enforcing 改为 disabled    
       
   setenforce 0   临时关闭  也起到一个 起到让配置生效的目的   
   getenforce  获取selinux的状态    
```



## 开机启动     

```
chkconfig httpd on   不是所有的 服务都可以加入到 开机启动    而是只有在/etc/init.d 的服务才可以   或者说支持  加入开机启动的服务才可以         init.d  是  rc.d/init.d的软链接           
在 /etc/rc.d/init.d  下面的服务  可以 使用service  服务 restart|start|stop    

service  iptables  restart  
/etc/init.d/iptables restart  
/etc/rc.d/init.d/iptables restart   三个效果一样  

chkconfig --list  #列出所有的开机启动项       
vim /etc/inittab   
存放系统的启动 级别      
init 0 关机     
init 1 单用户模式    
init 2 多用户模式  没有网络       启用
init 3 多用户模式 有网络  有NFS   启用
init 4 留给用户自定义             启用
init 6 重启   
init 5 进入图形界面               启用 

chkconfig  服务名  on  
chkconfig 服务名 off 取消开机启动      
chkconfig --level 2345 mysqld on|off   也是加入开机启动取消开机启动的方法  

```

## 计划任务    

```shell
php while+sleep  
js  setTimeout  setInterval  clearTimeout clearInterval   
linux   
date -s 分：时：秒 
date -s 月：日：年

vim /etc/crontab  
* * * * * 用户名   命令      

crontab  -e  增加一条任务    
#这里边  只是 分 时 日  月 周  命令    没有  用户名  
crontab  -l 列出所有的计划任务  
#这里列出的计划任务是  crontab -e 添加的计划任务  不是  /etc/crontab   
crontab -r 删除所有的计划任务   
#这里删除的是 crontab -e 添加的 计划任务      

crontab -u  # 跟  -e 一起使用 制定用户名 计划任务      
```

| 命令                     | 任务名称                                        |
| ------------------------ | ----------------------------------------------- |
| 0 0 * * * mysqldump      | 每天的0点 备份数据库                            |
| 0 5 * * 0  rsync         | 周日凌晨五点  备份所有数据                      |
| 0 0  1 * * 工资.php      | 每月的1号清算上个月的工资                       |
| 0 */8    *  *  * 脚本    | 每隔8小时 执行一次脚本                          |
| 0 8,12,18 * * * 短信.php | 每天的 8点  12点 18点发送一次短信  看谁迟到早退 |
| 0 9-12 * * * 点名.php    | 每天的9点到12点  点名 一次  看谁在班里          |
|                          |                                                 |
|                          |                                                 |
|                          |                                                 |

## 进程管理   

```
ps  显示进程  

  -a 显示会话信息    
  -u 显示用户的id  
  -x 进程的信息    
  -e 显示所有的进程  
  -f 显示组信息   uid gid  
ps -ef  显示所有的进程  


ps -aux | grep httpd  查看httpd 服务是否启动   

kill -9  进程号   杀死进程   

killall  -TERM httpd 所有httpd 相关的进程全部干掉    


netstat  -an   显示当前所有的连接     
     -a 所有    
     -n 端口号  
     -t tcp 协议 监测tcp协议    
     -u udp 协议 
     -l 监听   
     -p 进程 
     
     netstat  -ntlp  | grep  80  
     
总结  : 1.ps -aux | grep httpd   
		2.netstat -ntlp | grep 80 查看 服务是否启动  
		
		
管理服务器的时候  这个 非常重要    

```

## 查看服务器的负载情况  

```
w   
 16:14:07 up  7:29,  2 users,  load average: 0.00, 0.02, 0.05
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
root     tty1     -                15:55   26:18   0.08s  0.08s -bash
root     pts/0    10.36.138.237    15:48    0.00s  0.09s  0.00s w
load average: 0.00, 0.02, 0.05      每一分钟的负载情况 0.02 每五分钟的负载情况   0.05 每15分钟的负载情况  

单核 服务器   这三个值不能超过 1   
双核 不能超过2  
值 越大  负载越高    

tty1  终端登录  
 pts/ 远程登录  




top  相当于windows 的任务管理器 Ctrl+shift+esc    
yum -y install  htop    
不行的化 搜索  htop-1.0.3-1.el6.x86_64   rpm  用 rpm -ivh  来安装使用即可     
```

## htop 安装  

```
wget -c  http://hisham.hm/htop/releases/2.2.0/htop-2.2.0.tar.gz
yum -y install gcc gcc-c++ 
 yum install -y ncurses-devel
tar -zxvf  htop-2.2.0.tar.gz 
./configure --prefix=/usr/local/htop --disable-unicode  
make 
make install  

cd /usr/local/htop/bin   
./htop   

cp /usr/local/htop/bin/htop /sbin/htop 

然后  htop  即可  
/sbin/下面的所有 可以直接使用    
```

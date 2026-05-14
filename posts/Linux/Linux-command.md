---
title: Linux 使用及常用命令
description: 一些Linux中常使用的命令记录
date: 2019-03-10 15:10:00
slug: Linux-command
image:
categories:
    - Linux
tags: ["Linux"]

---

### 常见端口 

```shell
http  80
https 443
ftp 21
ssh 22
scp 22 
smtp 邮件发送服务器  25
pop3 接收邮件  110 
系统的 预留的端口号 0~127 
```

### 常用的命令  

```shell
--help    #查看帮助  比如 ls --help      
man ls    #man手册查看帮助   需要先  安装  yum -y install man   
whoami #查看当前到底哪个用户登录  
date #查看当前日期和时间  
cal #查看日历  
cal 2018 #查看制定年份全年的日历 
sync #将内存中的数据 写入磁盘中   在关机或者重启的时候 要执行一次  

reboot 
init 6  #这两个是重启命令   

shutdown -h now #立即关机  
shutdown -h 0:00 #定时关机  

halt 
init 0 
power off #上面三个 都是关机命令  

ifconfig  #查看网卡 信息 ip地址  
ping #查看网络是否通    
su #切换用户    
cd #切换目录 
ls #列出 目录下面的文件 和子目录   
mv #重命名  
passwd 用户名 #修改密码  
vi 文件名  #修改内容  
service 服务名 restart|start|stop 

echo 内容  #打印内容   

windows 常用的命令  

notepad #打开记事本  
note #设备和打印机 
calc #计算器 
logoff #注销  退出当前用户   
shutdown #关机  
任务计划 #定时任务  
lusrmgr.msc #本地用户和组    
services.msc  #本地服务 
cleanmgr #垃圾清理
diskmgmt.msc #磁盘分区工具  

gpedit.msc #组策略   

```

### Linux 目录结构 

Linux下面  一切都是文件   访问 设备等方式 跟访问文件的方式 是一样的   所有的目录都是从 /  根目录触发   

```shell
yum -y install tree   
cd  / 
tree -L 1  查看第一级目录树     

7个重要:root,home,usr,var,dev,etc,,mnt(挂载)
一个一般重要:tmp


.
├── bin   #存放 经常使用的命令  普通用户可以使用 
├── boot  #Linux启动的核心文件
├── 重要 dev   #device 设备 硬盘在这个目录下 存放Linux的外部设备 比如打印机显示器   
├──重要 etc   #类似于tp框架中的config.php 存放系统管理所需要的配置文件     
├── 重要 home  #普通用户的家目录
├── lib   # 存放系统最基本的动态链接库 共享库   类似于windows 下面的.dll文件   
├── lib64 #64位操作系统所需要的动态链接库  
├── lost+found #当非法关机的时候 这里产生一些文件  临时文件  
├── media  #系统自动识别外部设备  比如我们的U盘  自动挂载到这里 挂载就是 类似于U盘插到电脑上  不能直接查看u盘内容 但是 我们可以访问我的电脑  把U盘插到电脑上 我们就可以跟访问 D盘 一样访问 U盘    
├── 重要 mnt  # mount 挂载的意思   挂载不同文件系统类型的文件 比如 挂载 NTFS类型的文件 一般使用它来手动挂载文件
├── opt # 安装额外装X的软件  一般安装这个目录下  比如oracle  
├── proc #从这里获取系统的相关信息  但是这里边的信息 来源于内存中  
├── 重要 root  #管理员用户的家目录 和 ~ 是一个目录
├── sbin  #也是存放命令的目录  不过是管理员才有权限使用的命令 
├── selinux # 红帽阵营特有的 软件     好比杀毒软件 
├── srv  # 系统启动以后要从这里提取数据   
├── sys # 驱动的实时信息 
├── 一般重要 tmp #临时目录   当系统重启以后 可能会丢失 
├── 重要  usr # 类似于 windows的 C:\Program Files 软件安装目录   一般的应用软件 默认安装在这个目录下   
└──  重要  var # 可以变化的目录   日志  进程  文件存放目录  
```

### 终端快捷键  

| 快捷键 | 作用         |
| ------ | ------------ |
| Ctrl+c | 强制终止     |
| tab    | 自动补全     |
| Ctrl+a | 回到命令开头 |
| Ctrl+e | 回到命令结尾 |
| Ctrl+U | 清空命令行   |
| Ctrl+L | 清空屏幕     |

### 文件的相关操作  

```shell
ls  
   -a  显示 . 开头的隐藏文件  
   -l 以相信信息的方式展示文件或者目录  
   -al    
ll 等同于  ls -l   


cd  切换目录   
   cd  不写路径 默认切换到  /root 目录下 
   cd /etc/sysconfig    
   cd ../../ 切换到上两级目录   
   cd .. 上一级   
   cd .  当前目录   
   cd ./  也是当前目录  
   
pwd  查看当前位于哪个目录下面  
   
vi 名称  保存  可以创建一个文件       

touch  文件   创建文件
touch 文件1 文件2 可以批量创建    #Linux 不严格注重扩展名
rm  文件名  #会有提示  
rm -f 文件名  #强制删除    不会提示 

mkdir 目录名称   #创建目录  
mkdir 目录1  目录2 目录3  #可以批量创建  
mkdir -p 目录/子目录/孙目录  #递归创建目录   

rm -rf 目录1 目录2 文件1 文件2 

rm -rf test* *.php   
慎用 rm -rf   / 下面 所有的目录 不要这么用 

cp 文件1 文件2  
cp -r 目录1  目录2  这是复制目录  

mv 目录或者文件   新目录名/新文件名  #在当前目录下  就是 重命名   move 
mv 目录或者文件   新的路径下    #移动

echo 内容 > /root/test.php  打印消息到文件中   
echo 内容 >> /root/test.log 追加消息到文件中    
```

### 文件的搜索  

```shell
 find / -name 关键词   #find / -name *.php 
 查找命令 : 
 	which  find  查找 指定的命令所在的目录   
 	whereis find  
```

### vi/vim 编辑器 

| 快捷键      | 作用                   |
| ----------- | ---------------------- |
| H           | 向左移动               |
| J           | 向下移动               |
| K           | 向上移动               |
| L           | 向右移动               |
| ESC         | 从编辑模式回到命令模式 |
| yy          | 复制一行               |
| p           | 粘贴一行               |
| nyy         | 复制n行                |
| np          | 粘贴n行                |
| dd          | 删除1行                |
| ndd         | 删除n行                |
| gg  shift+6 | 回到文档开头           |
| GG  shift+4 | 回到文档结尾           |
| u           | 撤销更改               |

### 编辑模式 

| 命令             | 作用                       |
| ---------------- | -------------------------- |
| i                | 在光标当前位置插入内容     |
| a                | 在光标下一个位置输入内容   |
| o                | 在光标下一行输入内容       |
| 在英文状态下操作 | 在写内容的时候可以中文状态 |

### 底部命令模式 

| 命令                          | 作用                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| 在英文状态                    |                                                              |
| :                             | 进入底部命令模式                                             |
| wq                            | 保存并推出                                                   |
| q                             | 不保存退出                                                   |
|                               | 强制                                                         |
| :set nu                       | 显示行号                                                     |
| :set nonu                     | 取消显示行号                                                 |
| :行号                         | 将光标定位到制定的行号                                       |
| /                             | n  下一个  从上往下查找   shift +n(上一个) 从下往上查找      |
| ?                             | n 下一个  从下往上查找(上一个)     shift+n 从上往下查找(下一个) |
| :s/查找的目标/要替换的内容    | 只替换当前行                                                 |
| :s/查找的目标/要替换的内容/g  | 当前行的所有的关键词 全部被替换                              |
| :%s/查找的目标/要替换的内容   | 匹配全局 但是 只是一部分                                     |
| :%s/查找的目标/要替换的内容/g | 匹配全局所有的关键词                                         |

```shell
:%s/http:\/\/www.google.com\/index.php/https:\/\/www.caoliu.com\/video.avi    

特殊符号记得转义   
```

### Linux用户管理  

```shell
添加用户  useradd  用户名
     
删除用户 userdel 用户名  此时只是删除 /etc/passwd  一条记录 home 目录下 用户名为命名的目录还在 
userdel  -r 用户名   删除 /etc/passwd 记录的同时 删除 home 目录下  用户名为命名的目录  

修改密码  passwd  用户名  不写用户名 默认 root   

切换 用户   su 用户名  不写  默认切换到root  

修改用户名  usermod  -l 新用户名   原来的用户名  

添加完用户  会在  /home 目录下 生成一个  以 用户名为命名的 目录    
还会在  /etc/passwd 下面 产生一条记录    

root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
yinshanshan:x:500:501::/home/yinshanshan:/bin/bash

第一部分: yinshanshan 用户名
第二部分:x 用户的密码   
第三部分:用户的id  
第四部分:组id  
第五部分: 空白  注释  
第六部分: /home/yinshanshan 表示用户的家目录  
第七部分: /bin/bash  用户具备脚本执行的权限    简单说 这个用户可以登录 /sbin/nologin 用户不具备脚本执行的权限   也就是 用户不可以登录     

添加一个用户  不让他 登录   
useradd test -s /sbin/nologin  
```

### 用户组管理  

```shell
groupadd 组名  #添加用户组    
#会在  /etc/group下面生成一条记录    
wenhai:x:503:    
#组名   组密码  组id 
groupdel 组名 # 删除组     
groupmod -n 新组名  原来的组名   #修改组名
usermod  -l 新用户名   原来的用户名  

usermod  -g  用户组   用户名  #将用户从原来的组  加入到新的组 
useradd -g 用户组  用户名    #添加用户 直接把他加入到指定的组里   

gpasswd  -a 用户名  组名    #将用户加入到临时的组中  主组保持不变   
gpasswd  -d 用户名  组名    #将用户从临时组中删除      

```

### 用户用户组密码配置文件  

```shell
/etc/passwd 
/etc/group 
/etc/shadow 
```

### df  列出整体磁盘的使用量    

```shell
df  默认以 kb为单位 
    -a  列出所有文件系统 
    -h 以最佳阅读体验查看  
    -m 以MB为单位 
    -k 以KB为单位
```

### du 查看文件及目录  对磁盘的占用情况  

```shell
du   默认 以KB 为单位  
    -a  列出所有文件系统 
    -h 以最佳阅读体验查看  
    -m 以MB为单位 
    -k 以KB为单位
```

### 查看内存   

```shell
free   
     -h 以最佳阅读体验阅读   
     swap  交换分区        
```

### gz  是gzip 的简称    

```shell
yum -y install  gzip 
gzip -h  #查看帮助
gzip 文件1 文件2 文件3 #可以批量压缩   源文件不存在了  生成.gz的压缩文件     
-f 强制压缩 
gzip 不能压缩目录   
gzip -d 1.php.gz 2.php.gz  #支持批量解压缩   

```

### bz2    bzip2 的简称   

```php
bzip2 -z 1.php 2.php 3.php# 可以批量压缩   源文件不存在了  生成.bz2的压缩文件
bzip2 -h #查看帮助    
bzip2 -d 1.php.bz2 2.php.bz2 3.php.bz2       #支持批量解压缩   

不支持压缩目录   
```

## xz  

yum -y install xz

```shell
xz -h #查看帮助  
xz -z  文件1 文件2 文件3 文件4  支持批量压缩   #源文件也不存在  生成.xz的压缩文件  也不支持压缩目录 
xz -d 1.php.xz 2.php.xz  
```

### 打包 解包  

```shell
tar 
   -c 打包   
   -x 解包
   -f 制定文件名  
   -t 列出归档内容 
   -v 可视化输出   
   
tar -cvf kangbazi.tar  1.php 2.php 3.php test #可以打包目录  也可以打包文件   源文件还在 
tar -xvf kangbazi.tar  #解包  
tar -tf kangbazi.tar  #查看包里的内容   
```

### gz的打包并压缩   

```shell
tar 
	-z
 tar -zcvf kangbazi.tar.gz 4.php 5.php 6.php haha   打包并压缩   生成一个  kangbazi.tar.gz  源文件还存在 
```

### gz的解包并解压缩  

```shell
tar 
	-z
	tar -zxvf kangbazi.tar.gz   #源文件还在  xxxxxxxxxx tar     -z    tar -zxvf kangbazi.tar.gz   #源文件还在  gz的解包并解压缩  
```

### bz2的 打包并压缩  

```shell
tar 
-j
 tar -jcvf kangbazi.tar.bz2 4.php 5.php 6.php haha   打包并压缩   生成一个  kangbazi.tar.bz2  源文件还存在 
```

### bz2的解包并解压缩  

```shell
tar 
	-j
	tar -jxvf kangbazi.tar.bz2   #源文件还在  
```

### xz的 打包并压缩

```shell
tar 
	-J
	tar -Jcvf test2.tar.xz 4.php 5.php 6.php haha  打包并压缩   生成一个  test2.tar.xz  源文件还存在 
```

### xz的解包并解压缩  

```shell
tar 
    -J  
    tar -Jxvf test2.tar.xz 
```

### wget 递归扒站    

```shell
-c  断点续传   
-r,  指定递归下载。
-k  将页面中的连接转化为相对连接也就是本地链接   
-p, 下载所有用于显示 HTML 页面的图片之类的元素。
-np, 不追溯至父目录。
-nc, 不要重复下载已存在的文件
```

### zip  unzip  

```shell
yum -y install zip  unzip  好比windows 安装  WinRAR  软件   
zip xibuguigu.zip 1.php 2.php 3.avi 4.jpg kangbazi/     可以压缩文件  也可压缩目录  原文还存在
unzip xibuguigu.zip
```


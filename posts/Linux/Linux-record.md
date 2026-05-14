---
title: Linux 日志分析命令
description: 一些Linux 日志分析命令
date: 2019-04-10 16:10:00
slug: Linux-record
image:
categories:
    - Linux
tags: ["Linux"]


---

## 日志分析命令    

* screen  
* grep 
* sort 
* wc
* cut
* awk   

# mysql 远程连接授权   

```
mysql -u root -p  

use mysql  

 grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;  

 flush privileges; 
 注意 防火墙
```

## screen  

```
在一个窗口里并行多个任务   互不影响    
yum -y install  screen    

screen -S 任务名称  

Ctrl+A +D 离开会话 任务并没结束    

screen -ls  列出所有的任务        
会显示 任务id   

想进入其中一个任务  

screen -r  任务id     

想结束任务    
 
screen -X -S 任务id  quit  
```

## grep  

```
-c 打印符合要求的行数  
-i 忽略大小写  
-n 在输出符合要求的行的同时 连同行号一起输出  
-v 打印不符合要求的行  
  grep -c 'root' /etc/passwd  #打印符合要求的行数    
 grep -nv 'root' /etc/passwd  #打印不符合要求的行  并显示行号   
 grep '[0-9]' /etc/inittab #匹配所有包含数字的行 
 grep -v '[0-9]' /etc/inittab 
 grep -v '^#' /etc/inittab   #匹配不包含#号的数据   
 grep -v '^#' /etc/inittab | grep -v '^$'  #排除掉 注释 及 空行   
 grep 'r..o' /etc/passwd #匹配的是同时有一个或者多个 r  o的行  
 grep 'ooo*' /etc/passwd  #匹配 oo ooo oooo 
 grep 'o\{2\}' /etc/passwd   
```

## wc   

```
wc -w 统计单词数   
   -l 显示行数    
   -c 字节数     
一般配合 grep awk 来使用       
```

## sort  

```
排序  一般也是配合 来使用  

sort  默认按照字母 排序  
sort -r 逆向排序 
-f 忽略大小写  
-b 忽略前面的空格部分  
-M 按照月份排序  
-n 按照纯数字排序 
-u 重复的显示一行   
-t 制定分割符  
-k  制定区间来排序  


cat /etc/passwd | sort -t ":" -k 3n   #以：为分隔符   制定第3个区间  进行排序   n 表示 纯数字排序  
cat /etc/passwd | sort -t ":" -k 3nr  #倒序排列   
cat /etc/passwd | sort -t ":" -k 1.2,1.3 #对第1个区间的第2到第三个字符 进行排序  
cat /etc/passwd | sort -t ":" -k 1.2,1.3 -k 3r #对第1个区间的第2到第三个字符 进行排序  同时对第3个区间进行反向排序     
cat /etc/passwd | sort -t ":" -k 1.2,1.3 -k 3r -u #对第1个区间的第2到第三个字符 进行排序  同时对第3个区间进行反向排序  然后去重  s

```

## uniq  unique      

```
去除文档中的重复行   经常跟sort 一起用   

uniq   -u 只显示唯一的行  
				[root@boy ~]# cat test.log | sort | uniq -u
                apple
                caomei
                huanggua
                juhua
                orange
 						banana出现多次 被忽略掉  
	   -c 计数   显示每行出现的次数    
	   		[root@boy ~]# cat test.log | sort | uniq -c 
     					1 apple
      					2 banana
                        1 caomei
                        1 huanggua
                        1 juhua
                        1 orange

	   -i 忽略大小写   
	   
	   

```

## cut   从文本文件 或者文件流中提取文件  

```shell
-d  指定分隔符
-f  值第几个区间 


cat /etc/passwd | grep root | cut -d ":" -f 3  #以：为分隔符  截取第3个区间   
cat /etc/passwd | grep root | cut -d ":" -f 3,5,6 #:为分隔符  截取第3个 第5个  第6个  区间     
cat /etc/passwd | grep root | cut -d ":" -f 1-3 #截取第1到第3个区间  
cat /etc/passwd | grep root | cut -d ":" -f 1- #截取1到最后

cat /etc/passwd | grep root | cut -d ":" -f 1-3,7 #截取第1到第3个 还有第7个区间   
```



## awk   

```shell
-F #制定分割符      
head -n 2 /etc/passwd | awk -F ":" '{print $1}'  #$0表示打印全部的区间  $1 表示打印第一个区间
head -n 2 /etc/passwd | awk -F ":" '{print $1"~"$3}' #打印第一个区间  和第三个区间  自定义连接符    
awk /root/ /etc/passwd #匹配关键词     
awk -F ':' '$1 ~/root/' /etc/passwd # 匹配第一个区间有root的   
awk -F ':' '/root/ {print $1 $3} /test/ {print $1 $3}' /etc/passwd  #同时匹配多个关键词     只有一对单引号    
awk -F ':' '$3==0' /etc/passwd #匹配第三列为0的 文本    
awk -F ':' '$7!="/sbin/nologin"' /etc/passwd  #匹配第七个区间不是 /sbin/nologin的   
awk -F ':' '$3 < $4' /etc/passwd  #匹配第三个区间的值小于第4个区间值得 文本行   
awk -F ':' '$3>5 && $3<7' /etc/passwd #匹配第三列的值  大于5 小于7 的    
awk -F ':' '$3>5 || $7 == "/bin/bash"' /etc/passwd #匹配第三列的值 大于5 或者 第七列的值 等于 /bin/bash  
 tail -n1 /etc/passwd | awk -F ':' '$1="taotao"'  #查看最后一行  并 修改第一个区间的值  原来的数据没变  生成新的内容  并输出         
slice(1,2) concat() 
slice(0);
$arr1 = []
$arr2 = [];
$arr3 = $arr1 ;
head -n 10 /etc/passwd | awk -F ':' '{$7=$3+$4;print $0}' >> /tmp/test.log  #将第七列的值变为 第3列+第4列的和  并全部输出  追加到制定的文件中    

cat /etc/passwd  | awk -F ':' '{(sum=sum+$3)};END {print sum}';  #求所有行的第三列的和      
```

## 题 目   找出 耗时最长的100条请求    

```
log_format main '$remote_addr^A$remote_user^A[$time_local]^A$request' '^A$status^A$body_bytes_sent^A$http_referer' '^A$http_user_agent^A$http_x_forwarded_for^A$request_time';
cat access.log|awk -F '^A''{print $10,"****",$4}'|sort -r | head -n 100 
```


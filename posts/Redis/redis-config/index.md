---
title: redis.conf配置参数说明
description: redis.conf一些参数说明
date: 2019-03-05 15:10:00
slug: redis config
image:
categories:
    - Redis
tags: ["Redis"]
---

今天熟悉下 Redis 的配置文件，看看redis.conf中有哪些配置项。

<!--more--> 

```php
# 按照指定的配置文件启动 
# ./redis-server /path/to/redis.conf

# 包含其它的redis配置文件 
# include /path/to/local.conf
# include /path/to/other.conf

# 加载模块
# loadmodule /path/to/my_module.so
# loadmodule /path/to/other_module.so

# 监听指定的网络接口 
bind 127.0.0.1

# 指定使用的端口号 
port 6379

# 此参数确定TCP连接中已完成队列（3次握手之后）的长度，应小于Linux系统的/proc/sys/net/core/somaxconn的值
# 此选项默认值为511，而Linux的somaxconn默认值为128，当并发量比较大且客户端反应缓慢的时候，可以同时提高这两个参数。
tcp-backlog 511

# 指定监听的socket，适用于unix环境
# unixsocket /tmp/redis.sock
# unixsocketperm 700

# 客户端空闲N秒后断开连接，参数0表示不启用 
timeout 0


# 指定ACKs的时间周期，单位为秒，值非0的情况表示将周期性的检测客户端是否可用
tcp-keepalive 300

################################# GENERAL 常规 #####################################
# 启用后台守护进程运行模式 
daemonize yes

# redis启动后的进程ID保存文件
pidfile /var/run/redis_6379.pid


# 指定服务器信息显示的等级，4个参数分别为debug\verbose\notice\warning 
loglevel notice

# 指定日志文件，默认是使用系统的标准输出 
logfile ""

# 是否启用将记录记载到系统日志功能，默认为不启用
# syslog-enabled no

# 若启用日志记录，则需要设置日志记录的身份
# syslog-ident redis

# 若启用日志记录，则需要设置日志facility，可取值范围为local0~local7，表示不同的日志级别
# syslog-facility local0

# 设置数据库的数量，默认启动时使用DB0，使用“select ”可以更换数据库 
databases 16

################################ SNAPSHOTTING 快照  ################################

# 将数据库保存在磁盘上：
# SNAPSHOTTING
# save 900 1 900秒后保存，至少有1个key被更改时才会触发
# save 300 10 300秒后保存，至少有10个key被更改时才会触发 
# save 60 10000 60秒后保存，至少有10000个key被更改时才会触发
save 900 1
save 300 10
save 60 10000

# 最近一次save操作失败则停止写操作 
stop-writes-on-bgsave-error yes

# 启用压缩
rdbcompression yes

# 启用CRC64校验码，当然这个会影响一部份性能 
rdbchecksum yes

# 指定存储数据的文件名 
dbfilename dump.rdb


# 指定工作目录，rdb文件和aof文件都会存放在这个目录中，默认为当前目录
dir ./

################################# REPLICATION 复制  #################################

# Redis的主从复制采用异步的方式进行。 
# 如果同步连接时slave端短暂的与master端断开了连接，那连接恢复后允许slave端与master端进行一次局部的再同步。 
# 主从复制是自动进行的，并不需要用户的介入，slave端会自动尝试重连master并进行数据同步。 


# 设置master端的IP与端口信息 
# slaveof <masterip> <masterport>


# 如果master端启用了密码保护（requirepass），那slave端就需要配置此选项 
# masterauth <master-password>

# 当slave端在主从复制的过程中与master端断开了连接，此时有2种处理方法：一种是继续提供服务即使数据可能不是最新的，另一种是对请求返回一个错误信息，默认配置是继续提供服务
slave-serve-stale-data yes

# 自redis 2.6版本开始，slave端默认为readonly
slave-read-only yes

# 主从同步支持两种策略，即disk和socket方式（socket方式尚不完善，还处于实验阶段）。
# 新的slave端和重连的salve端不允许去继续同步进程，这被称之为“完全同步”。
# 一个RDB文件从master端传到slave端，分为两种情况：
# 1、支持disk：master端将RDB file写到disk，稍后再传送到slave端； 
# 2、无磁盘diskless：master端直接将RDB file传到slave socket，不需要与disk进行交互。
# 无磁盘diskless方式适合磁盘读写速度慢但网络带宽非常高的环境。 
#  默认不使用diskless同步方式
repl-diskless-sync no


# 无磁盘diskless方式在进行数据传递之前会有一个时间的延迟，以便slave端能够进行到待传送的目标队列中，这个时间默认是5秒 
repl-diskless-sync-delay 5

# slave端向server端发送pings的时间区间设置，默认为10秒
# repl-ping-slave-period 10

# 设置超时时间 
# repl-timeout 60

# 是否启用TCP_NODELAY，如果启用则会使用少量的TCP包和带宽去进行数据传输到slave端，当然速度会比较慢；如果不启用则传输速度比较快，但是会占用比较多的带宽。
repl-disable-tcp-nodelay no


# 设置backlog的大小，backlog是一个缓冲区，在slave端失连时存放要同步到slave的数据，因此当一个slave要重连时，经常是不需要完全同步的，执行局部同步就足够了。backlog设置的越大，slave可以失连的时间就越长。
# repl-backlog-size 1mb


# 如果一段时间后没有slave连接到master，则backlog size的内存将会被释放。如果值为0则表示永远不释放这部份内存。 
# repl-backlog-ttl 3600

# slave端的优先级设置，值是一个整数，数字越小表示优先级越高。当master故障时将会按照优先级来选择slave端进行恢复，如果值设置为0，则表示该slave永远不会被选择。
slave-priority 100


# min-slaves-to-write 3
# 设置当一个master端的可用slave少于N个，延迟时间大于M秒时，不接收写操作。
# min-slaves-max-lag 10

# 有slave端连接时是否需要密码验证
# requirepass foobared

# 同一时间内最大clients连接的数量，超过数量的连接会返回一个错误信息
# maxclients 10000

############################## MEMORY MANAGEMENT 内存管理 ################################

# 设置最大内存
# maxmemory <bytes>

# 如果内存使用量到达了最大内存设置，有以下几种处理方法
# volatile-lru -> Evict using approximated LRU among the keys with an expire set.
# allkeys-lru -> Evict any key using approximated LRU.
# volatile-lfu -> Evict using approximated LFU among the keys with an expire set.
# allkeys-lfu -> Evict any key using approximated LFU.
# volatile-random -> Remove a random key among the ones with an expire set.
# allkeys-random -> Remove a random key, any key.
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
# noeviction -> Don't evict anything, just return an error on write operations.
#
# LRU means Least Recently Used
# LFU means Least Frequently Used
#

# 默认的设置
# maxmemory-policy noeviction


# LRU算法检查的keys个数
# maxmemory-samples 5

############################# LAZY FREEING ####################################

lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
slave-lazy-flush no

############################## APPEND ONLY MODE ###############################

# 启用AOF模式
appendonly no

# 设置AOF记录的文件名
appendfilename "appendonly.aof"


# 向磁盘进行数据刷写的频率，有3个选项：
# always 有新数据则马上刷写，速度慢但可靠性高
# everysec 每秒钟刷写一次，折衷方法，所谓的redis可以只丢失1秒钟的数据就是源于此处
# no 按照OS自身的刷写策略来进行，速度最快 

# appendfsync always
appendfsync everysec
# appendfsync no

# 使用选项来进行设置 appendfsync everysec
# 当主进程在进行向磁盘的写操作时，将会阻止其它的fsync调用 
no-appendfsync-on-rewrite no

#  aof文件触发自动rewrite的百分比，值为0则表示禁用自动rewrite 
auto-aof-rewrite-percentage 100
# aof文件触发自动rewrite的最小文件size 
auto-aof-rewrite-min-size 64mb

# 是否加载不完整的aof文件来进行启动
aof-load-truncated yes


# 设置lua脚本的最大运行时间，单位为毫秒
lua-time-limit 5000

################################ REDIS CLUSTER  ###############################

# 一个正常的redis实例是不能做为一个redis集群的节点的，除非它是以一个集群节点的方式进行启动。
# 配置redis做为一个集群节点来启动 
# cluster-enabled yes


# 每个集群节点都有一个集群配置文件，这个文件不需要编辑，它由redis节点来创建和更新。每个redis节点的集群配置文件不可以相同。
# cluster-config-file nodes-6379.conf


# 设置集群节点超时时间，如果超过了指定的超时时间后仍不可达，则节点被认为是失败状态，单位为毫秒。
# cluster-node-timeout 15000


# 一个属于失效的master端的slave，如果它的数据较旧，将不会启动failover。
# 现在来讲并没有一个简单的方法去解决如何判定一个slave端的数据的时效性问题，所以可以执行以下两个选择：
# 1、如果有多个slave可用于failover，它们会交换信息以便选出一个最优的进行主从复制的offset，slave端会尝试依据offset去获取每个slave的rank，这样在启动failover时对每个slave的利用就与slave端的rank成正比。 
# 2、每个slave端和它的master端进行最后交互的时间，这可能是最近的ping或指令接收时间，或自与master端失连的过时时间。如果最近的交互时间太久，slave就不会尝试去进行failover。 
# 第2点可以由用户来进行调整，明确一个slave不会进行failover。自最近一次与master端进行交互，过时时间有一个计算公式： 
#（node-timeout * slave-validity-factor）+repl-ping-slave-period 
# 一个比较大的slave-validity-factor参数能够允许slave端使用比较旧的数据去failover它的master端，而一个比较小的值可能会阻止集群去选择slave端。
# 为获得最大的可用性，可以设置slave-validity-factor的值为0，这表示slave端将会一直去尝试failover它的master端而不管它与master端的最后交互时间。
# 默认值为10
# cluster-slave-validity-factor 10


# 集群中的slave可以迁移到那些没有可用slave的master端，这提升了集群处理故障的能力。毕竟一个没有slave的master端如果发生了故障是没有办法去进行failover的。
# 要将一个slave迁移到别的master，必须这个slave的原master端有至少给定数目的可用slave才可以进行迁移，这个给定的数目由migration barrier参数来进行设置，默认值为1，表示这个要进行迁移的slave的原master端应该至少还有1个可用的slave才允许其进行迁移，要禁用这个功能只需要将此参数设置为一个非常大的值。
# 
# cluster-migration-barrier 1


# 默认情况下当redis集群节点发现有至少一个hashslot未被covered时将会停止接收查询。
# 这种情况下如果有一部份的集群down掉了，那整个集群将变得不可用。
# 集群将会在所有的slot重新covered之后自动恢复可用。
# 若想要设置集群在部份key space没有cover完成时继续去接收查询，就将参数设置为no。
# cluster-require-full-coverage yes

################################## SLOW LOG ###################################

# redis的slow log是一个系统OS进行的记录查询，它是超过了指定的执行时间的。执行时间不包括类似与client进行交互或发送回复等I/O操作，它只是实际执行指令的时间。
# 有2个参数可以配置，一个用来告诉redis执行时间，这个时间是微秒级的（1秒=1000000微秒），这是为了不遗漏命令。
# 另一个参数是设置slowlog的长度，当一个新的命令被记录时，最旧的命令将会从命令记录队列中移除。
slowlog-log-slower-than 10000

# 可以使用“slowlog reset”命令来释放slowlog占用的内存。

slowlog-max-len 128

################################ LATENCY MONITOR ##############################

# 延迟监控，用于记录等于或超过了指定时间的操作，默认是关闭状态，即值为0。
latency-monitor-threshold 0

############################# EVENT NOTIFICATION ##############################

# 事件通知，默认不启用，具体参数查看配置文件
notify-keyspace-events ""

############################### ADVANCED CONFIG ###############################
# 当条目数量较少且最大不会超过给定阀值时，哈希编码将使用一个很高效的内存数据结构，阀值由以下参数来进行配置。

hash-max-ziplist-entries 512
hash-max-ziplist-value 64

# 与哈希类似，少量的lists也会通过一个指定的方式去编码从而节省更多的空间，它的阀值通过以下参数来进行配置。 
list-max-ziplist-size -2


# 集合sets在一种特殊的情况时有指定的编码方式，这种情况是集合由一组10进制的64位有符号整数范围内的数字组成的情况。以下选项可以设置集合使用这种特殊编码方式的size限制。
set-max-intset-entries 512

# 与哈希和列表类似，有序集合也会使用一种特殊的编码方式来节省空间，这种特殊的编码方式只用于这个有序集合的长度和元素均低于以下参数设置的值时。 

zset-max-ziplist-entries 128
zset-max-ziplist-value 64


# 设置HyeperLogLog的字节数限制，这个值通常在0~15000之间，默认为3000，基本不超过16000 
hll-sparse-max-bytes 3000


# redis将会在每秒中抽出10毫秒来对主字典进行重新散列化处理，这有助于尽可能的释放内存
activerehashing yes


# 因为某些原因，client不能足够快的从server读取数据，那client的输出缓存限制可能会使client失连，这个限制可用于3种不同的client种类，分别是：normal、slave和pubsub。
# normal -> normal clients including MONITOR clients
# slave  -> slave clients
# pubsub -> clients subscribed to at least one pubsub channel or pattern

client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

# redis使用一个内部程序来处理后台任务，例如关闭超时的client连接，清除过期的key等等。它并不会同时处理所有的任务，redis通过指定的hz参数去检查和执行任务。
# hz默认设为10，提高它的值将会占用更多的cpu，当然相应的redis将会更快的处理同时到期的许多key，以及更精确的去处理超时。
# hz的取值范围是1~500，通常不建议超过100，只有在请求延时非常低的情况下可以将值提升到100。 
hz 10

# 当一个子进程要改写AOF文件，如果以下选项启用，那文件将会在每产生32MB数据时进行同步，这样提交增量文件到磁盘时可以避免出现比较大的延迟。
aof-rewrite-incremental-fsync yes

```

# 🦖
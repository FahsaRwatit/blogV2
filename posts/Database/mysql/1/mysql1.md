

MySQL官方社区版：https://dev.mysql.com/downloads/mysql/

MySQL常见发行版本：

- Percona MySQL： https://www.percona.com/downloads/Percona-Server-LATEST/
- MariaDB：https://downloads.mariadb.org/

各个发行版本之间的区别及优缺点：

| 对比       | MySQL                      | Percona MySQL         | MariaDB              |
| ---------- | -------------------------- | --------------------- | -------------------- |
| 服务器特性 | 开源                       | 开源                  | 开源                 |
|            | 支持分区表                 | 支持分区表            | 支持分区表           |
|            | InnoDB                     | XtraDB                | XtraDB               |
|            | 企业监控工具，社区版不提供 | PerCcon Monitor工具   | Monyog               |
| 安全特性   | 企业版防火墙               | Proxy FireWall        | MaxScale FireWall    |
|            | 企业版用户审计             | 审计日志              | 审计日志             |
|            | 用户密码生命周期           | 用户密码生命周期      | -                    |
|            | sha256_password            | sha256_password       | sha256_password      |
|            | caching_sha2_password      | caching_sha2_password | ed25519              |
| 开发及管理 | 窗口函数(8.0)              | 窗口函数(8.0)         | 窗口函数(10.2)       |
|            | -                          | -                     | 支持日志回滚         |
|            | -                          | -                     | 支持记录表中记录修改 |
|            | Super read_only            | Super read_only       | -                    |



### 对MySQL进行升级

MySQL升级前需要考虑的问题：

- 升级可以给业务带来的益处
- 升级可能对业务带来的影响
- 数据库升级方案的制定
- 升级失败的回滚方案

1、升级可以给业务带来的益处

- 是否可以解决业务上某一方面的痛点
- 是否解决运维上某一方面的痛点

2、升级可能对业务带来的影响

- 对原业务程序的支持是否有影响
- 对原业务程序的性能是否有影响

3、数据库升级方案的制定

- 评估受影响的业务系统
- 升级的详细步骤
- 升级后的数据库环境检查
- 升级后的业务检查

4、升级失败的回滚方案

- 升级失败回滚的步骤
- 回滚后的数据库环境检查
- 回滚后的业务检查

数据库升级的步骤

- 对待升级数据库进行备份
- 升级Slave服务器版本
- 手动进行主从切换
- 升级Master服务器版本
- 升级完后进行业务检查

MySQL8.0版本主要的新特性

| 功能类别   | 新特性                                   |
| ---------- | ---------------------------------------- |
| 服务器功能 | 所有元数据使用InnoDB引擎存储，无frm文件  |
|            | 系统表采用InnoDB存储并采用独立表空间     |
|            | 支持定义资源管理组（目前仅支持CPU资源）  |
|            | 支持不可见索引和降序索引，支持直方图优化 |
|            | 支持窗口函数                             |
|            | 支持在线修改全局参数持久化               |
| 用户及安全 | 默认使用caching_sha2_password认证插件    |
|            | 新增支持定义角色（role）                 |
|            | 新增密码历史记录功能，限制重复使用密码   |
| InnoDB功能 | InnoDB DDL语句支持原子操作               |
|            | 支持在线修改undo表空间                   |
|            | 新增管理视图用于监控InnoDB表状态         |
|            | 新增innodb_dedicated_server配置项        |



### 用户管理类常见问题

- 如何在给定场景下为某用户授权？
- 如何保证数据账号的安全？
- 如何从一个实例迁移数据库账号到另一个实例？

#### 如何在给定场景下为某用户授权？

- 如何定义MySQL数据库账号？
- MySQL常用的用户权限
- 如何为用户授权？

如何定义MySQL数据库账号？

- 用户名@可访问控制列表
  - 1、%：戴白哦可以从所有外部主机访问
  - 2、192.168.1.%：表示可以从192.168.1网段访问
  - locahost:DB服务器本地访问
- 使用CREATE USER命令建立用户

MySQL常用的用户权限

| -     | 语句         | 说明                 |
| ----- | ------------ | -------------------- |
| Admin | Create User  | 建立新的用户的权限   |
|       | Grant option | 为其他用户授权的权限 |
|       | Super        | 管理服务器的权限     |
| DDL   | Create       | 新建数据库，表的权限 |
|       | Alter        | 修改表结构的权限     |
|       | Drop         | 删除数据库和表的权限 |
|       | Index        | 建立和删除索引的权限 |
| DML   | Select       | 查询表中数据的权限   |
|       | Insert       | 向表中插入数据的权限 |
|       | Update       | 更新表中数据的权限   |
|       | Delete       | 删除表中数据的权限   |
|       | Execute      | 执行存储过程的权限   |

#### 如何为用户授权

- 遵循最小权限原则
- 使用Grant命令对用户授权

```mysql
grant select,insert,update,delete on db.tb to user@ip;
revoke delete on db.tb from user@ip;
```

### 如何保证数据账号的安全？


























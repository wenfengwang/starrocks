---
displayed_sidebar: English
---

# 负载均衡

在部署多个前端（FE）节点时，用户可以在这些FE节点上层部署负载均衡层，以实现高可用性。

以下是一些高可用性的选项：

## 代码方式

一种方式是在应用层实现代码，以执行重试和负载均衡操作。例如，当一个连接断开时，系统会自动在其他连接上进行重试。采用这种方法需要用户配置多个FE节点的地址。

## JDBC连接器

JDBC连接器支持自动重试功能。

```sql
jdbc:mysql:loadbalance://[host1][:port],[host2][:port][,[host3][:port]]...[/[database]][?propertyName1=propertyValue1[&propertyName2=propertyValue2]...]
```

## ProxySQL

ProxySQL是一个MySQL代理层，支持读写分离、查询路由、SQL缓存、动态负载配置、故障转移以及SQL过滤。

StarRocks的前端（FE）负责处理连接和查询请求，它具备水平扩展和高可用性的特点。然而，FE需要用户在其上设置一个代理层，以实现自动负载均衡。具体设置步骤如下：

### 1. 安装相关依赖项

```shell
yum install -y gnutls perl-DBD-MySQL perl-DBI perl-devel
```

### 2. 下载安装包

```shell
wget https://github.com/sysown/proxysql/releases/download/v2.0.14/proxysql-2.0.14-1-centos7.x86_64.rpm
```

### 3. 将安装包解压到当前目录

```shell
rpm2cpio proxysql-2.0.14-1-centos7.x86_64.rpm | cpio -ivdm
```

### 4. 修改配置文件

```shell
vim ./etc/proxysql.cnf 
```

确保配置文件指向用户有权限访问的目录（绝对路径）：

```vim
datadir="/var/lib/proxysql"
errorlog="/var/lib/proxysql/proxysql.log"
```

### 5. 启动服务

```shell
./usr/bin/proxysql -c ./etc/proxysql.cnf --no-monitor
```

### 6. 登录系统

```shell
mysql -u admin -padmin -h 127.0.0.1 -P6032
```

### 7. 配置全局日志

```shell
SET mysql-eventslog_filename='proxysql_queries.log';
SET mysql-eventslog_default_log=1;
SET mysql-eventslog_format=2;
LOAD MYSQL VARIABLES TO RUNTIME;
SAVE MYSQL VARIABLES TO DISK;
```

### 8. 添加主节点

```sql
insert into mysql_servers(hostgroup_id, hostname, port) values(1, '172.26.92.139', 8533);
```

### 9. 添加观察者节点

```sql
insert into mysql_servers(hostgroup_id, hostname, port) values(2, '172.26.34.139', 9931);
insert into mysql_servers(hostgroup_id, hostname, port) values(2, '172.26.34.140', 9931);
```

### 10. 载入配置

```sql
load mysql servers to runtime;
save mysql servers to disk;
```

### 11. 设置用户名和密码

```sql
insert into mysql_users(username, password, active, default_hostgroup, backend, frontend) values('root', '*94BDCEBE19083CE2A1F959FD02F964C7AF4CFC29', 1, 1, 1, 1);
```

### 12. 载入配置

```sql
load mysql users to runtime; 
save mysql users to disk;
```

### 13. 编写代理规则

```sql
insert into mysql_query_rules(rule_id, active, match_digest, destination_hostgroup, mirror_hostgroup, apply) values(1, 1, '.', 1, 2, 1);
```

### 14. 载入配置

```sql
load mysql query rules to runtime; 
save mysql query rules to disk;
```

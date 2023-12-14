---
displayed_sidebar: "English"
---

# 负载均衡

当部署多个 FE 节点时，用户可以在 FE 的顶部部署负载均衡层以实现高可用性。

以下是一些高可用性选项：

## 代码方法

一种方法是在应用层实现代码以执行重试和负载平衡。例如，如果连接中断，它将自动重试其他连接。此方法要求用户配置多个 FE 节点地址。

## JDBC Connector

JDBC 连接器支持自动重试：

~~~sql
jdbc:mysql:loadbalance://[host1][:port],[host2][:port][,[host3][:port]]...[/[database]][?propertyName1=propertyValue1[&propertyName2=propertyValue2]...]
~~~

## ProxySQL

ProxySQL 是支持读写分离、查询路由、SQL 缓存、动态负载配置、故障转移和 SQL 过滤的 MySQL 代理层。

StarRocks FE 负责接收连接和查询请求，它具有水平伸缩性和高可用性。但是 FE 需要用户在其顶部设置代理层以实现自动负载平衡。请参阅以下设置步骤：

### 1. 安装相关依赖项

~~~shell
yum install -y gnutls perl-DBD-MySQL perl-DBI perl-devel
~~~

### 2. 下载安装包

~~~shell
wget https://github.com/sysown/proxysql/releases/download/v2.0.14/proxysql-2.0.14-1-centos7.x86_64.rpm
~~~

### 3. 解压缩到当前目录

~~~shell
rpm2cpio proxysql-2.0.14-1-centos7.x86_64.rpm | cpio -ivdm
~~~

### 4. 修改配置文件

~~~shell
vim ./etc/proxysql.cnf 
~~~

定位到用户具有访问权限的目录（绝对路径）：

~~~vim
datadir="/var/lib/proxysql"
errorlog="/var/lib/proxysql/proxysql.log"
~~~

### 5. 启动

~~~shell
./usr/bin/proxysql -c ./etc/proxysql.cnf --no-monitor
~~~

### 6. 登录

~~~shell
mysql -u admin -padmin -h 127.0.0.1 -P6032
~~~

### 7. 配置全局日志

~~~shell
SET mysql-eventslog_filename='proxysql_queries.log';
SET mysql-eventslog_default_log=1;
SET mysql-eventslog_format=2;
LOAD MYSQL VARIABLES TO RUNTIME;
SAVE MYSQL VARIABLES TO DISK;
~~~

### 8. 插入到领头节点

~~~sql
insert into mysql_servers(hostgroup_id, hostname, port) values(1, '172.26.92.139', 8533);
~~~

### 9. 插入观察者节点

~~~sql
insert into mysql_servers(hostgroup_id, hostname, port) values(2, '172.26.34.139', 9931);
insert into mysql_servers(hostgroup_id, hostname, port) values(2, '172.26.34.140', 9931);
~~~

### 10. 加载配置

~~~sql
load mysql servers to runtime;
save mysql servers to disk;
~~~

### 11. 配置用户名和密码

~~~sql
insert into mysql_users(username, password, active, default_hostgroup, backend, frontend) values('root', '*94BDCEBE19083CE2A1F959FD02F964C7AF4CFC29', 1, 1, 1, 1);
~~~

### 12. 加载配置

~~~sql
load mysql users to runtime; 
save mysql users to disk;
~~~

### 13. 写入代理规则

~~~sql
insert into mysql_query_rules(rule_id, active, match_digest, destination_hostgroup, mirror_hostgroup, apply) values(1, 1, '.', 1, 2, 1);
~~~

### 14. 加载配置

~~~sql
load mysql query rules to runtime; 
save mysql query rules to disk;
~~~
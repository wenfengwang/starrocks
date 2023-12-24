---
displayed_sidebar: English
---

# 介绍

您可以使用流加载或例程加载来导入半结构化数据（例如 JSON）。

## 使用场景

* 流加载：对于存储在文本文件中的 JSON 数据，使用流加载进行导入。
* 例程加载：对于 Kafka 中的 JSON 数据，使用例程加载进行导入。

### 流加载导入

示例数据：

~~~json
{ "id": 123, "city" : "beijing"},
{ "id": 456, "city" : "shanghai"},
    ...
~~~

示例：

~~~shell
curl -v --location-trusted -u <username>:<password> \
    -H "format: json" -H "jsonpaths: [\"$.id\", \"$.city\"]" \
    -T example.json \
    http://FE_HOST:HTTP_PORT/api/DATABASE/TABLE/_stream_load
~~~

`format: json` 参数允许您执行导入数据的格式。`jsonpaths` 用于执行相应的数据导入路径。

相关参数：

* jsonpaths：选择每列的 JSON 路径
* json\_root：选择开始解析 JSON 的列
* strip\_outer\_array：裁剪最外层的数组字段
* strict\_mode：在导入过程中严格筛选列类型转换

当 JSON 数据 schema 和 StarRocks 数据 schema 不完全相同时，修改 `Jsonpath`。

示例数据：

~~~json
{"k1": 1, "k2": 2}
~~~

导入示例：

~~~bash
curl -v --location-trusted -u <username>:<password> \
    -H "format: json" -H "jsonpaths: [\"$.k2\", \"$.k1\"]" \
    -H "columns: k2, tmp_k1, k1 = tmp_k1 * 100" \
    -T example.json \
    http://127.0.0.1:8030/api/db1/tbl1/_stream_load
~~~

导入过程中执行将 k1 乘以 100 的 ETL 操作，并将列与原始数据进行匹配 `Jsonpath`。

导入结果如下：

~~~plain text
+------+------+
| k1   | k2   |
+------+------+
|  100 |    2 |
+------+------+
~~~

对于缺少的列，如果列定义可以为 null，则将添加 `NULL`，或者可以通过 `ifnull` 添加默认值。

示例数据：

~~~json
[
    {"k1": 1, "k2": "a"},
    {"k1": 2},
    {"k1": 3, "k2": "c"},
]
~~~

导入示例-1：

~~~shell
curl -v --location-trusted -u <username>:<password> \
    -H "format: json" -H "strip_outer_array: true" \
    -T example.json \
    http://127.0.0.1:8030/api/db1/tbl1/_stream_load
~~~

导入结果如下：

~~~plain text
+------+------+
| k1   | k2   |
+------+------+
|    1 | a    |
+------+------+
|    2 | NULL |
+------+------+
|    3 | c    |
+------+------+
~~~

导入示例-2：

~~~shell
curl -v --location-trusted -u <username>:<password> \
    -H "format: json" -H "strip_outer_array: true" \
    -H "jsonpaths: [\"$.k1\", \"$.k2\"]" \
    -H "columns: k1, tmp_k2, k2 = ifnull(tmp_k2, 'x')" \
    -T example.json \
    http://127.0.0.1:8030/api/db1/tbl1/_stream_load
~~~

导入结果如下：

~~~plain text
+------+------+
| k1   | k2   |
+------+------+
|    1 | a    |
+------+------+
|    2 | x    |
+------+------+
|    3 | c    |
+------+------+
~~~

### 例程加载导入

与流加载类似，Kafka 数据源的消息内容被视为完整的 JSON 数据。

1. 如果一条消息包含数组格式的多行数据，则所有行都将被导入，Kafka 的偏移量将仅递增 1。
2. 如果 Array 格式的 JSON 代表了多行数据，但由于 JSON 格式错误导致 JSON 解析失败，则错误行只会递增 1（假设解析失败，StarRocks 实际上无法确定它包含多少行数据，只能将错误数据记录为一行）。

### 使用 Canal 从 MySQL 导入 StarRocks，支持增量同步 binlog

[Canal](https://github.com/alibaba/canal) 是阿里巴巴开源的 MySQL binlog 同步工具，通过它可以将 MySQL 数据同步到 Kafka。数据在 Kafka 中以 JSON 格式生成。这里演示了如何在 Kafka 中使用例程加载同步数据，实现与 MySQL 的增量数据同步。

* 在 MySQL 中，我们有一个数据表，其中包含以下表创建语句。

~~~sql
CREATE TABLE `query_record` (
  `query_id` varchar(64) NOT NULL,
  `conn_id` int(11) DEFAULT NULL,
  `fe_host` varchar(32) DEFAULT NULL,
  `user` varchar(32) DEFAULT NULL,
  `start_time` datetime NOT NULL,
  `end_time` datetime DEFAULT NULL,
  `time_used` double DEFAULT NULL,
  `state` varchar(16) NOT NULL,
  `error_message` text,
  `sql` text NOT NULL,
  `database` varchar(128) NOT NULL,
  `profile` longtext,
  `plan` longtext,
  PRIMARY KEY (`query_id`),
  KEY `idx_start_time` (`start_time`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8
~~~

* 前提条件：确保 MySQL 已启用 binlog，格式为 ROW。

~~~bash
[mysqld]
log-bin=mysql-bin # 启用 binlog
binlog-format=ROW # 选择 ROW 模式
server_id=1 # MySQL 复制需要定义，并且不要重复 canal 的 slaveId
~~~

* 创建一个帐户并授予辅助 MySQL 服务器的权限：

~~~sql
CREATE USER canal IDENTIFIED BY 'canal';
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
-- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%';
FLUSH PRIVILEGES;
~~~

* 然后下载并安装 Canal。

~~~bash
wget https://github.com/alibaba/canal/releases/download/canal-1.0.17/canal.deployer-1.0.17.tar.gz

mkdir /tmp/canal
tar zxvf canal.deployer-$version.tar.gz -C /tmp/canal
~~~

* 修改配置（MySQL 相关）。

`$ vi conf/example/instance.properties`

~~~bash
## mysql serverId
canal.instance.mysql.slaveId = 1234
#position info, need to change to your own database information
canal.instance.master.address = 127.0.0.1:3306
canal.instance.master.journal.name =
canal.instance.master.position =
canal.instance.master.timestamp =
#canal.instance.standby.address =
#canal.instance.standby.journal.name =
#canal.instance.standby.position =
#canal.instance.standby.timestamp =
#username/password, need to change to your own database information
canal.instance.dbUsername = canal  
canal.instance.dbPassword = canal
canal.instance.defaultDatabaseName =
canal.instance.connectionCharset = UTF-8
#table regex
canal.instance.filter.regex = .\*\\\\..\*
# 选择要同步的表的名称和 kafka 目标的分区名称。
canal.mq.dynamicTopic=databasename.query_record
canal.mq.partitionHash= databasename.query_record:query_id
~~~

* 修改配置（Kafka 相关）。

`$ vi /usr/local/canal/conf/canal.properties`

~~~bash
# 可用选项：tcp（默认），kafka，RocketMQ
canal.serverMode = kafka
# ...
# kafka/rocketmq 集群配置：192.168.1.117:9092,192.168.1.118:9092,192.168.1.119:9092
canal.mq.servers = 127.0.0.1:6667
canal.mq.retries = 0
# 此值可以在 flagMessage 模式下增加，但不要超过 MQ 消息的最大大小。
canal.mq.batchSize = 16384
canal.mq.maxRequestSize = 1048576
# 在 flatMessage 模式下，请将此值更改为较大的值，建议为 50-200。
canal.mq.lingerMs = 1
canal.mq.bufferMemory = 33554432
# Canal 的批量大小，默认值为 50K。 由于 Kafka 的最大消息大小限制（在 900K 以下），请不要超过 1M。
canal.mq.canalBatchSize = 50
# `Canal get` 的超时时间，以毫秒为单位。 空表示无限超时。
canal.mq.canalGetTimeout = 100
# 对象是否以 flat json 格式
canal.mq.flatMessage = false
canal.mq.compressionType = none
canal.mq.acks = all
# Kafka 消息传递是否使用事务
canal.mq.transaction = false
~~~

* 启动

`bin/startup.sh`

相应的同步日志显示在 `logs/example/example.log` 中，并且在 Kafka 中的格式如下：

~~~json
{
    "data": [{
        "query_id": "3c7ebee321e94773-b4d79cc3f08ca2ac",
        "conn_id": "34434",
        "fe_host": "172.26.34.139",
        "user": "zhaoheng",
        "start_time": "2020-10-19 20:40:10.578",
        "end_time": "2020-10-19 20:40:10",
        "time_used": "1.0",
        "state": "FINISHED",
        "error_message": "",
        "sql": "COMMIT",
        "database": "",
        "profile": "",
        "plan": ""
    }, {
        "query_id": "7ff2df7551d64f8e-804004341bfa63ad",
        "conn_id": "34432",
        "fe_host": "172.26.34.139",
        "user": "zhaoheng",
        "start_time": "2020-10-19 20:40:10.566",
        "end_time": "2020-10-19 20:40:10",
        "time_used": "0.0",
        "state": "FINISHED",
        "error_message": "",
        "sql": "COMMIT",
        "database": "",
        "profile": "",
        "plan": ""
    }, {
        "query_id": "3a4b35d1c1914748-be385f5067759134",
        "conn_id": "34440",
        "fe_host": "172.26.34.139",
        "user": "zhaoheng",
        "start_time": "2020-10-19 20:40:10.601",
        "end_time": "1970-01-01 08:00:00",
        "time_used": "-1.0",
        "state": "RUNNING",
        "error_message": "",
        "sql": " SELECT SUM(length(lo_custkey)), SUM(length(c_custkey)) FROM lineorder_str INNER JOIN customer_str ON lo_custkey=c_custkey;",
        "database": "ssb",
        "profile": "",
        "plan": ""
    }],
    "database": "center_service_lihailei",
    "es": 1603111211000,
    "id": 122,
    "isDdl": false,
    "mysqlType": {
        "query_id": "varchar(64)",
        "conn_id": "int(11)",
        "fe_host": "varchar(32)",
        "user": "varchar(32)",
        "start_time": "datetime(3)",
        "end_time": "datetime",
        "time_used": "double",
        "state": "varchar(16)",
        "error_message": "text",
        "sql": "text",
        "database": "varchar(128)",
        "profile": "longtext",
        "plan": "longtext"
    },
    "old": null,
    "pkNames": ["query_id"],
    "sql": "",
    "sqlType": {
        "query_id": 12,
        "conn_id": 4,
        "fe_host": 12,
        "user": 12,
        "start_time": 93,
        "end_time": 93,
        "time_used": 8,
        "state": 12,
        "error_message": 2005,
        "sql": 2005,
        "database": 12,
        "profile": 2005,
        "plan": 2005
    },
    "table": "query_record",
    "ts": 1603111212015,
    "type": "INSERT"
}
~~~

添加 `json_root` 和 `strip_outer_array = true` 以从 `data` 导入数据。

~~~sql
create routine load manual.query_job on query_record   
columns (query_id,conn_id,fe_host,user,start_time,end_time,time_used,state,error_message,`sql`,`database`,profile,plan)  
PROPERTIES (  
    "format"="json",  
    "json_root"="$.data",
    "desired_concurrent_number"="1",  
    "strip_outer_array" ="true",    
    "max_error_number"="1000" 
) 
FROM KAFKA (     
    "kafka_broker_list"= "172.26.92.141:9092",     
    "kafka_topic" = "databasename.query_record" 
);
~~~

这样就完成了从 MySQL 到 StarRocks 的近乎实时的数据同步。

通过 `show routine load` 查看导入作业的状态和错误消息。

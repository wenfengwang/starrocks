---
displayed_sidebar: English
---

# 介绍

您可以通过流加载或常规加载来导入半结构化数据（例如，JSON格式的数据）。

## 使用场景

* 流加载：用于将存储在文本文件中的JSON数据导入。
* 常规加载：用于将存储在Kafka中的JSON数据导入。

### 流加载导入

样例数据：

```json
{ "id": 123, "city" : "beijing"},
{ "id": 456, "city" : "shanghai"},
    ...
```

示例：

```shell
curl -v --location-trusted -u <username>:<password> \
    -H "format: json" -H "jsonpaths: [\"$.id\", \"$.city\"]" \
    -T example.json \
    http://FE_HOST:HTTP_PORT/api/DATABASE/TABLE/_stream_load
```

format:json 参数允许您指定导入数据的格式。jsonpaths 用于指定相应的数据导入路径。

相关参数：

* jsonpaths：为每一列选择 JSON 路径
* json_root：指定开始解析 JSON 的列
* strip_outer_array：去除最外层的数组字段
* strict_mode：在导入过程中对列类型转换进行严格筛选

当 JSON 数据的结构与 StarRocks 数据的结构不完全一致时，需要修改 Jsonpath。

样例数据：

```json
{"k1": 1, "k2": 2}
```

导入示例：

```bash
curl -v --location-trusted -u <username>:<password> \
    -H "format: json" -H "jsonpaths: [\"$.k2\", \"$.k1\"]" \
    -H "columns: k2, tmp_k1, k1 = tmp_k1 * 100" \
    -T example.json \
    http://127.0.0.1:8030/api/db1/tbl1/_stream_load
```

在导入过程中执行将 k1 乘以 100 的 ETL 操作，并通过 Jsonpath 将列与原始数据匹配。

导入结果如下：

```plain
+------+------+
| k1   | k2   |
+------+------+
|  100 |    2 |
+------+------+
```

对于缺失的列，如果列的定义允许为空值，则会添加 NULL，或者可以通过 ifnull 函数添加默认值。

样例数据：

```json
[
    {"k1": 1, "k2": "a"},
    {"k1": 2},
    {"k1": 3, "k2": "c"},
]
```

导入示例-1：

```shell
curl -v --location-trusted -u <username>:<password> \
    -H "format: json" -H "strip_outer_array: true" \
    -T example.json \
    http://127.0.0.1:8030/api/db1/tbl1/_stream_load
```

导入结果如下：

```plain
+------+------+
| k1   | k2   |
+------+------+
|    1 | a    |
+------+------+
|    2 | NULL |
+------+------+
|    3 | c    |
+------+------+
```

导入示例-2：

```shell
curl -v --location-trusted -u <username>:<password> \
    -H "format: json" -H "strip_outer_array: true" \
    -H "jsonpaths: [\"$.k1\", \"$.k2\"]" \
    -H "columns: k1, tmp_k2, k2 = ifnull(tmp_k2, 'x')" \
    -T example.json \
    http://127.0.0.1:8030/api/db1/tbl1/_stream_load
```

导入结果如下：

```plain
+------+------+
| k1   | k2   |
+------+------+
|    1 | a    |
+------+------+
|    2 | x    |
+------+------+
|    3 | c    |
+------+------+
```

### 常规加载导入

与流加载相似，将 Kafka 数据源的消息内容视为完整的 JSON 数据。

1. 如果一条消息包含以数组格式表示的多行数据，则所有行都将被导入，而 Kafka 的偏移量仅增加 1。
2. 如果一个以数组格式表示的 JSON 包含多行数据，但由于 JSON 格式错误导致解析失败，那么错误行数将仅增加 1（鉴于解析失败，StarRocks 实际上无法确定它包含多少行数据，只能将错误数据记录为一行）。

### 使用 Canal 从 MySQL 同步数据到 StarRocks 并进行增量同步 binlog

[Canal](https://github.com/alibaba/canal) 是阿里巴巴开源的 MySQL binlog 同步工具，通过它我们可以将 MySQL 数据同步到 Kafka。数据以 JSON 格式生成并存储在 Kafka 中。这里展示了如何使用常规加载同步 Kafka 中的数据，以实现与 MySQL 的增量数据同步。

* 在 MySQL 中，我们有一个数据表，其创建表语句如下。

```sql
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
```

* 前提条件：确保 MySQL 启用了 binlog 且格式为 ROW。

```bash
[mysqld]
log-bin=mysql-bin # Enable binlog
binlog-format=ROW # Select ROW mode
server_id=1 # MySQL replication need to be defined, and do not duplicate canal's slaveId
```

* 创建账户并授予从属 MySQL 服务器的权限：

```sql
CREATE USER canal IDENTIFIED BY 'canal';
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
-- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%';
FLUSH PRIVILEGES;
```

* 接着下载并安装 Canal。

```bash
wget https://github.com/alibaba/canal/releases/download/canal-1.0.17/canal.deployer-1.0.17.tar.gz

mkdir /tmp/canal
tar zxvf canal.deployer-$version.tar.gz -C /tmp/canal
```

* 修改配置（针对 MySQL）。

$ vi conf/example/instance.properties

```bash
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
# Select the name of the table to be synchronized and the partition name of the kafka target.
canal.mq.dynamicTopic=databasename.query_record
canal.mq.partitionHash= databasename.query_record:query_id
```

* 修改配置（针对 Kafka）。

$ vi /usr/local/canal/conf/canal.properties

```bash
# Available options: tcp(by default), kafka, RocketMQ
canal.serverMode = kafka
# ...
# kafka/rocketmq Cluster Configuration: 192.168.1.117:9092,192.168.1.118:9092,192.168.1.119:9092
canal.mq.servers = 127.0.0.1:6667
canal.mq.retries = 0
# This value can be increased in flagMessage mode, but do not exceed the maximum size of the MQ message.
canal.mq.batchSize = 16384
canal.mq.maxRequestSize = 1048576
# In flatMessage mode, please change this value to a larger value, 50-200 is recommended.
canal.mq.lingerMs = 1
canal.mq.bufferMemory = 33554432
# Canal's batch size with a default value of 50K. Please do not exceed 1M due to Kafka's maximum message size limit (under 900K)
canal.mq.canalBatchSize = 50
# Timeout of `Canal get`, in milliseconds. Empty indicates unlimited timeout.
canal.mq.canalGetTimeout = 100
# Whether the object is in flat json format
canal.mq.flatMessage = false
canal.mq.compressionType = none
canal.mq.acks = all
# Whether Kafka message delivery uses transactions
canal.mq.transaction = false
```

* 启动

bin/startup.sh

相应的同步日志显示在 logs/example/example.log 和 Kafka 中，格式如下：

```json
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
```

添加 json_root 和 strip_outer_array = true 参数以从 data 表导入数据。

```sql
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
```

这样就完成了从 MySQL 到 StarRocks 的近实时数据同步。

通过 show routine load 命令查看导入作业的状态和错误信息。

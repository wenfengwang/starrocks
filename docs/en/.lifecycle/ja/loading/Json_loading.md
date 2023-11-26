---
displayed_sidebar: "Japanese"
---

# イントロダクション

ストリームロードまたはルーチンロードを使用して、セミストラクチャ化されたデータ（例：JSON）をインポートできます。

## 使用シナリオ

* ストリームロード：テキストファイルに保存されたJSONデータの場合、ストリームロードを使用してインポートします。
* ルーチンロード：KafkaにあるJSONデータの場合、ルーチンロードを使用してインポートします。

### ストリームロードのインポート

サンプルデータ：

~~~json
{ "id": 123, "city" : "beijing"},
{ "id": 456, "city" : "shanghai"},
    ...
~~~

例：

~~~shell
curl -v --location-trusted -u <ユーザ名>:<パスワード> \
    -H "format: json" -H "jsonpaths: [\"$.id\", \"$.city\"]" \
    -T example.json \
    http://FE_HOST:HTTP_PORT/api/DATABASE/TABLE/_stream_load
~~~

`format: json`パラメータは、インポートデータの形式を実行するために使用します。`jsonpaths`は、対応するデータのインポートパスを実行するために使用されます。

関連するパラメータ：

* jsonpaths：各列のJSONパスを選択します
* json\_root：JSONのパースが開始される列を選択します
* strip\_outer\_array：最も外側の配列フィールドを切り取ります
* strict\_mode：インポート中の列の型変換に対する厳密なフィルタリングを行います

JSONデータのスキーマとStarRocksデータのスキーマが完全に一致しない場合は、`Jsonpath`を変更します。

サンプルデータ：

~~~json
{"k1": 1, "k2": 2}
~~~

インポートの例：

~~~bash
curl -v --location-trusted -u <ユーザ名>:<パスワード> \
    -H "format: json" -H "jsonpaths: [\"$.k2\", \"$.k1\"]" \
    -H "columns: k2, tmp_k1, k1 = tmp_k1 * 100" \
    -T example.json \
    http://127.0.0.1:8030/api/db1/tbl1/_stream_load
~~~

インポート結果は次のようになります：

~~~plain text
+------+------+
| k1   | k2   |
+------+------+
|  100 |    2 |
+------+------+
~~~

欠落している列の場合、列の定義がnull許容であれば、`NULL`が追加されます。または、`ifnull`を使用してデフォルト値を追加できます。

サンプルデータ：

~~~json
[
    {"k1": 1, "k2": "a"},
    {"k1": 2},
    {"k1": 3, "k2": "c"},
]
~~~

インポートの例1：

~~~shell
curl -v --location-trusted -u <ユーザ名>:<パスワード> \
    -H "format: json" -H "strip_outer_array: true" \
    -T example.json \
    http://127.0.0.1:8030/api/db1/tbl1/_stream_load
~~~

インポート結果は次のようになります：

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

インポートの例2：

~~~shell
curl -v --location-trusted -u <ユーザ名>:<パスワード> \
    -H "format: json" -H "strip_outer_array: true" \
    -H "jsonpaths: [\"$.k1\", \"$.k2\"]" \
    -H "columns: k1, tmp_k2, k2 = ifnull(tmp_k2, 'x')" \
    -T example.json \
    http://127.0.0.1:8030/api/db1/tbl1/_stream_load
~~~

インポート結果は次のようになります：

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

### ルーチンロードのインポート

ストリームロードと同様に、Kafkaデータソースのメッセージコンテンツは完全なJSONデータとして扱われます。

1. メッセージに複数の行のデータが配列形式で含まれている場合、すべての行がインポートされ、Kafkaのオフセットは1だけ増加します。
2. 配列形式のJSONが複数の行のデータを表しているが、JSONのパースがJSON形式のエラーにより失敗した場合、エラーローは1だけ増加します（パースが失敗するため、StarRocksは実際にどのくらいの行のデータを含んでいるかを判断することができず、エラーデータを1行として記録するしかありません）。

### MySQLの増分同期バイナリログを使用してStarRocksにインポートするためのCanalの使用

[Canal](https://github.com/alibaba/canal)は、アリババから提供されるオープンソースのMySQLバイナリログ同期ツールであり、MySQLデータをKafkaに同期することができます。データはKafkaでJSON形式で生成されます。ここでは、MySQLとの増分データ同期のためにKafkaのデータを使用してルーチンロードを使用する方法のデモンストレーションを行います。

* MySQLには、次のテーブル作成ステートメントを持つデータテーブルがあります。

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

* 前提条件：MySQLのバイナリログが有効になっており、フォーマットがROWになっていることを確認してください。

~~~bash
[mysqld]
log-bin=mysql-bin # バイナリログを有効にする
binlog-format=ROW # ROWモードを選択する
server_id=1 # MySQLのレプリケーションを定義する必要があり、canalのslaveIdと重複しないようにする
~~~

* アカウントを作成し、セカンダリMySQLサーバーに権限を付与します：

~~~sql
CREATE USER canal IDENTIFIED BY 'canal';
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
-- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%';
FLUSH PRIVILEGES;
~~~

* 次に、Canalをダウンロードしてインストールします。

~~~bash
wget https://github.com/alibaba/canal/releases/download/canal-1.0.17/canal.deployer-1.0.17.tar.gz

mkdir /tmp/canal
tar zxvf canal.deployer-$version.tar.gz -C /tmp/canal
~~~

* 設定を変更します（MySQL関連）。

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
# Select the name of the table to be synchronized and the partition name of the kafka target.
canal.mq.dynamicTopic=databasename.query_record
canal.mq.partitionHash= databasename.query_record:query_id
~~~

* 設定を変更します（Kafka関連）。

`$ vi /usr/local/canal/conf/canal.properties`

~~~bash
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
~~~

* 初期化

`bin/startup.sh`

対応する同期ログは`logs/example/example.log`に表示され、Kafkaでは次の形式で表示されます：

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

`json_root`と`strip_outer_array = true`を追加して、`data`からデータをインポートします。

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

これでMySQLからStarRocksへのデータのほぼリアルタイム同期が完了しました。

インポートジョブのステータスとエラーメッセージは、`show routine load`コマンドで表示できます。

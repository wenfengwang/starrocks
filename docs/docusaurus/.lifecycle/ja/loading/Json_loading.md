---
displayed_sidebar: "Japanese"
---

# はじめに

半構造化データ（例：JSON）をストリームロードまたはルーチンロードを使用してインポートできます。

## 使用シナリオ

* ストリームロード：テキストファイルに保存されたJSONデータをインポートするには、ストリームロードを使用します。
* ルーチンロード：KafkaにあるJSONデータをインポートするには、ルーチンロードを使用します。

### ストリームロードのインポート

サンプルデータ:

~~~json
{ "id": 123, "city" : "beijing"},
{ "id": 456, "city" : "shanghai"},
    ...
~~~

例:

~~~shell
curl -v --location-trusted -u <ユーザー名>:<パスワード> \
    -H "format: json" -H "jsonpaths: [\"$.id\", \"$.city\"]" \
    -T example.json \
    http://FE_HOST:HTTP_PORT/api/DATABASE/TABLE/_stream_load
~~~

`format: json`パラメータは、インポートされるデータの形式を実行することができます。`jsonpaths`は、対応するデータのインポートパスを実行するために使用されます。

関連パラメータ:

* jsonpaths: 各列のJSONパスを選択します
* json\_root: JSONの開始列を選択します
* strip\_outer\_array: 最も外側の配列フィールドを切り取ります
* strict\_mode: インポート中の列型変換に厳密にフィルタリングします

JSONデータのスキーマとStarRocksデータのスキーマが完全に一致しない場合は、`Jsonpath`を変更します。

サンプルデータ:

~~~json
{"k1": 1, "k2": 2}
~~~

インポート例:

~~~bash
curl -v --location-trusted -u <ユーザー名>:<パスワード> \
    -H "format: json" -H "jsonpaths: [\"$.k2\", \"$.k1\"]" \
    -H "columns: k2, tmp_k1, k1 = tmp_k1 * 100" \
    -T example.json \
    http://127.0.0.1:8030/api/db1/tbl1/_stream_load
~~~

インポート時にk1を100倍にするETL操作を実行し、`Jsonpath`によって元のデータと列が一致します。

インポート結果は以下の通りです:

~~~plain text
+------+------+
| k1   | k2   |
+------+------+
|  100 |    2 |
+------+------+
~~~

欠落している列について、列の定義がヌル可能であれば、`NULL`が追加されます。または、`ifnull`によってデフォルト値を追加できます。

サンプルデータ:

~~~json
[
    {"k1": 1, "k2": "a"},
    {"k1": 2},
    {"k1": 3, "k2": "c"},
]
~~~

インポート例-1:

~~~shell
curl -v --location-trusted -u <ユーザー名>:<パスワード> \
    -H "format: json" -H "strip_outer_array: true" \
    -T example.json \
    http://127.0.0.1:8030/api/db1/tbl1/_stream_load
~~~

インポート結果は以下の通りです:

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

インポート例-2:

~~~shell
curl -v --location-trusted -u <ユーザー名>:<パスワード> \
    -H "format: json" -H "strip_outer_array: true" \
    -H "jsonpaths: [\"$.k1\", \"$.k2\"]" \
    -H "columns: k1, tmp_k2, k2 = ifnull(tmp_k2, 'x')" \
    -T example.json \
    http://127.0.0.1:8030/api/db1/tbl1/_stream_load
~~~

インポート結果は以下の通りです:

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

1. メッセージが配列形式で複数のデータ行を含む場合、すべての行がインポートされ、Kafkaのオフセットは1だけ増加します。
2. 配列形式のJSONが複数のデータ行を表しているが、JSONの解析がJSON形式エラーのために失敗した場合、エラー行は1だけ増加します（解析が失敗したため、StarRocksは実際にどのくらいのデータ行が含まれているかを決定することができず、エラーデータを1行として記録することしかできません）。

### Canalを使用してMySQLからStarRocksにインクリメンタル同期ビンログをインポートする

[Canal](https://github.com/alibaba/canal)は、アリババのオープンソースMySQL binlog同期ツールで、MySQLデータをKafkaに同期することができます。データはKafkaでJSON形式で生成されます。ここでは、MySQLのインクリメンタルデータ同期のためにルーチンロードを使用してKafkaでデータを同期する方法を示します。

* MySQLには以下のテーブル作成文があるデータテーブルがあります。

~~~sql
CREATE TABLE `query_record` (
  `query_id` varchar(64) NOT NULL,
  `conn_id` int(11) DEFAULT NULL,
  `fe_host` varchar(32) DEFAULT NULL,
  `user` varchar(32) DEFAULT NULL,
  `start_time` datetime NOT NULL,
  `end_time` datetime DEFAULT NULL,
  ...
```markdown
# Canalのバッチサイズはデフォルトで50Kです。Kafkaの最大メッセージサイズ制限（900K未満）を超えないでください。
canal.mq.canalBatchSize = 50
# `Canal get`のタイムアウトはミリ秒単位です。空の場合はタイムアウトが無制限を表します。
canal.mq.canalGetTimeout = 100
# オブジェクトがフラットなJSON形式にあるかどうか
canal.mq.flatMessage = false
canal.mq.compressionType = none
canal.mq.acks = all
# Kafkaメッセージの配信にトランザクションを使用するかどうか
canal.mq.transaction  = false
~~~

* 初期化

`bin/startup.sh`

対応する同期ログは `logs/example/example.log` およびKafkaに、以下の形式で表示されます：

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
        "sql": "SELECT SUM(length(lo_custkey)), SUM(length(c_custkey)) FROM lineorder_str INNER JOIN customer_str ON lo_custkey=c_custkey;",
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

`json_root` および  `strip_outer_array = true` を追加してデータを `data` からインポートします。

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

これでMySQLからStarRocksへのほぼリアルタイムのデータ同期が完了しました。

インポートジョブのステータスとエラーメッセージは `show routine load` で表示できます。
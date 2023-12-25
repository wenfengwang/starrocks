---
displayed_sidebar: Chinese
---

# Flink Connectorのよくある質問

## トランザクションインターフェースのexactly-onceを使用した際に、インポートが失敗する

**問題の説明：**

```plaintext
com.starrocks.data.load.stream.exception.StreamLoadFailException: {
    "TxnId": 33823381,
    "Label": "502c2770-cd48-423d-b6b7-9d8f9a59e41a",
    "Status": "Fail",
    "Message": "timeout by txn manager",--具体的なエラーメッセージ
    "NumberTotalRows": 1637,
    "NumberLoadedRows": 1637,
    "NumberFilteredRows": 0,
    "NumberUnselectedRows": 0,
    "LoadBytes": 4284214,
    "LoadTimeMs": 120294,
    "BeginTxnTimeMs": 0,
    "StreamLoadPlanTimeMs": 7,
    "ReadDataTimeMs": 9,
    "WriteDataTimeMs": 120278,
    "CommitAndPublishTimeMs": 0
}
```

**原因分析：**

`sink.properties.timeout`がFlinkのcheckpoint intervalより小さいため、トランザクションがタイムアウトしました。

**解決策：**

timeoutをcheckpoint intervalより大きく設定する必要があります。

## flink-connector-jdbc_2.11を使用してStarRocksにデータをsinkすると、時間が8時間遅れる

**問題の説明：**

Flink内のlocaltimestap関数で生成された時間はFlink内では正常ですが、StarRocksにsinkした後、時間が8時間遅れていることが確認されました。FlinkとStarRocksがインストールされているサーバーのタイムゾーンは共にAsia/Shanghai（東八区）であることが確認されています。Flinkのバージョンは1.12で、ドライバーはflink-connector-jdbc_2.11ですが、どのように対処すればよいでしょうか？

**解決策：**

Flinkのsinkテーブルでタイムゾーンパラメータ'server-time-zone' = 'Asia/Shanghai'を設定するか、またはjdbc URLに&serverTimezone=Asia/Shanghaiを追加することができます。以下に例を示します：

```sql
CREATE TABLE sk (
    sid int,
    local_dtm TIMESTAMP,
    curr_dtm TIMESTAMP
)
WITH (
    'connector' = 'jdbc',
    'url' = 'jdbc:mysql://192.168.110.66:9030/sys_device?characterEncoding=utf-8&serverTimezone=Asia/Shanghai',
    'table-name' = 'sink',
    'driver' = 'com.mysql.jdbc.Driver',
    'username' = 'sr',
    'password' = 'sr123',
    'server-time-zone' = 'Asia/Shanghai'
);
```

## [Flinkインポート] StarRocksクラスターにデプロイされたKafkaクラスターのデータはインポートできるが、他のKafkaクラスターからはインポートできない

**問題の説明：**

```SQL
failed to query watermark offset, err: Local: Bad message format
```

**解決策：**

Kafka通信ではホスト名が使用されるため、StarRocksクラスターのノードにKafkaホスト名の解決を/etc/hostsで設定する必要があります。

## Beが要求したメモリがオペレーティングシステムに解放されない

これは正常な現象です。データベースがオペレーティングシステムから割り当てられた大量のメモリは、割り当て時に余分に予約され、解放時には遅延されることがあります。これは大量のメモリの割り当てコストが高いため、再利用を目的としています。テスト環境での検証時には、メモリ使用状況を監視し、長期間にわたってメモリが解放されるかどうかを確認することをお勧めします。

## flink connectorをダウンロードした後に有効にならない問題について

**問題の説明：**

このパッケージは、阿里云のミラーアドレスを通じて取得する必要があります。

**解決策:**

/etc/maven/settings.xmlのmirrorセクションが阿里云のミラーを通じて全て取得するように設定されているか確認してください。

もしそうなら、以下のように変更してください：

 <mirror>
    <id>aliyunmaven</id>
    <mirrorOf>central</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>

## Flink-connector-StarRocksのsink.buffer-flush.interval-msパラメータの意味

**問題の説明:**

```plain text
+----------------------+--------------------------------------------------------------+
|         Option       | Required |  Default   | Type   |       Description           |
+-------------------------------------------------------------------------------------+
|  sink.buffer-flush.  |  NO      |   300000   | String | the flushing time interval, |
|  interval-ms         |          |            |        | range: [1000ms, 3600000ms]  |
+----------------------+--------------------------------------------------------------+
```

このパラメータを15秒に設定した場合、しかしチェックポイント間隔が5分である場合、この値はまだ有効ですか？

**解決策:**

どの閾値に最初に到達するかにより、その閾値が最初に有効になります。これはcheckpoint intervalの設定値とは関係ありません。checkpoint intervalはexactly onceにのみ有効で、at_least_onceではinterval-msを使用します。

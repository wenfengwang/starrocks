---
displayed_sidebar: "Japanese"
---

# Flinkコネクタ

## flink-connector-jdbc_2.11sinkがStarRocksで8時間遅れる

**問題の説明:**

localtimestap関数によって生成された時間はFlinkでは正常ですが、StarRocksにシンクされると8時間遅れます。FlinkサーバーとStarRocksサーバーは同じタイムゾーンであるAsia/Shanghai UTC/GMT+08:00に位置しています。Flinkのバージョンは1.12です。ドライバー: flink-connector-jdbc_2.11です。この問題を解決する方法を教えていただけますか？

**解決策:**

Flinkのシンクテーブルで時間パラメーター'server-time-zone' = 'Asia/Shanghai'を設定してみてください。また、jdbc URLに&serverTimezone=Asia/Shanghaiを追加することもできます。以下に例を示します：

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

## flinkインポートでは、StarRocksクラスターにデプロイされたkafkaクラスターのみをインポートできます

**問題の説明:**

```SQL
failed to query wartermark offset, err: Local: Bad message format
```

**解決策:**

Kafkaの通信にはホスト名が必要です。ユーザーはStarRocksクラスターノードでホスト名の解決を/etc/hostsに設定する必要があります。

## StarRocksはバッチで'create tableステートメント'をエクスポートできますか？

**解決策:**

StarRocks Toolsを使用してステートメントをエクスポートすることができます。

## BEが要求したメモリがオペレーティングシステムに解放されない

これは正常な現象です。データベースに割り当てられた大きなメモリブロックは、割り当て時に予約され、解放時に遅延されるため、メモリを再利用し、メモリの割り当てをより便利にするためです。ユーザーはメモリの使用状況を長時間監視して、メモリが解放されるかどうかを確認するためにテスト環境を検証することをお勧めします。

## ダウンロード後、Flinkコネクタが機能しない

**問題の説明:**

このパッケージはAliyunミラーアドレスを通じて取得する必要があります。

**解決策:**

`/etc/maven/settings.xml`のミラーパートがすべてAliyunミラーアドレスを通じて取得するように構成されていることを確認してください。

もし構成されている場合は、以下のように変更してください：

 <mirror>
    <id>aliyunmaven </id>
    <mirrorf>central</mirrorf>
    <name>aliyun public repo</name>
    <url>https: //maven.aliyun.com/repository/public</url>
</mirror>

## Flink-connector-StarRocksのパラメーターsink.buffer-flush.interval-msの意味

**問題の説明:**

```plain text
+----------------------+--------------------------------------------------------------+
|         Option       | Required |  Default   | Type   |       Description           |
+-------------------------------------------------------------------------------------+
|  sink.buffer-flush.  |  NO      |   300000   | String | the flushing time interval, |
|  interval-ms         |          |            |        | range: [1000ms, 3600000ms]  |
+----------------------+--------------------------------------------------------------+
```

このパラメーターをチェックポイント間隔が5分で15秒に設定した場合、この値はまだ有効ですか？

**解決策:**

3つの閾値のうち、最初に到達したものが最初に有効になります。これはチェックポイント間隔の値に影響を受けず、チェックポイント間隔は完全に一致する場合にのみ機能します。interval-msはat_least_onceで使用されます。

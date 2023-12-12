---
displayed_sidebar: "Japanese"
---

# Flinkコネクタ

## flink-connector-jdbc_2.11sinkはStarRocksで8時間遅れています

**問題の説明:**

localtimestap関数によって生成される時間はFlinkでは正常です。しかしStarRocksに沈んだ時に8時間遅れました。FlinkサーバーとStarRocksサーバーは同じタイムゾーン、つまりAsia/Shanghai UTC/GMT+08:00に位置しています。Flinkバージョンは1.12です。ドライバー: flink-connector-jdbc_2.11. この問題を解決する方法を教えてもらえますか？

**解決方法:**

Flinkのsinkテーブルで'time'パラメーターを 'server-time-zone' = 'Asia/Shanghai' に設定してみてください。jdbc URLにも &serverTimezone=Asia/Shanghai を追加してみてください。以下に例を示します:

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

## Flinkインポートでは、StarRocksクラスタに展開されたkafkaクラスタのみをインポートできます

**問題の説明:**

```SQL
failed to query wartermark offset, err: Local: Bad message format
```

**解決方法:**

Kafkaの通信にはホスト名が必要です。ユーザーはStarRocksクラスタノードでホスト名解決を/etc/hostsに設定する必要があります。

## StarRocksは'create table statements'をバッチでエクスポートできますか？

**解決方法:**

StarRocksツールを使用してステートメントをエクスポートすることができます。

## BEが要求したメモリがオペレーティングシステムに返却されない

これは正常な現象です。データベースにオペレーティングシステムから割り当てられた大きなメモリブロックは割り当て時に予約され、解放時には遅延されて再利用されるため、メモリ割り当てをより便利に行うためです。メモリが解放されるかどうかを確認するために、ユーザーはメモリ使用状況を長期間監視してテスト環境を検証することを推奨します。

## ダウンロード後にFlinkコネクタが機能しない

**問題の説明:**

このパッケージはAliyunミラーアドレスを介して取得する必要があります。

**解決方法:**

/etc/maven/settings.xmlのミラー部分がすべてAliyunミラーアドレスを介して取得するように構成されていることを確認してください。

構成されている場合は、次のように変更してください:

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

このパラメータがチェックポイント間隔が5分であり、値を15秒に設定した場合、この値は有効ですか？

**解決方法:**

3つの閾値のうち、最初に到達したものが最初に有効になります。チェックポイント間隔の値は、単一の値に対してのみ機能するため、これに影響を与えません。interval-msはat_least_onceに使用されます。
---
displayed_sidebar: "Japanese"
---

# Flinkコネクタ

## flink-connector-jdbc_2.11のシンクがStarRocksで8時間遅れています

**問題の説明:**

Localtimestap関数によって生成された時間はFlinkでは正常です。しかし、StarRocksにシンクされると8時間遅れます。FlinkサーバーとStarRocksサーバーは同じタイムゾーン、つまりAsia/Shanghai UTC/GMT+08:00に位置しています。Flinkのバージョンは1.12で、ドライバーはflink-connector-jdbc_2.11です。この問題を解決する方法を教えていただけますか？

**解決策:**

Flinkのシンクテーブルで'time'というパラメータを'Asia/Shanghai'に設定してみてください。また、jdbcのURLに&serverTimezone=Asia/Shanghaiを追加してください。以下に例を示します：

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

## FlinkのインポートではStarRocksクラスタに展開されたkafkaクラスタのみインポートできます

**問題の説明:**

```SQL
failed to query wartermark offset, err: Local: Bad message format
```

**解決策:**

Kafka通信にはホスト名が必要です。ユーザーはStarRocksクラスタノードでホスト名の解決を/etc/hostsに設定する必要があります。

## StarRocksは'create tableステートメント'をバッチでエクスポートできますか？

**解決策:**

StarRocks Toolsを使用してステートメントをエクスポートできます。

## BEによって要求されたメモリが操作システムに戻されない

これは正常な現象です。データベースに割り当てられた大きなメモリブロックは、メモリを再利用してメモリの割り当てをより便利にするために、割り当て時に予約され、解放時に保留されています。メモリが解放されそうかを長期間監視して、メモリが解放されるかどうかを検証することをお勧めします。

## ダウンロード後にFlinkコネクタが機能しない

**問題の説明:**

このパッケージはAliyunミラーアドレスを通じて取得する必要があります。

**解決策:**

/etc/maven/settings.xmlのミラーパートがすべてAliyunミラーアドレスを通じて取得するように構成されていることを確認してください。

もしそうであれば、次のように変更してください：

 <mirror>
    <id>aliyunmaven</id>
    <mirrorof>central</mirrorof>
    <name>aliyun public repo</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>

## Flink-connector-StarRocksのパラメータ'sink.buffer-flush.interval-ms'の意味

**問題の説明:**

```plain text
+----------------------+--------------------------------------------------------------+
|         Option       | Required |  Default   | Type   |       Description           |
+-------------------------------------------------------------------------------------+
|  sink.buffer-flush.  |  NO      |   300000   | String | the flushing time interval, |
|  interval-ms         |          |            |        | range: [1000ms, 3600000ms]  |
+----------------------+--------------------------------------------------------------+
```

このパラメータを15秒に設定し、チェックポイント間隔が5分と等しい場合、この値は引き続き効果がありますか？

**解決策:**

これらのうちどちらかのしきい値が最初に達成された場合、それが最初に効果を発揮します。これはcheckpointインターバル値に影響されず、それは単一の値にのみ適用されます。Interval-msはat_least_onceに使用されます。
---
displayed_sidebar: English
---

# Flink Connector

## flink-connector-jdbc_2.11sink が StarRocks で 8 時間遅れる問題

**問題の説明:**

Flink で生成される localtimestap 関数の時間は正常ですが、StarRocks にデータを送ると 8 時間遅れになります。Flink サーバーと StarRocks サーバーは同じタイムゾーンであるアジア/上海 UTC/GMT+08:00 に位置しています。Flink のバージョンは 1.12、ドライバは flink-connector-jdbc_2.11 です。この問題をどのように解決すればよいでしょうか？

**解決策:**

Flink シンクテーブルの設定で時間パラメータ 'server-time-zone' = 'Asia/Shanghai' を設定してみてください。また、jdbc URL に &serverTimezone=Asia/Shanghai を追加することもできます。以下に例を示します。

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

## flink インポートでは、StarRocks クラスターにデプロイされた Kafka クラスターのみがインポート可能

**問題の説明:**

```SQL
failed to query watermark offset, err: Local: Bad message format
```

**解決策:**

Kafka 通信にはホスト名が必要です。ユーザーは StarRocks クラスターノードでホスト名解決のための /etc/hosts を設定する必要があります。

## StarRocks は「create table ステートメント」をバッチでエクスポートできますか？

**解決策:**

StarRocks Tools を使用してステートメントをエクスポートできます。

## BE によって要求されたメモリがオペレーティングシステムに返却されない

これは正常な現象で、オペレーティングシステムからデータベースに割り当てられた大規模なメモリブロックは、割り当て時に予約され、解放時には遅延されることで、メモリの再利用を容易にし、メモリ割り当てをより効率的に行うためです。ユーザーは、長期間にわたるメモリ使用量の監視を通じてテスト環境を検証し、メモリが解放されるかどうかを確認することを推奨します。

## ダウンロード後に Flink コネクタが機能しない

**問題の説明:**

このパッケージは Aliyun ミラーアドレスを通じて取得する必要があります。

**解決策:**

`/etc/maven/settings.xml` のミラー部分が Aliyun ミラーアドレスを通じて取得されるようにすべて設定されていることを確認してください。

そうであれば、以下のように変更してください。

 <mirror>
    <id>aliyunmaven</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun public repo</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>

## Flink-connector-StarRocks のパラメータ sink.buffer-flush.interval-ms の意味

**問題の説明:**

```plain text
+----------------------+--------------------------------------------------------------+
|         Option       | Required |  Default   | Type   |       Description           |
+-------------------------------------------------------------------------------------+
|  sink.buffer-flush.  |  NO      |   300000   | String | the flushing time interval, |
|  interval-ms         |          |            |        | range: [1000ms, 3600000ms]  |
+----------------------+--------------------------------------------------------------+
```

このパラメータが 15 秒に設定され、チェックポイント間隔が 5 分に等しい場合、この値はまだ有効ですか？

**解決策:**

3 つのしきい値のうち、最初に到達したものが最初に効果を発揮します。これはチェックポイント間隔の値には影響されず、チェックポイント間隔は exactly-once にのみ機能します。interval-ms は at_least_once で使用されます。

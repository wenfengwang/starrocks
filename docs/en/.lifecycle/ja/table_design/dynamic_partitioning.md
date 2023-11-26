---
displayed_sidebar: "Japanese"
---

# ダイナミックパーティショニング

StarRocksはダイナミックパーティショニングをサポートしており、テーブル内の新しい入力データをパーティション分割し、期限が切れたパーティションを自動的に削除することができます。この機能により、メンテナンスコストを大幅に削減することができます。

## ダイナミックパーティショニングの有効化

テーブル`site_access`を例に取ります。ダイナミックパーティショニングを有効にするには、PROPERTIESパラメータを設定する必要があります。設定項目の詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

```SQL
CREATE TABLE site_access(
    event_day DATE,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT DEFAULT '0'
)
DUPLICATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY RANGE(event_day)(
    PARTITION p20200321 VALUES LESS THAN ("2020-03-22"),
    PARTITION p20200322 VALUES LESS THAN ("2020-03-23"),
    PARTITION p20200323 VALUES LESS THAN ("2020-03-24"),
    PARTITION p20200324 VALUES LESS THAN ("2020-03-25")
)
DISTRIBUTED BY HASH(event_day, site_id)
PROPERTIES(
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-3",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "32",
    "dynamic_partition.history_partition_num" = "0"
);
```

**`PROPERTIES`**:

| パラメーター                            | 必須     | 説明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|-----------------------------------------| -------- |------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| dynamic_partition.enable                | いいえ   | ダイナミックパーティショニングを有効にします。有効な値は`TRUE`と`FALSE`です。デフォルト値は`TRUE`です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| dynamic_partition.time_unit             | はい     | 動的に作成されるパーティションの時間の粒度です。必須パラメーターです。有効な値は`HOUR`、`DAY`、`WEEK`、`MONTH`、`YEAR`です。時間の粒度は、動的に作成されるパーティションの接尾辞の形式を決定します。<ul><li>値が`DAY`の場合、動的に作成されるパーティションの接尾辞の形式はyyyyMMddです。例えば、パーティション名の接尾辞は`20200321`です。</li><li>値が`WEEK`の場合、動的に作成されるパーティションの接尾辞の形式はyyyy_wwです。例えば、2020年の13週目は`2020_13`です。</li><li>値が`MONTH`の場合、動的に作成されるパーティションの接尾辞の形式はyyyyMMです。例えば、2020年3月は`202003`です。</li><li>値が`YEAR`の場合、動的に作成されるパーティションの接尾辞の形式はyyyyです。例えば、2020年です。</li></ul> |
| dynamic_partition.time_zone             | いいえ   | 動的パーティションのタイムゾーンです。デフォルトではシステムのタイムゾーンと同じです。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| dynamic_partition.start                 | いいえ   | ダイナミックパーティショニングの開始オフセットです。このパラメーターの値は負の整数である必要があります。このオフセットより前のパーティションは、パラメーター`dynamic_partition.time_unit`の値によって現在の日、週、または月に基づいて削除されます。デフォルト値は`Integer.MIN_VALUE`、つまり-2147483648で、過去のパーティションは削除されません。                                                                                                                                                                                                                                                                                                                                                                  |
| dynamic_partition.end                   | はい     | ダイナミックパーティショニングの終了オフセットです。このパラメーターの値は正の整数である必要があります。現在の日、週、または月から終了オフセットまでのパーティションが事前に作成されます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| dynamic_partition.prefix                | いいえ   | 動的パーティションの名前に追加される接頭辞です。デフォルト値は`p`です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| dynamic_partition.buckets               | いいえ   | 動的パーティションごとのバケツの数です。デフォルト値は、予約語BUCKETSによって決定されるバケツの数と同じです。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| dynamic_partition.history_partition_num | いいえ   | ダイナミックパーティショニングメカニズムによって作成される過去のパーティションの数で、デフォルト値は`0`です。値が0より大きい場合、過去のパーティションが事前に作成されます。v2.5.2以降、StarRocksはこのパラメーターをサポートしています。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| dynamic_partition.start_day_of_week     | いいえ   | `dynamic_partition.time_unit`が`WEEK`の場合、各週の最初の日を指定するために使用されるパラメーターです。有効な値は`1`から`7`です。`1`は月曜日を意味し、`7`は日曜日を意味します。デフォルト値は`1`で、毎週月曜日から始まります。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| dynamic_partition.start_day_of_month    | いいえ   | `dynamic_partition.time_unit`が`MONTH`の場合、各月の最初の日を指定するために使用されるパラメーターです。有効な値は`1`から`28`です。`1`は毎月1日を意味し、`28`は毎月28日を意味します。デフォルト値は`1`で、毎月1日から始まります。最初の日は29日、30日、または31日にすることはできません。                                                                                                                                                                                                                                                                                                                                                                                                                                |
| dynamic_partition.replication_num       | いいえ   | 動的に作成されるパーティションのタブレットのレプリカ数です。デフォルト値は、テーブル作成時に設定されたレプリカ数と同じです。  |

**FEの設定:**

`dynamic_partition_check_interval_seconds`: ダイナミックパーティショニングのスケジューリング間隔です。デフォルト値は600秒で、10分ごとにパーティションの状況をチェックし、`PROPERTIES`で指定されたダイナミックパーティショニングの条件を満たしているかどうかを確認します。満たしていない場合、パーティションは自動的に作成および削除されます。

## パーティションの表示

テーブルにダイナミックパーティションを有効にした後、入力データは連続的に自動的にパーティション分割されます。次のステートメントを使用して、現在のパーティションを表示することができます。例えば、現在の日付が2020-03-25の場合、2020-03-22から2020-03-28の時間範囲内のパーティションのみ表示されます。

```SQL
SHOW PARTITIONS FROM site_access;

[types: [DATE]; keys: [2020-03-22]; ‥types: [DATE]; keys: [2020-03-23]; )
[types: [DATE]; keys: [2020-03-23]; ‥types: [DATE]; keys: [2020-03-24]; )
[types: [DATE]; keys: [2020-03-24]; ‥types: [DATE]; keys: [2020-03-25]; )
[types: [DATE]; keys: [2020-03-25]; ‥types: [DATE]; keys: [2020-03-26]; )
[types: [DATE]; keys: [2020-03-26]; ‥types: [DATE]; keys: [2020-03-27]; )
[types: [DATE]; keys: [2020-03-27]; ‥types: [DATE]; keys: [2020-03-28]; )
[types: [DATE]; keys: [2020-03-28]; ‥types: [DATE]; keys: [2020-03-29]; )
```

## ダイナミックパーティショニングのプロパティの変更

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)ステートメントを使用して、ダイナミックパーティショニングのプロパティを変更することができます。以下のステートメントを例に取ります。

```SQL
ALTER TABLE site_access 
SET("dynamic_partition.enable"="false");
```

> 注意:
>
> - テーブルのダイナミックパーティショニングのプロパティを確認するには、[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)ステートメントを実行します。
> - ALTER TABLEステートメントを使用して、テーブルの他のプロパティを変更することもできます。

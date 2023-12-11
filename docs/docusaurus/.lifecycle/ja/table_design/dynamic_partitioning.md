---
displayed_sidebar: "Japanese"
---

# 動的パーティション

StarRocksは動的パーティションをサポートしており、テーブル内の新規入力データをパーティション分割して期限切れのパーティションを自動的に削除するなど、パーティションの時間ライフ（TTL）を自動的に管理できます。この機能により、保守コストを大幅に削減できます。

## 動的パーティションの有効化

`site_access`テーブルを例として挙げます。動的パーティションを有効にするには、PROPERTIESパラメータを構成する必要があります。構成項目については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

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

| パラメータ名                               | 必須 | 説明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|-----------------------------------------| -------- |-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| dynamic_partition.enable                | いいえ       | 動的パーティションを有効にします。有効な値は`TRUE`と`FALSE`です。デフォルト値は`TRUE`です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| dynamic_partition.time_unit             | はい      | 動的に作成されたパーティションの時間の粒度です。必須パラメータです。有効な値は`HOUR`、`DAY`、`WEEK`、`MONTH`、`YEAR`です。時間の粒度は、動的に作成されたパーティションのサフィックス形式を決定します。<ul><li>値が`DAY`の場合、動的に作成されたパーティションのサフィックス形式はyyyyMMddです。例として、パーティション名のサフィックスは`20200321`です。</li><li>値が`WEEK`の場合、動的に作成されたパーティションのサフィックス形式はyyyy_wwです。例として、2020年の13週目は`2020_13`です。</li><li>値が`MONTH`の場合、動的に作成されたパーティションのサフィックス形式はyyyyMMです。例として、`202003`です。</li><li>値が`YEAR`の場合、動的に作成されたパーティションのサフィックス形式はyyyyです。例として、`2020`です。</li></ul> |
| dynamic_partition.time_zone             | いいえ       | ダイナミックパーティション用のタイムゾーンで、デフォルトでシステムのタイムゾーンと同じです。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| dynamic_partition.start                 | いいえ       | 動的パーティションの開始オフセットです。このパラメータの値は負の整数でなければなりません。このオフセットより前のパーティションは、パラメータ`dynamic_partition.time_unit`の値に応じて現在の日、週、または月に基づいて削除されます。デフォルト値は`Integer.MIN_VALUE`で、つまり-2147483648で、これは過去のパーティションが削除されないことを意味します。                                                                                                                                                                                                                                                                                                                                                                  |
| dynamic_partition.end                   | はい      | 動的パーティションの終了オフセットです。このパラメータの値は正の整数でなければなりません。現在の日、週、または月から終了オフセットまでのパーティションが事前に作成されます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| dynamic_partition.prefix                | いいえ       | 動的パーティションの名前に追加されるプレフィックスです。デフォルト値は`p`です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| dynamic_partition.buckets               | いいえ       | 動的パーティションごとのバケツの数です。デフォルト値は、StarRocksによって予約語BUCKETSによって決定されたバケツの数と同じです。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| dynamic_partition.history_partition_num | いいえ       | 動的パーティションメカニズムによって作成された過去のパーティションの数で、デフォルト値は`0`です。値が0より大きい場合、過去のパーティションが事前に作成されます。v2.5.2から、StarRocksはこのパラメータをサポートしています。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| dynamic_partition.start_day_of_week     | いいえ       | `dynamic_partition.time_unit`が`WEEK`の場合、このパラメータを使用して各週の最初の日を指定します。有効な値: `1` から `7`。`1` は月曜日を意味し、`7` は日曜日を意味します。デフォルト値は`1`で、つまり毎週月曜日から始まります。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| dynamic_partition.start_day_of_month    | いいえ       | `dynamic_partition.time_unit`が`MONTH`の場合、このパラメータを使用して各月の最初の日を指定します。有効な値: `1` から `28`。`1` は毎月1日を意味し、`28` は毎月28日を意味します。デフォルト値は`1`で、つまり毎月1日から始まります。初日は29日、30日、または31日にはできません。                                                                                                                                                                                                                                                                                                                                                                                                                                |
| dynamic_partition.replication_num       | いいえ       | 動的に作成されたパーティションのタブレットのレプリカ数です。デフォルト値は、テーブル作成時に構成されたレプリカの数と同じです。  |

**FE構成:**

`dynamic_partition_check_interval_seconds`：動的パーティションスケジューリングの間隔です。デフォルト値は600秒で、つまり、パーティションの状況をプロパティで指定された動的パーティション条件を満たすかどうかを10分ごとにチェックします。満たさない場合、パーティションは自動的に作成および削除されます。

## パーティションの表示

テーブルの動的パーティションを有効にした後、入力データは連続して自動的にパーティション分割されます。次のステートメントを使用して、現在のパーティションを表示できます。たとえば、現在の日付が2020-03-25の場合、2020-03-22から2020-03-28までの時間範囲内のパーティションのみを表示できます。

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

## 動的パーティションのプロパティの変更

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)ステートメントを使用して、動的パーティションのプロパティを変更できます。たとえば、次のステートメントを参照してください。

```SQL
ALTER TABLE site_access 
SET("dynamic_partition.enable"="false");
```

> 注：
>
> - テーブルの動的パーティションのプロパティを確認するには、[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)ステートメントを実行します。
> - ALTER TABLEステートメントを使用して、テーブルのその他のプロパティを変更することもできます。
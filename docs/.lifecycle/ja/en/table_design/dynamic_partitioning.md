---
displayed_sidebar: English
---

# 動的パーティショニング

StarRocksは動的パーティショニングをサポートしており、テーブル内の新しい入力データのパーティショニングや期限切れのパーティションの削除など、パーティションの存続時間(TTL)を自動的に管理することができます。この機能により、メンテナンスコストが大幅に削減されます。

## 動的パーティショニングを有効にする

例として `site_access` テーブルを取り上げます。動的パーティショニングを有効にするには、PROPERTIES パラメーターを設定する必要があります。設定項目の詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

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

| パラメーター                               | 必須 | 説明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|-----------------------------------------| -------- |-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| dynamic_partition.enable                | いいえ       | 動的パーティショニングを有効にします。有効な値は `TRUE` と `FALSE`です。デフォルト値は `TRUE`です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| dynamic_partition.time_unit             | はい      | 動的に作成されるパーティションの時間粒度。これは必須パラメーターです。有効な値は `HOUR`、`DAY`、`WEEK`、`MONTH`、`YEAR`です。時間粒度によって、動的に作成されるパーティションのサフィックス形式が決まります。<ul><li>値が `DAY`の場合、サフィックス形式は yyyyMMdd です。例えば、パーティション名のサフィックスは `20200321`です。</li><li>値が `WEEK`の場合、サフィックス形式は yyyy_ww です。例えば、 `2020_13` は2020年の第13週です。</li><li>値が `MONTH`の場合、サフィックス形式は yyyyMM です。例えば、 `202003`です。</li><li>値が `YEAR`の場合、サフィックス形式は yyyy です。例えば、 `2020`です。</li></ul> |
| dynamic_partition.time_zone             | いいえ       | 動的パーティションのタイムゾーン。デフォルトではシステムのタイムゾーンと同じです。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| dynamic_partition.start                 | いいえ       | 動的パーティショニングの開始オフセット。このパラメータの値は負の整数でなければなりません。このオフセットより前のパーティションは、`dynamic_partition.time_unit`の値に基づいて現在の日、週、または月を基準に削除されます。デフォルト値は `Integer.MIN_VALUE`、つまり-2147483648で、これは履歴パーティションが削除されないことを意味します。                                                                                                                                                                                                                                                                                                                                                                  |
| dynamic_partition.end                   | はい      | 動的パーティショニングの終了オフセット。このパラメータの値は正の整数でなければなりません。現在の日、週、または月から終了オフセットまでのパーティションが事前に作成されます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| dynamic_partition.prefix                | いいえ       | 動的パーティションの名前に追加される接頭辞。デフォルト値は `p`です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| dynamic_partition.buckets               | いいえ       | 動的パーティションごとのバケット数。デフォルト値は、予約語BUCKETSによって決定されるバケット数、またはStarRocksによって自動的に設定されるバケット数と同じです。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| dynamic_partition.history_partition_num | いいえ       | 動的パーティショニングメカニズムによって作成された履歴パーティションの数。デフォルト値は `0`です。値が0より大きい場合、履歴パーティションは事前に作成されます。v2.5.2から、StarRocksはこのパラメータをサポートしています。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| dynamic_partition.start_day_of_week     | いいえ       | `dynamic_partition.time_unit`が `WEEK`の場合、このパラメータは各週の最初の曜日を指定するために使用されます。有効な値は `1`から `7`です。`1`は月曜日、`7`は日曜日を意味します。デフォルト値は `1`で、これは毎週月曜日が始まりであることを意味します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| dynamic_partition.start_day_of_month    | いいえ       | `dynamic_partition.time_unit`が `MONTH`の場合、このパラメータは各月の最初の日を指定するために使用されます。有効な値は `1`から `28`です。`1`は毎月1日、`28`は毎月28日を意味します。デフォルト値は `1`で、これは毎月1日が始まりであることを意味します。初日は29日、30日、31日にはできません。                                                                                                                                                                                                                                                                                                                                                                                                                                |
| dynamic_partition.replication_num       | いいえ       | 動的に作成されたパーティション内のタブレットのレプリカ数。デフォルト値は、テーブル作成時に設定されたレプリカ数と同じです。  |

**FEの設定:**

`dynamic_partition_check_interval_seconds`: 動的パーティショニングをスケジュールする間隔。デフォルト値は600秒で、これはパーティションの状況が10分ごとにチェックされ、`PROPERTIES`で指定された動的パーティショニング条件を満たしているかどうかを確認します。満たしていない場合は、パーティションが自動的に作成および削除されます。

## パーティションの表示

テーブルに動的パーティショニングを有効にすると、入力データは継続的かつ自動的にパーティション分割されます。現在のパーティションを表示するには、次のステートメントを使用します。たとえば、現在の日付が2020-03-25の場合、2020-03-22から2020-03-28までの時間範囲のパーティションのみを表示できます。

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

## 動的パーティショニングのプロパティを変更する

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) ステートメントを使用して、動的パーティショニングのプロパティを変更できます。例えば、動的パーティショニングを無効にする場合は、以下のステートメントを使用します。

```SQL
ALTER TABLE site_access 
SET("dynamic_partition.enable"="false");
```

> 注記：
>
> - テーブルの動的パーティショニングのプロパティを確認するには、[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md) ステートメントを実行します。
> - また、ALTER TABLE ステートメントを使用して、テーブルの他のプロパティを変更することもできます。

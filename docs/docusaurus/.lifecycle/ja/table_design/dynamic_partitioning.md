---
displayed_sidebar: "Japanese"
---

# ダイナミックパーティショニング

StarRocksはダイナミックパーティショニングをサポートしており、テーブルにおける新しい入力データのパーティショニングや有効期限切れのパーティションの削除などを自動的に管理できます。この機能により、メンテナンスコストを大幅に削減できます。

## ダイナミックパーティショニングを有効にする

`site_access` テーブルを例に取ると、ダイナミックパーティショニングを有効にするには、PROPERTIESパラメータを構成する必要があります。構成項目の詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) を参照してください。

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

| パラメータ                               | 必須     | 説明                                                                                                                                                                                                                                                                                                                        |
|-----------------------------------------| -------- |-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| dynamic_partition.enable                | なし     | ダイナミックパーティショニングを有効にします。`TRUE` と `FALSE` の有効な値があります。デフォルト値は `TRUE` です。                                                                                                                                       |
| dynamic_partition.time_unit             | はい     | 動的に作成されたパーティションの時間単位です。必須パラメータです。有効な値は `HOUR`, `DAY`, `WEEK`, `MONTH`, `YEAR` です。                                                                                                                               |
| dynamic_partition.time_zone             | なし     | ダイナミックなパーティションのタイムゾーンであり、デフォルトでシステムのタイムゾーンと同じです。                                                                                                                                                  |
| dynamic_partition.start                 | なし     | 動的パーティショニングの開始オフセット。このパラメータの値は負の整数でなければなりません。このオフセットより前のパーティションは、`dynamic_partition.time_unit` パラメータの値によって現在の日、週、または月に基づいて削除されます。デフォルト値は `Integer.MIN_VALUE`、つまり -2147483648 です。                                                                                                                                               |
| dynamic_partition.end                   | はい     | ダイナミックパーティショニングの終了オフセット。このパラメータの値は正の整数でなければなりません。                                                                                                                                                    |
| dynamic_partition.prefix                | なし     | 動的パーティションの名前に追加されるプレフィックス。デフォルト値は `p` です。                                                                                                                                                                           |
| dynamic_partition.buckets               | なし     | 動的パーティションごとのバケツの数。デフォルト値は、StarRocksによって自動的に設定されたバケツの数と同じです。                                                                                                                                          |
| dynamic_partition.history_partition_num | なし     | ダイナミックパーティショニングメカニズムによって作成された過去のパーティションの数で、デフォルト値は `0` です。値が0より大きい場合、過去のパーティションが事前に作成されます。StarRocksはv2.5.2以降このパラメータをサポートしています。                                                                                                                                      |
| dynamic_partition.start_day_of_week     | なし     | `dynamic_partition.time_unit` が `WEEK` の場合、このパラメータは各週の最初の日を指定するために使用されます。有効な値: `1` から `7` まで。`1` は月曜日を意味し、`7` は日曜日を意味します。デフォルト値は `1` であり、各週が月曜日に始まることを意味します。                                                                                                                                          |
| dynamic_partition.start_day_of_month    | なし     | `dynamic_partition.time_unit` が `MONTH` の場合、このパラメータは各月の最初の日を指定するために使用されます。有効な値: `1` から `28` まで。`1` は毎月1日を意味し、`28` は毎月28日を意味します。デフォルト値は `1` であり、毎月1日に始まることを意味します。最初の日は29日、30日、または31日にすることはできません。                                                                                                                      |
| dynamic_partition.replication_num       | なし     | 動的に作成されたパーティションのタブレットのレプリカ数。デフォルト値は、テーブル作成時に構成されたレプリカ数と同じです。                                                                                                                       |

**FEの構成:**

`dynamic_partition_check_interval_seconds`: ダイナミックパーティショニングのスケジューリング間隔です。デフォルト値は600秒です。これは、`PROPERTIES` で指定されたダイナミックパーティショニング条件を満たすかどうかを10分ごとに確認して、パーティションが自動的に作成および削除されるかどうかを確認します。

## パーティションを表示する

テーブルのダイナミックパーティションを有効にした後、入力データは連続して自動的にパーティショニングされます。以下のステートメントを使用して、現在のパーティションを表示できます。例えば、現在の日付が2020-03-25の場合、2020-03-22から2020-03-28までの期間内のパーティションのみを表示できます。

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

## ダイナミックパーティショニングのプロパティを変更する

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) ステートメントを使用して、ダイナミックパーティショニングのプロパティを変更し、ダイナミックパーティショニングを無効にするなどができます。

```SQL
ALTER TABLE site_access 
SET("dynamic_partition.enable"="false");
```

> 注意:
>
> - テーブルのダイナミックパーティショニングのプロパティを確認するには、[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md) ステートメントを実行します。
> - 他のテーブルのプロパティを変更するには、ALTER TABLE ステートメントを使用できます。
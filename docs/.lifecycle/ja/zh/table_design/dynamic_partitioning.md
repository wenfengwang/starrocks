---
displayed_sidebar: Chinese
---

# 動的パーティション

動的パーティション機能が有効になると、新しいデータに必要に応じて動的にパーティションを作成でき、StarRocks は期限切れのパーティションを自動的に削除し、データの時宜性を保証します。

## 動的パーティションをサポートするテーブルの作成

以下の例では、動的パーティションをサポートするテーブルを作成しています。テーブル名は `site_access` で、動的パーティションは `PROPERTIES` で設定されます。パーティションの範囲は現在時刻の前後3日間で、合計6日間です。

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
    "dynamic_partition.history_partition_num" = "0"
);
```

 **動的パーティションに関連する `PROPERTIES`:**

| パラメータ                      | 必須 | 説明                                                                                                                                                                                                                                                                                                                    |
| ----------------------- |-----|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| dynamic_partition.enable | いいえ   | 動的パーティション機能を有効にするかどうかを設定します。値は `true`（デフォルト）または `false` です。                                                                                                                                                                                                                                                                                    |
| dynamic_partition.time_unit | はい   | 動的パーティションの時間単位。値は `HOUR`、`DAY`、`WEEK`、`MONTH`、または `YEAR` です。時間単位は動的に作成されるパーティション名のサフィックス形式を決定します。<br />`HOUR` を選択した場合、Datetime型のみをサポートし、パーティション名のサフィックス形式は yyyyMMddHH になります。例：`2020032101`。<br />`DAY` を選択した場合、パーティション名のサフィックス形式は yyyyMMdd になります。例：`20200321`。<br />`WEEK` を選択した場合、パーティション名のサフィックス形式は yyyy_ww になります。例：`2020_13` は2020年の第13週を表します。<br />`MONTH` を選択した場合、パーティション名のサフィックス形式は yyyyMM になります。例：`202003`。<br />`YEAR` を選択した場合、パーティション名のサフィックス形式は yyyy になります。例：`2020`。|
| dynamic_partition.time_zone |  いいえ   | 動的パーティションのタイムゾーン。デフォルトではシステムのタイムゾーンと同じです。                                                                                                                                                                                                                                                                                            |
| dynamic_partition.start | いいえ   | 保持する動的パーティションの開始オフセット。値は負の整数です。`dynamic_partition.time_unit` の設定に応じて、その日（週/月）を基準に、このオフセットより前のパーティションは削除されます。例えば `-3` に設定し、`dynamic_partition.time_unit` が `DAY` の場合、3日前のパーティションが削除されます。<br />設定しない場合、デフォルトは `Integer.MIN_VALUE`、つまり `-2147483648` で、これは過去のパーティションを削除しないことを意味します。                                                                                            |
| dynamic_partition.end   | はい   | 事前に作成するパーティションの数。値は正の整数です。`dynamic_partition.time_unit` の設定に応じて、その日（週/月）を基準に、対応する範囲のパーティションを事前に作成します。                                                                                                                                                                                                                                    |
| dynamic_partition.prefix | いいえ   | 動的パーティションのプレフィックス名。デフォルトは `p` です。                                                                                                                                                                                                                                                                                                    |
| dynamic_partition.buckets | いいえ   | 動的パーティションのバケット数。デフォルトでは、BUCKETS 予約語で指定されたバケット数、または StarRocks が自動的に設定したバケット数と同じです。                                                                                                                                                                                                                                                          |
| dynamic_partition.history_partition_num | いいえ   | 作成する履歴パーティションの数。デフォルトは `0` です。値が0より大きい場合、履歴パーティションが事前に作成されます。StarRocks バージョン2.5.2以降、このパラメータを設定できます。                                                                                                                                                                                                                                                                             |
| dynamic_partition.start_day_of_week     | いいえ   | `dynamic_partition.time_unit` が `WEEK` の場合、このパラメータは週の最初の日を指定します。有効な値は `1` から `7` です。`1` は月曜日を、`7` は日曜日を意味します。デフォルトは `1` で、週は月曜日から始まります。  |
| dynamic_partition.start_day_of_month     | いいえ   | `dynamic_partition.time_unit` が `MONTH` の場合、このパラメータは月の最初の日を指定します。有効な値は `1` から `28` です。`1` は月の最初の日を、`28` は月の28日を意味します。デフォルトは `1` で、月は最初の日から始まります。月の最初の日が29日、30日、または31日であることはサポートされていません。  |
| dynamic_partition.replication_num     | いいえ   | 動的に作成されるパーティション内の各タブレットのレプリカ数。デフォルトはテーブル作成時に設定されたレプリカ数と同じです。      |

**動的パーティションに関連する FE 設定項目:**

`dynamic_partition_check_interval_seconds`：FE の設定項目で、動的パーティションのチェック間隔です。デフォルトは 600 秒、つまり10分ごとに一度、`PROPERTIES` に設定された動的パーティションの属性を満たしているかどうかをチェックし、満たしていない場合は自動的にパーティションを作成および削除します。

## テーブルの現在のパーティション状況を確認する

動的パーティション機能が有効になると、パーティションは自動的に増減します。以下のステートメントを実行して、テーブルの現在のパーティション状況を確認できます：

```SQL
SHOW PARTITIONS FROM site_access;
```

現在時刻が 2020-03-25 であると仮定すると、動的パーティションのスケジュール時に、2020-03-22 より前のパーティションが削除され、同時に今後3日間のパーティションが作成されます。したがって、上記のステートメントの結果は、`Range` 列に現在のパーティション情報が以下のように表示されます：

```SQL
[types: [DATE]; keys: [2020-03-22]; ‥types: [DATE]; keys: [2020-03-23]; )
[types: [DATE]; keys: [2020-03-23]; ‥types: [DATE]; keys: [2020-03-24]; )
[types: [DATE]; keys: [2020-03-24]; ‥types: [DATE]; keys: [2020-03-25]; )
[types: [DATE]; keys: [2020-03-25]; ‥types: [DATE]; keys: [2020-03-26]; )
[types: [DATE]; keys: [2020-03-26]; ‥types: [DATE]; keys: [2020-03-27]; )
[types: [DATE]; keys: [2020-03-27]; ‥types: [DATE]; keys: [2020-03-28]; )
[types: [DATE]; keys: [2020-03-28]; ‥types: [DATE]; keys: [2020-03-29]; )
```

## テーブルの動的パーティション属性を変更する

ALTER TABLE を実行して、動的パーティションの属性を変更します。例えば、動的パーティション機能を一時停止または有効にします。

```SQL
ALTER TABLE site_access SET("dynamic_partition.enable"="false");
ALTER TABLE site_access SET("dynamic_partition.enable"="true");
```

> 説明：
>
> - SHOW CREATE TABLE コマンドを実行して、テーブルの動的パーティション属性を確認できます。
> - ALTER TABLE は `PROPERTIES` 内の他の設定項目を変更するのにも使用できます。

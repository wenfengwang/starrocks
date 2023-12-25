---
displayed_sidebar: English
---

# マテリアライズドビューの表示

## 説明

すべてまたは特定の非同期マテリアライズドビューを表示します。

v3.0 以降、このステートメントの名前は SHOW MATERIALIZED VIEW から SHOW MATERIALIZED VIEWS に変更されました。

:::tip

この操作には権限は必要ありません。

:::

## 構文

```SQL
SHOW MATERIALIZED VIEWS
[FROM db_name]
[
WHERE NAME { = "mv_name" | LIKE "mv_name_matcher"}
]
```

角括弧 [] 内のパラメータはオプショナルです。

## パラメーター

| **パラメーター**   | **必須** | **説明**                                              |
| --------------- | ------------ | ------------------------------------------------------------ |
| db_name         | いいえ           | マテリアライズドビューが存在するデータベースの名前。このパラメーターを指定しない場合は、デフォルトで現在のデータベースが使用されます。 |
| mv_name         | いいえ           | 表示するマテリアライズドビューの名前。                   |
| mv_name_matcher | いいえ           | マテリアライズドビューをフィルタリングするために使用されるマッチャー。               |

## 戻り値

| **戻り値**                 | **説明**                                              |
| -------------------------- | ------------------------------------------------------------ |
| id                         | マテリアライズドビューのID。                             |
| database_name              | マテリアライズドビューが存在するデータベースの名前。 |
| name                       | マテリアライズドビューの名前。                           |
| refresh_type               | マテリアライズドビューの更新タイプ。ROLLUP、MANUAL、ASYNC、INCREMENTAL を含む。 |
| is_active                  | マテリアライズドビューの状態がアクティブかどうか。有効な値: `true` と `false`。 |
| partition_type             | マテリアライズドビューのパーティションタイプ。RANGE および UNPARTITIONED を含む。                |
| task_id                    | マテリアライズドビュー更新タスクのID。                  |
| task_name                  | マテリアライズドビュー更新タスクの名前。                |
| last_refresh_start_time    | マテリアライズドビューの最後の更新の開始時刻。 |
| last_refresh_finished_time | マテリアライズドビューの最後の更新の終了時刻。   |
| last_refresh_duration      | 最後の更新にかかった時間。単位: 秒。           |
| last_refresh_state         | 最後の更新の状態。PENDING、RUNNING、FAILED、SUCCESS を含む。 |
| last_refresh_force_refresh | 最後の更新が FORCE リフレッシュであったかどうか。                 |
| last_refresh_start_partition | マテリアライズドビューの最後の更新の開始パーティション。 |
| last_refresh_end_partition | マテリアライズドビューの最後の更新の終了パーティション。 |
| last_refresh_base_refresh_partitions | 最後の更新でリフレッシュされたベーステーブルのパーティション。 |
| last_refresh_mv_refresh_partitions | 最後の更新でリフレッシュされたマテリアライズドビューのパーティション。 |
| last_refresh_error_code    | マテリアライズドビューの最後の失敗した更新のエラーコード（マテリアライズドビューの状態がアクティブでない場合）。 |
| last_refresh_error_message | 最後の更新が失敗した理由（マテリアライズドビューの状態がアクティブでない場合）。 |
| rows                       | マテリアライズドビューに含まれるデータ行の数。            |
| text                       | マテリアライズドビューを作成するために使用されたステートメント。          |

## 例

以下の例は、このビジネスシナリオに基づいています。

```Plain
-- Create Table: customer
CREATE TABLE customer ( C_CUSTKEY     INTEGER NOT NULL,
                        C_NAME        VARCHAR(25) NOT NULL,
                        C_ADDRESS     VARCHAR(40) NOT NULL,
                        C_NATIONKEY   INTEGER NOT NULL,
                        C_PHONE       CHAR(15) NOT NULL,
                        C_ACCTBAL     double   NOT NULL,
                        C_MKTSEGMENT  CHAR(10) NOT NULL,
                        C_COMMENT     VARCHAR(117) NOT NULL,
                        PAD char(1) NOT NULL)
    ENGINE=OLAP
DUPLICATE KEY(`c_custkey`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`c_custkey`)
PROPERTIES (
"replication_num" = "1",
"storage_format" = "DEFAULT"
);

-- Create MV: customer_mv
CREATE MATERIALIZED VIEW customer_mv
DISTRIBUTED BY HASH(c_custkey)
REFRESH MANUAL
PROPERTIES (
    "replication_num" = "1"
)
AS SELECT
              c_custkey, c_phone, c_acctbal, count(1) as c_count, sum(c_acctbal) as c_sum
   FROM
              customer
   GROUP BY c_custkey, c_phone, c_acctbal;

-- Refresh the MV
REFRESH MATERIALIZED VIEW customer_mv;
```

例 1: 特定のマテリアライズドビューを表示する。

```Plain
mysql> SHOW MATERIALIZED VIEWS WHERE NAME='customer_mv'\G;
*************************** 1. row ***************************
                        id: 10142
                      name: customer_mv
             database_name: test
              refresh_type: MANUAL
                 is_active: true
   last_refresh_start_time: 2023-02-17 10:27:33
last_refresh_finished_time: 2023-02-17 10:27:33
     last_refresh_duration: 0
        last_refresh_state: SUCCESS
             inactive_code: 0
           inactive_reason:
                      text: CREATE MATERIALIZED VIEW `customer_mv`
COMMENT "MATERIALIZED_VIEW"
DISTRIBUTED BY HASH(`c_custkey`)
REFRESH MANUAL
PROPERTIES (
"replication_num" = "1",
"storage_medium" = "HDD"
)
AS SELECT `customer`.`c_custkey`, `customer`.`c_phone`, `customer`.`c_acctbal`, count(1) AS `c_count`, sum(`customer`.`c_acctbal`) AS `c_sum`
FROM `test`.`customer`
GROUP BY `customer`.`c_custkey`, `customer`.`c_phone`, `customer`.`c_acctbal`;
                      rows: 0
1 row in set (0.11 sec)
```

例 2: 名前にマッチするマテリアライズドビューを表示する。

```Plain
mysql> SHOW MATERIALIZED VIEWS WHERE NAME LIKE 'customer_mv'\G;
*************************** 1. row ***************************
                        id: 10142
                      name: customer_mv
             database_name: test
              refresh_type: MANUAL
                 is_active: true
   last_refresh_start_time: 2023-02-17 10:27:33
last_refresh_finished_time: 2023-02-17 10:27:33
     last_refresh_duration: 0
        last_refresh_state: SUCCESS
             inactive_code: 0
           inactive_reason:
                      text: CREATE MATERIALIZED VIEW `customer_mv`
COMMENT "MATERIALIZED_VIEW"
DISTRIBUTED BY HASH(`c_custkey`)
REFRESH MANUAL
PROPERTIES (
"replication_num" = "1",
"storage_medium" = "HDD"
)
AS SELECT `customer`.`c_custkey`, `customer`.`c_phone`, `customer`.`c_acctbal`, count(1) AS `c_count`, sum(`customer`.`c_acctbal`) AS `c_sum`
FROM `test`.`customer`
GROUP BY `customer`.`c_custkey`, `customer`.`c_phone`, `customer`.`c_acctbal`;
                      rows: 0
1 row in set (0.12 sec)
```

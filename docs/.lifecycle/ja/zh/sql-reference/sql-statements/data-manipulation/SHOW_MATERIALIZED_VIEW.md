---
displayed_sidebar: Chinese
---

# マテリアライズドビューの表示

## 機能

すべてまたは指定された非同期マテリアライズドビューの情報を表示します。このステートメントはバージョン3.0から `SHOW MATERIALIZED VIEWS` に名前が変更されました。以前のバージョンでは `SHOW MATERIALIZED VIEW` でした。

> **注意**
>
> このコマンドは現在、非同期マテリアライズドビューにのみ有効です。同期マテリアライズドビューについては、[SHOW ALTER MATERIALIZED VIEW](../data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md) コマンドを使用して、現在のデータベースの同期マテリアライズドビューの構築状態を確認できます。
> この操作には権限は必要ありません。

## 構文

```SQL
SHOW MATERIALIZED VIEWS
[FROM db_name]
[
WHERE NAME { = "mv_name" | LIKE "mv_name_matcher"}
]
```

## パラメータ

| **パラメータ**        | **必須** | **説明**                                                     |
| --------------- | -------- | ------------------------------------------------------------ |
| db_name         | いいえ       | マテリアライズドビューが属するデータベース名。指定しない場合は、現在のデータベースが使用されます。 |
| mv_name         | いいえ       | 精密なマッチングに使用されるマテリアライズドビューの名前。                                 |
| mv_name_matcher | いいえ       | 曖昧なマッチングに使用されるマテリアライズドビューの名前のマッチャー。                         |

## 戻り値

最近の REFRESH タスクの状態を返します。

| **戻り値**                   | **説明**                                                    |
| -------------------------- | --------------------------------------------------------- |
| id                         | マテリアライズドビューのID。                                               |
| database_name              | マテリアライズドビューが属するデータベース名。                                     |
| name                       | マテリアライズドビューの名前。                                               |
| refresh_type               | マテリアライズドビューの更新方法。ROLLUP、MANUAL、ASYNC、INCREMENTAL を含む。   |
| is_active                  | マテリアライズドビューの状態が active かどうか。有効な値は `true` と `false`。          |
| partition_type             | マテリアライズドビューのパーティションタイプ。RANGE と UNPARTITIONED を含む。|
| task_id                    | マテリアライズドビューのリフレッシュタスクID。                                       |
| task_name                  | マテリアライズドビューのリフレッシュタスク名。                                       |
| last_refresh_start_time    | マテリアライズドビューの最後のリフレッシュ開始時間。                                    |
| last_refresh_finished_time | マテリアライズドビューの最後のリフレッシュ終了時間。                                    |
| last_refresh_duration      | マテリアライズドビューの最後のリフレッシュの所要時間（秒単位）。                               |
| last_refresh_state         | マテリアライズドビューの最後のリフレッシュの状態。PENDING、RUNNING、FAILED、SUCCESS を含む。 |
| last_refresh_force_refresh | マテリアライズドビューの最後のリフレッシュが強制（FORCE）リフレッシュだったかどうか。                     ｜
| last_refresh_start_partition | 最後のリフレッシュが開始されたマテリアライズドビューのパーティション。                               ｜
| last_refresh_end_partition | 最後のリフレッシュが終了したマテリアライズドビューのパーティション。                                 ｜
| last_refresh_base_refresh_partitions | 最後のリフレッシュで更新されたベーステーブルのパーティション。                           ｜
| last_refresh_mv_refresh_partitions | 最後のリフレッシュで更新されたマテリアライズドビューのパーティション。                         ｜
| last_refresh_error_code    | マテリアライズドビューの最後のリフレッシュで失敗した場合の ErrorCode（マテリアライズドビューの状態が active でない場合）。 |
| last_refresh_error_message | マテリアライズドビューの最後のリフレッシュで失敗した場合の ErrorMessage（マテリアライズドビューの状態が active でない場合）。 |
| rows                       | マテリアライズドビューに含まれるデータ行数。                                           |
| text                       | マテリアライズドビューを作成するためのクエリ文。                                        |

## 例

以下の例は、現在のビジネスシナリオに基づいています：

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
DISTRIBUTED BY HASH(`c_custkey`) BUCKETS 10
PROPERTIES (
"replication_num" = "1",
"storage_format" = "DEFAULT"
);

-- Create MV: customer_mv
CREATE MATERIALIZED VIEW customer_mv
DISTRIBUTED BY HASH(c_custkey) buckets 10
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

例1：特定のマテリアライズドビューを正確にマッチングして表示

```Plain
mysql> show materialized views where name='customer_mv'\G;
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
DISTRIBUTED BY HASH(`c_custkey`) BUCKETS 10
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

例2：マテリアライズドビューを曖昧にマッチングして表示

```Plain
mysql> show materialized views where name like 'customer_mv'\G;
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
DISTRIBUTED BY HASH(`c_custkey`) BUCKETS 10
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

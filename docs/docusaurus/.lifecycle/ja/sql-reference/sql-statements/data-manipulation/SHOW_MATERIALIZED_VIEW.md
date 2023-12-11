```yaml
---
displayed_sidebar: "Japanese"
---

# マテリアライズド・ビュー表示

## 説明

すべてまたは特定の非同期マテリアライズド・ビューを表示します。

v3.0から、このステートメントの名前が「SHOW MATERIALIZED VIEW」から「SHOW MATERIALIZED VIEWS」に変更されました。

## 構文

```SQL
SHOW MATERIALIZED VIEWS
[FROM db_name]
[
WHERE NAME { = "mv_name" | LIKE "mv_name_matcher"}
]
```

角かっこ[ ]内のパラメータはオプションです。

## パラメータ

| **パラメータ**     | **必須** | **説明**                                              |
| --------------- | ------------ | ------------------------------------------------------------ |
| db_name         | いいえ           | マテリアライズド・ビューが存在するデータベースの名前です。このパラメータが指定されていない場合、デフォルトで現在のデータベースが使用されます。 |
| mv_name         | いいえ           | 表示するマテリアライズド・ビューの名前です。                   |
| mv_name_matcher | いいえ           | マテリアライズド・ビューをフィルタリングするために使用されるマッチャーです。               |

## 戻り値

| **戻り値**                 | **説明**                                              |
| -------------------------- | ------------------------------------------------------------ |
| id                         | マテリアライズド・ビューのID。                             |
| database_name              | マテリアライズド・ビューが存在するデータベースの名前。 |
| name                       | マテリアライズド・ビューの名前。                           |
| refresh_type               | マテリアライズド・ビューのリフレッシュタイプ（ROLLUP、MANUAL、ASYNC、INCREMENTALを含む）。 |
| is_active                  | マテリアライズド・ビューの状態がアクティブであるかどうか。有効な値：`true`および`false`。 |
| partition_type             | マテリアライズド・ビューのパーティションタイプ（RANGEおよびUNPARTITIONEDを含む）。                |
| task_id                    | マテリアライズド・ビューのリフレッシュタスクのID。                  |
| task_name                  | マテリアライズド・ビューのリフレッシュタスクの名前。                |
| last_refresh_start_time    | マテリアライズド・ビューの最終リフレッシュの開始時刻。 |
| last_refresh_finished_time | マテリアライズド・ビューの最終リフレッシュの終了時刻。   |
| last_refresh_duration      | 最終リフレッシュにかかった時間。単位：秒。           |
| last_refresh_state         | 最終リフレッシュのステータス（PENDING、RUNNING、FAILED、SUCCESSを含む）。 |
| last_refresh_force_refresh | 最終リフレッシュがFORCEリフレッシュであるかどうか。                 |
| last_refresh_start_partition | マテリアライズド・ビューの最終リフレッシュの開始パーティション。 |
| last_refresh_end_partition | マテリアライズド・ビューの最終リフレッシュの終了パーティション。 |
| last_refresh_base_refresh_partitions | 最終リフレッシュでリフレッシュされたベーステーブルのパーティション。 |
| last_refresh_mv_refresh_partitions | 最終リフレッシュでリフレッシュされたマテリアライズド・ビューのパーティション。 |
| last_refresh_error_code    | マテリアライズド・ビューの最後のリフレッシュが失敗した場合のエラーコード（マテリアライズド・ビューの状態がアクティブでない場合）。 |
| last_refresh_error_message | マテリアライズド・ビューの最後のリフレッシュが失敗した理由（マテリアライズド・ビューの状態がアクティブでない場合）。 |
| rows                       | マテリアライズド・ビュー内のデータ行数。            |
| text                       | マテリアライズド・ビューを作成するために使用されるステートメント。          |

## 例

次の例は、次のビジネスシナリオに基づいています。

```Plain
-- テーブルを作成：customer
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

-- マテリアライズド・ビューを作成：customer_mv
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

-- マテリアライズド・ビューをリフレッシュ
REFRESH MATERIALIZED VIEW customer_mv;
```

例1: 特定のマテリアライズド・ビューを表示する。

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

例2: 名前に一致するマテリアライズド・ビューを表示する。

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
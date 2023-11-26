---
displayed_sidebar: "Japanese"
---

# SHOW CREATE TABLE

指定されたテーブルを作成するために使用されたCREATE TABLEステートメントを返します。

> **注意**
>
> v3.0より前のバージョンでは、SHOW CREATE TABLEステートメントを実行するには、テーブルに対して`SELECT_PRIV`権限が必要でした。v3.0以降、SHOW CREATE TABLEステートメントを実行するには、テーブルに対して`SELECT`権限が必要です。

v3.0以降、SHOW CREATE TABLEステートメントを使用して、外部カタログで管理され、Apache Hive™、Apache Iceberg、Apache Hudi、またはDelta Lakeに格納されているテーブルのCREATE TABLEステートメントを表示できます。

v2.5.7以降、StarRocksでは、テーブルを作成するかパーティションを追加する際に、バケットの数（BUCKETS）を自動的に設定できます。バケットの数を手動で設定する必要はもはやありません。詳細については、[バケットの数を決定する](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

- テーブルを作成する際にバケットの数を指定した場合、SHOW CREATE TABLEの出力にはバケットの数が表示されます。
- テーブルを作成する際にバケットの数を指定しなかった場合、SHOW CREATE TABLEの出力にはバケットの数が表示されません。各パーティションのバケットの数を表示するには、[SHOW PARTITIONS](SHOW_PARTITIONS.md)を実行できます。

v2.5.7より前のバージョンでは、テーブルを作成する際にバケットの数を指定する必要があります。そのため、SHOW CREATE TABLEではデフォルトでバケットの数が表示されます。

## 構文

```SQL
SHOW CREATE TABLE [db_name.]table_name
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                                     |
| -------------- | --------- | ------------------------------------------------------------ |
| db_name        | いいえ    | データベース名。このパラメータが指定されていない場合、デフォルトで現在のデータベースの指定されたテーブルのCREATE TABLEステートメントが返されます。 |
| table_name     | はい      | テーブル名。                                                 |

## 出力

```Plain
+-----------+----------------+
| Table     | Create Table   |                                               
+-----------+----------------+
```

このステートメントによって返されるパラメータについては、以下の表を参照してください。

| **パラメータ** | **説明**                   |
| -------------- | -------------------------- |
| Table          | テーブル名。               |
| Create Table   | テーブルのCREATE TABLEステートメント。 |

## 例

### バケットの数が指定されていない場合

DISTRIBUTED BYでバケットの数が指定されていない`example_table`という名前のテーブルを作成します。

```SQL
CREATE TABLE example_table
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 CHAR(10) REPLACE,
    v2 INT SUM
)
ENGINE = olap
AGGREGATE KEY(k1, k2)
COMMENT "my first starrocks table"
DISTRIBUTED BY HASH(k1);
```

SHOW CREATE TABLEを実行して`example_table`のCREATE TABLEステートメントを表示します。DISTRIBUTED BYにはバケットの数が表示されません。テーブルを作成する際にPROPERTIESを指定しなかった場合、SHOW CREATE TABLEの出力にはデフォルトのプロパティが表示されます。

```Plain
SHOW CREATE TABLE example_table\G
*************************** 1. row ***************************
       Table: example_table
Create Table: CREATE TABLE `example_table` (
  `k1` tinyint(4) NULL COMMENT "",
  `k2` decimal64(10, 2) NULL DEFAULT "10.5" COMMENT "",
  `v1` char(10) REPLACE NULL COMMENT "",
  `v2` int(11) SUM NULL COMMENT ""
) ENGINE=OLAP 
AGGREGATE KEY(`k1`, `k2`)
COMMENT "my first starrocks table"
DISTRIBUTED BY HASH(`k1`)
PROPERTIES (
"replication_num" = "3",
"in_memory" = "false",
"enable_persistent_index" = "false",
"replicated_storage" = "true",
"compression" = "LZ4"
);
```

### バケットの数が指定されている場合

DISTRIBUTED BYでバケットの数を10に設定した`example_table1`という名前のテーブルを作成します。

```SQL
CREATE TABLE example_table1
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 CHAR(10) REPLACE,
    v2 INT SUM
)
ENGINE = olap
AGGREGATE KEY(k1, k2)
COMMENT "my first starrocks table"
DISTRIBUTED BY HASH(k1) BUCKETS 10;
```

SHOW CREATE TABLEを実行して`example_table1`のCREATE TABLEステートメントを表示します。DISTRIBUTED BYにはバケットの数（`BUCKETS 10`）が表示されます。テーブルを作成する際にPROPERTIESを指定しなかった場合、SHOW CREATE TABLEの出力にはデフォルトのプロパティが表示されます。

```plain
SHOW CREATE TABLE example_table1\G
*************************** 1. row ***************************
       Table: example_table1
Create Table: CREATE TABLE `example_table1` (
  `k1` tinyint(4) NULL COMMENT "",
  `k2` decimal64(10, 2) NULL DEFAULT "10.5" COMMENT "",
  `v1` char(10) REPLACE NULL COMMENT "",
  `v2` int(11) SUM NULL COMMENT ""
) ENGINE=OLAP 
AGGREGATE KEY(`k1`, `k2`)
COMMENT "my first starrocks table"
DISTRIBUTED BY HASH(`k1`) BUCKETS 10 
PROPERTIES (
"replication_num" = "3",
"in_memory" = "false",
"enable_persistent_index" = "false",
"replicated_storage" = "true",
"compression" = "LZ4"
);
```

## 参照

- [CREATE TABLE](../data-definition/CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [ALTER TABLE](../data-definition/ALTER_TABLE.md)
- [DROP TABLE](../data-definition/DROP_TABLE.md)

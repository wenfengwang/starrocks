---
displayed_sidebar: "Japanese"
---

# テーブルの作成文を表示

指定されたテーブルを作成した際に使用されたCREATE TABLE文を返します。

> **注記**
>
> v3.0より前のバージョンでは、SHOW CREATE TABLE文を使用するにはテーブルに対する`SELECT_PRIV`権限が必要です。v3.0以降、SHOW CREATE TABLE文を使用するにはテーブルに対する`SELECT`権限が必要です。

v3.0以降、SHOW CREATE TABLE文を使用して、外部カタログで管理され、Apache Hive™、Apache Iceberg、Apache Hudi、またはDelta Lakeに格納されているテーブルのCREATE TABLE文を表示できます。

v2.5.7以降、StarRocksはテーブルの作成やパーティションの追加時に自動的にバケット数（BUCKETS）を設定できます。バケット数の手動設定は不要です。詳細については、[バケット数の決定](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

- テーブル作成時にバケット数を指定した場合、SHOW CREATE TABLEの出力にはバケット数が表示されます。
- テーブル作成時にバケット数を指定しなかった場合、SHOW CREATE TABLEの出力にはバケット数が表示されません。各パーティションのバケット数を表示するには、[SHOW PARTITIONS](SHOW_PARTITIONS.md)を実行できます。

v2.5.7より前のバージョンでは、テーブル作成時にバケット数を設定する必要があります。そのため、SHOW CREATE TABLEはデフォルトでバケット数を表示します。

## 構文

```SQL
SHOW CREATE TABLE [db_name.]table_name
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                                     |
| -------------- | ---------- | ------------------------------------------------------------ |
| db_name         | いいえ      | データベース名。このパラメータを指定しない場合、現在のデータベース内の指定されたテーブルのCREATE TABLE文がデフォルトで返されます。 |
| table_name     | はい        | テーブル名。                                                  |

## 出力

```Plain
+-----------+----------------+
| テーブル  | Create Table   |                                               
+-----------+----------------+
```

次の表は、この文によって返されるパラメータを記述しています。

| **パラメータ** | **説明**                   |
| -------------- | -------------------------- |
| テーブル       | テーブル名。               |
| Create Table  | テーブルのCREATE TABLE文。 |

## 例

### バケット数が指定されていない

DISTRIBUTED BYでバケット数が指定されていない`example_table`というテーブルを作成します。

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

SHOW CREATE TABLEを実行して、`example_table` のCREATE TABLE文を表示します。 DISTRIBUTED BYにバケット数は表示されません。テーブル作成時にPROPERTIESを指定しなかった場合、SHOW CREATE TABLEの出力にはデフォルトのプロパティが表示されます。

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

### バケット数が指定されている

DISTRIBUTED BYでバケット数を10として`example_table1`というテーブルを作成します。

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

SHOW CREATE TABLEを実行して、`example_table1` のCREATE TABLE文を表示します。 DISTRIBUTED BYにバケット数（`BUCKETS 10`）が表示されます。テーブル作成時にPROPERTIESを指定しなかった場合、SHOW CREATE TABLEの出力にはデフォルトのプロパティが表示されます。

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
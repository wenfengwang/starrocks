---
displayed_sidebar: "Japanese"
---

# SHOW CREATE TABLE（テーブル作成文の表示）

指定されたテーブルを作成するために使用されたCREATE TABLE文を返します。

> **注記**
>
> v3.0より前のバージョンでは、SHOW CREATE TABLE文を実行するには、テーブルに対する`SELECT_PRIV`権限が必要です。v3.0以降、SHOW CREATE TABLE文を実行するには、テーブルに対する`SELECT`権限が必要です。 

v3.0以降、SHOW CREATE TABLE文を使用して、外部カタログによって管理され、Apache Hive™、Apache Iceberg、Apache Hudi、またはDelta Lakeに保存されているテーブルのCREATE TABLE文を表示できます。

v2.5.7以降、StarRocksは、テーブルの作成またはパーティションの追加時にバケツ（BUCKETS）の数を自動的に設定できます。バケツの数を手動で設定する必要はもはやありません。詳細については、[バケツの数を決定する](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

- テーブル作成時にバケツの数を指定した場合、SHOW CREATE TABLEの出力にはバケツの数が表示されます。
- テーブル作成時にバケツの数を指定しなかった場合、SHOW CREATE TABLEの出力にはバケツの数が表示されません。各パーティションのバケツの数を表示するには、[SHOW PARTITIONS](SHOW_PARTITIONS.md)を実行できます。

v2.5.7より前のバージョンでは、テーブルを作成する際にバケツの数を設定する必要があります。そのため、SHOW CREATE TABLEではデフォルトでバケツの数が表示されます。

## 構文

```SQL
SHOW CREATE TABLE [db_name.]table_name
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                                     |
| ------------- | ------------ | ------------------------------------------------------------ |
| db_name       | No           | データベース名。このパラメータを指定しない場合、デフォルトで現在のデータベース内の指定されたテーブルのCREATE TABLE文が返されます。 |
| table_name    | Yes          | テーブル名。                                                 |

## 出力

```Plain
+-----------+----------------+
| テーブル   | Create Table   |                                               
+-----------+----------------+
```

次の表は、この文によって返されるパラメータを説明しています。

| **パラメータ** | **説明**                          |
| ------------- | ---------------------------------------- |
| テーブル         | テーブル名。                          |
| Create Table  | テーブルのCREATE TABLE文。 |

## 例

### バケツ数が指定されていない場合

DISTRIBUTED BYでバケツ番号が指定されていない`example_table`という名前のテーブルを作成します。

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

SHOW CREATE TABLEを実行して`example_table`のCREATE TABLE文を表示します。DISTRIBUTED BYにバケツの数が表示されません。テーブルを作成する際にPROPERTIESを指定していない場合は、SHOW CREATE TABLEの出力にはデフォルトのプロパティが表示されます。

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

### バケツ数が指定されている場合

DISTRIBUTED BYでバケツ番号を10に設定した`example_table1`という名前のテーブルを作成します。

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

SHOW CREATE TABLEを実行して`example_table`のCREATE TABLE文を表示します。DISTRIBUTED BYにバケツの数（`BUCKETS 10`）が表示されます。テーブルを作成する際にPROPERTIESを指定していない場合は、SHOW CREATE TABLEの出力にはデフォルトのプロパティが表示されます。

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
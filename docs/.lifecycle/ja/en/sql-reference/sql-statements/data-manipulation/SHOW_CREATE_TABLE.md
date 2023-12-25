---
displayed_sidebar: English
---

# SHOW CREATE TABLE

特定のテーブルを作成するために使用されたCREATE TABLEステートメントを返します。

> **注記**
>
> v3.0より前のバージョンでは、SHOW CREATE TABLEステートメントを使用するには、テーブルに対する`SELECT_PRIV`権限が必要でした。v3.0以降、SHOW CREATE TABLEステートメントを使用するには、テーブルに対する`SELECT`権限が必要です。

v3.0以降、SHOW CREATE TABLEステートメントを使用して、外部カタログによって管理され、Apache Hive™、Apache Iceberg、Apache Hudi、またはDelta Lakeに格納されているテーブルのCREATE TABLEステートメントを表示できます。

v2.5.7以降、StarRocksはテーブル作成時やパーティション追加時にバケット数（BUCKETS）を自動的に設定するようになりました。バケット数を手動で設定する必要はありません。詳細情報については、[バケット数の決定](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

- テーブル作成時にバケット数を指定した場合、SHOW CREATE TABLEの出力にバケット数が表示されます。
- テーブル作成時にバケット数を指定しなかった場合、SHOW CREATE TABLEの出力にバケット数は表示されません。各パーティションのバケット数を表示するには、[SHOW PARTITIONS](SHOW_PARTITIONS.md)を実行してください。

v2.5.7より前のバージョンでは、テーブル作成時にバケット数を設定する必要がありました。そのため、SHOW CREATE TABLEはデフォルトでバケット数を表示します。

## 構文

```SQL
SHOW CREATE TABLE [db_name.]table_name
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| db_name       | いいえ           | データベース名。このパラメータが指定されていない場合、現在のデータベース内の指定されたテーブルのCREATE TABLEステートメントがデフォルトで返されます。 |
| table_name    | はい          | テーブル名。                                              |

## 出力

```Plain
+-----------+----------------+
| Table     | Create Table   |                                               
+-----------+----------------+
```

次の表は、このステートメントによって返されるパラメータを説明しています。

| **パラメータ** | **説明**                          |
| ------------- | ---------------------------------------- |
| Table         | テーブル名。                          |
| Create Table  | テーブルのCREATE TABLEステートメント。 |

## 例

### バケット番号が指定されていない

DISTRIBUTED BYでバケット番号を指定せずに`example_table`という名前のテーブルを作成します。

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

SHOW CREATE TABLEを実行して`example_table`のCREATE TABLEステートメントを表示します。DISTRIBUTED BYにバケット番号は表示されません。テーブルを作成する際にPROPERTIESを指定しなかった場合、SHOW CREATE TABLEの出力にデフォルトのプロパティが表示されます。

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

### バケット番号が指定されている

DISTRIBUTED BYでバケット番号を10に設定して`example_table1`という名前のテーブルを作成します。

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

SHOW CREATE TABLEを実行して`example_table1`のCREATE TABLEステートメントを表示します。DISTRIBUTED BYにバケット番号（`BUCKETS 10`）が表示されます。テーブルを作成する際にPROPERTIESを指定しなかった場合、SHOW CREATE TABLEの出力にデフォルトのプロパティが表示されます。

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

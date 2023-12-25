---
displayed_sidebar: Chinese
---

# SHOW CREATE TABLE

## 機能

指定されたテーブルのCREATE TABLE文を表示します。

バージョン3.0以降では、このステートメントを使用してExternal Catalogにあるテーブルを表示できます。これにはApache Hive™、Apache Iceberg、Apache Hudi、Delta Lakeのテーブルが含まれます。

バージョン2.5.7以降、StarRocksはテーブル作成とパーティション追加時にバケット数（BUCKETS）を自動的に設定する機能をサポートしています。手動でバケット数を設定する必要はありません。詳細については、[バケット数の決定](../../../table_design/Data_distribution.md#バケット数の決定)を参照してください。

- テーブル作成時にバケット数を指定した場合、SHOW CREATE TABLEはバケット数を表示します。

- テーブル作成時にバケット数を指定しなかった場合、SHOW CREATE TABLEはバケット数を表示しませんが、[SHOW PARTITIONS](SHOW_PARTITIONS.md)でパーティションのバケット数を確認できます。

バージョン2.5.7以前では、テーブル作成時にバケット数を設定する必要があったため、SHOW CREATE TABLEはバケット数を表示します。

> **注意**
>
> - バージョン3.0以前では、SELECT_PRIV権限を持つユーザーのみが表示できます。
> - バージョン3.0以降では、SELECT権限を持つユーザーのみが表示できます。

## 構文

```SQL
SHOW CREATE TABLE [db_name.]table_name
```

## パラメータ説明

| **パラメータ** | **必須** | **説明**                                       |
| -------------- | -------- | ---------------------------------------------- |
| db_name        | いいえ   | データベース名。指定された場合、指定されたデータベース内のテーブルのCREATE TABLE文を表示します。 |
| table_name     | はい     | テーブル名。                                     |

## 戻り値の説明

```Plain
+-----------+----------------+
| Table     | Create Table   |       
+-----------+----------------+
```

戻り値に含まれるパラメータの説明は以下の通りです：

| **パラメータ** | **説明**   |
| -------------- | ---------- |
| Table          | テーブル名。 |
| Create Table   | CREATE TABLE文。 |

## 例

### テーブル作成時にバケット数を指定しない

バケット数を指定せずに `example_table` というテーブルを作成します。

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

`example_table` のCREATE TABLE文を表示します。バケット数は表示されません。PROPERTIESが指定されていない場合、SHOW CREATE TABLE文はデフォルトのPROPERTIESを表示します。

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

### テーブル作成時にバケット数を指定

バケット数を10に指定して `example_table1` というテーブルを作成します。

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

`example_table1` のCREATE TABLE文を表示します。バケット数が表示されます。PROPERTIESが指定されていない場合、SHOW CREATE TABLE文はデフォルトのPROPERTIESを表示します。

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

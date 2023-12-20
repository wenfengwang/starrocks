---
displayed_sidebar: English
---

# 刷新外部表

## 描述

此操作用于更新 StarRocks 中缓存的 Hive 和 Hudi 元数据。适用于以下场景之一：

- **外部表**：当通过 Hive 外部表或 Hudi 外部表查询 Apache Hive™ 或 Apache Hudi 中的数据时，您可以执行此操作来更新在 StarRocks 中缓存的 Hive 表或 Hudi 表的元数据。
- **外部目录**：当使用[Hive catalog](../../../data_source/catalog/hive_catalog.md)或[Hudi catalog](../../../data_source/catalog/hudi_catalog.md)查询Hive或Hudi中的数据时，您可以执行此操作来更新StarRocks缓存中的Hive表或Hudi表的元数据。

## 基本概念

- **Hive外部表**：在StarRocks中创建并存储，用于查询Hive数据。
- **Hudi 外部表**：在 StarRocks 中创建并存储，用于查询 Hudi 数据。
- **Hive 表**：在 Hive 中创建并存储。
- **Hudi表**：在Hudi中创建并存储。

## 语法和参数

以下基于不同案例描述语法和参数：

- 外部表

  ```SQL
  REFRESH EXTERNAL TABLE table_name 
  [PARTITION ('partition_name', ...)]
  ```

  |参数|必填|说明|
|---|---|---|
  |table_name|Yes|Hive 外部表或 Hudi 外部表的名称。|
  |partition_name|No|Hive 表或 Hudi 表的分区名称。指定此参数会更新 StarRocks 中缓存的 Hive 表和 Hudi 表分区的元数据。|

- 外部目录

  ```SQL
  REFRESH EXTERNAL TABLE [external_catalog.][db_name.]table_name
  [PARTITION ('partition_name', ...)]
  ```

  |参数|必填|说明|
|---|---|---|
  |external_catalog|No|Hive 目录或 Hudi 目录的名称。|
  |db_name|No|Hive表或Hudi表所在数据库的名称。|
  |table_name|Yes|Hive 表或 Hudi 表的名称。|
  |partition_name|No|Hive 表或 Hudi 表的分区名称。指定此参数会更新 StarRocks 中缓存的 Hive 表和 Hudi 表分区的元数据。|

## 使用须知

只有拥有 ALTER_PRIV 权限的用户才能执行此操作来刷新 StarRocks 缓存中的 Hive 表和 Hudi 表元数据。

## 示例

以下是不同情形下的使用示例：

### 外部表

示例 1：通过指定外部表 hive1 来刷新 StarRocks 中相应 Hive 表的缓存元数据。

```SQL
REFRESH EXTERNAL TABLE hive1;
```

示例 2：通过指定外部表 hudi1 及其对应 Hudi 表的分区来刷新 StarRocks 中相应 Hudi 表分区的缓存元数据。

```SQL
REFRESH EXTERNAL TABLE hudi1
PARTITION ('date=2022-12-20', 'date=2022-12-21');
```

### 外部目录

示例 1：刷新 StarRocks 中 hive_table 的缓存元数据。

```SQL
REFRESH EXTERNAL TABLE hive_catalog.hive_db.hive_table;
```

或

```SQL
USE hive_catalog.hive_db;
REFRESH EXTERNAL TABLE hive_table;
```

示例 2：刷新 StarRocks 中 hudi_table 分区的缓存元数据。

```SQL
REFRESH EXTERNAL TABLE hudi_catalog.hudi_db.hudi_table
PARTITION ('date=2022-12-20', 'date=2022-12-21');
```

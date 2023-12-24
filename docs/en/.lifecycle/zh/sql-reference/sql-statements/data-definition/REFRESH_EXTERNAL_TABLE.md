---
displayed_sidebar: English
---

# 刷新外部表

## 描述

更新 StarRocks 中缓存的 Hive 和 Hudi 元数据。此语句用于以下情况之一：

- **外部表**：在使用 Hive 外部表或 Hudi 外部表查询 Apache Hive 或 Apache Hudi 中的数据时，您可以执行此语句来更新 StarRocks 中缓存的 Hive™ 表或 Hudi 表的元数据。
- **外部目录**：在使用[Hive目录](../../../data_source/catalog/hive_catalog.md)或[Hudi目录](../../../data_source/catalog/hudi_catalog.md)查询 Hive 或 Hudi 中的数据时，您可以执行此语句来更新 StarRocks 中缓存的 Hive 表或 Hudi 表的元数据。

## 基本概念

- **Hive 外部表**：在 StarRocks 中创建并存储，可用于查询 Hive 数据。
- **Hudi 外部表**：在 StarRocks 中创建并存储，可用于查询 Hudi 数据。
- **Hive 表**：在 Hive 中创建并存储。
- **Hudi 表**：在 Hudi 中创建并存储。

## 语法和参数

下面描述了基于不同情况的语法和参数：

- 外部表

    ```SQL
    REFRESH EXTERNAL TABLE table_name 
    [PARTITION ('partition_name', ...)]
    ```

    | **参数**  | **必填** | **描述**                                              |
    | -------------- | ------------ | ------------------------------------------------------------ |
    | table_name     | 是          | Hive 外部表或 Hudi 外部表的名称。    |
    | partition_name | 否           | Hive 表或 Hudi 表的分区名称。指定此参数将更新 StarRocks 中缓存的 Hive 表和 Hudi 表的分区元数据。 |

- 外部目录

    ```SQL
    REFRESH EXTERNAL TABLE [external_catalog.][db_name.]table_name
    [PARTITION ('partition_name', ...)]
    ```

    | **参数**    | **必填** | **描述**                                              |
    | ---------------- | ------------ | ------------------------------------------------------------ |
    | external_catalog | 否           | Hive 目录或 Hudi 目录的名称。                  |
    | db_name          | 否           | Hive 表或 Hudi 表所在的数据库的名称。 |
    | table_name       | 是          | Hive 表或 Hudi 表的名称。                    |
    | partition_name   | 否           | Hive 表或 Hudi 表的分区名称。指定此参数将更新 StarRocks 中缓存的 Hive 表和 Hudi 表的分区元数据。 |

## 使用说明

只有拥有 `ALTER_PRIV` 权限的用户才能执行此语句来更新 StarRocks 中缓存的 Hive 表和 Hudi 表的元数据。

## 例子

不同情况下的使用示例如下：

### 外部表

示例 1：通过指定外部表名称 `hive1` 来更新 StarRocks 中对应 Hive 表的缓存元数据。

```SQL
REFRESH EXTERNAL TABLE hive1;
```

示例 2：通过指定外部表名称 `hudi1` 和对应 Hudi 表的分区来更新 StarRocks 中对应 Hudi 表分区的缓存元数据。

```SQL
REFRESH EXTERNAL TABLE hudi1
PARTITION ('date=2022-12-20', 'date=2022-12-21');
```

### 外部目录

示例 1：更新 StarRocks 中 `hive_table` 的缓存元数据。

```SQL
REFRESH EXTERNAL TABLE hive_catalog.hive_db.hive_table;
```

或

```SQL
USE hive_catalog.hive_db;
REFRESH EXTERNAL TABLE hive_table;
```

示例 2：更新 StarRocks 中 `hudi_table` 的分区缓存元数据。

```SQL
REFRESH EXTERNAL TABLE hudi_catalog.hudi_db.hudi_table
PARTITION ('date=2022-12-20', 'date=2022-12-21');
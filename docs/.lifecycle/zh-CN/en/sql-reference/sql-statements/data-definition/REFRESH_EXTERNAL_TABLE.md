---
displayed_sidebar: "中文"
---

# 刷新外部表

## 描述

更新StarRocks中缓存的Hive和Hudi元数据。该语句用于以下场景之一：

- **外部表**：当使用Hive外部表或Hudi外部表查询Apache Hive™或Apache Hudi中的数据时，可以执行此语句来更新StarRocks中缓存的Hive表或Hudi表的元数据。
- **外部目录**：当使用[Hive目录](../../../data_source/catalog/hive_catalog.md)或[Hudi目录](../../../data_source/catalog/hudi_catalog.md)查询Hive或Hudi中的数据时，可以执行此语句来更新StarRocks中缓存的Hive表或Hudi表的元数据。

## 基本概念

- **Hive外部表**：在StarRocks中创建并存储。您可以使用它来查询Hive数据。
- **Hudi外部表**：在StarRocks中创建并存储。您可以使用它来查询Hudi数据。
- **Hive表**：在Hive中创建并存储。
- **Hudi表**：在Hudi中创建并存储。

## 语法和参数

以下描述了基于不同情况的语法和参数：

- 外部表

    ```SQL
    REFRESH EXTERNAL TABLE table_name 
    [PARTITION ('partition_name', ...)]
    ```

    | **参数**          | **是否必须** | **描述**                                  |
    | ---------------- | ------------ | ---------------------------------------- |
    | table_name       | 是           | Hive外部表或Hudi外部表的名称。               |
    | partition_name   | 否           | Hive表或Hudi表的分区名称。指定此参数会更新StarRocks中缓存的Hive表和Hudi表的分区元数据。       |

- 外部目录

    ```SQL
    REFRESH EXTERNAL TABLE [external_catalog.][db_name.]table_name
    [PARTITION ('partition_name', ...)]
    ```

    | **参数**            | **是否必须** | **描述**                                  |
    | ------------------ | ------------ | ---------------------------------------- |
    | external_catalog   | 否           | Hive目录或Hudi目录的名称。                  |
    | db_name            | 否           | Hive表或Hudi表所在的数据库名称。                |
    | table_name         | 是           | Hive表或Hudi表的名称。                      |
    | partition_name     | 否           | Hive表或Hudi表的分区名称。指定此参数会更新StarRocks中缓存的Hive表和Hudi表的分区元数据。       |

## 使用注意事项

只有具有`ALTER_PRIV`特权的用户才能执行此语句，以更新StarRocks中缓存的Hive表和Hudi表的元数据。

## 示例

不同场景下的使用示例如下：

### 外部表

示例 1：通过指定外部表`hive1`来更新StarRocks中对应Hive表的缓存元数据。

```SQL
REFRESH EXTERNAL TABLE hive1;
```

示例 2：通过指定外部表`hudi1`和对应Hudi表的分区来更新StarRocks中对应Hudi表的分区元数据。

```SQL
REFRESH EXTERNAL TABLE hudi1
PARTITION ('date=2022-12-20', 'date=2022-12-21');
```

### 外部目录

示例 1：更新StarRocks中`hive_table`的缓存元数据。

```SQL
REFRESH EXTERNAL TABLE hive_catalog.hive_db.hive_table;
```

或

```SQL
USE hive_catalog.hive_db;
REFRESH EXTERNAL TABLE hive_table;
```

示例 2：更新StarRocks中`hudi_table`的分区元数据。

```SQL
REFRESH EXTERNAL TABLE hudi_catalog.hudi_db.hudi_table
PARTITION ('date=2022-12-20', 'date=2022-12-21');
```
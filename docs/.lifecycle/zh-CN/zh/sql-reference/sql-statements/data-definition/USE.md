---
displayed_sidebar: "中文"
---

# 使用

## 功能

指定会话使用的数据库。指定数据库之后，即可进行随后的建表或者查询等操作。

## 语法

```SQL
USE [<catalog_name>.]<db_name>
```

## 参数说明

| **参数**     | **必选** | **说明**                                                     |
| ------------ | -------- | ------------------------------------------------------------ |
| catalog_name | 否       | Catalog 名称。<ul><li>如果不指定该参数，则默认使用 `default_catalog` 下的数据库。</li><li>如果要使用 external catalog 下的数据库，则必须指定该参数，详见「示例二」。</li><li>如需从一个 catalog 下的数据库切换到另一 catalog 下的数据库，则必须指定该参数，详见「示例三」。</li></ul>有关 catalog 的更多信息，请参考[概述](../../../data_source/catalog/catalog_overview.md)。 |
| db_name      | 是       | 数据库名称。该数据库必须已存在。                             |

## 示例

示例一：使用 `default_catalog` 下的 `example_db` 作为会话当前数据库。

```SQL
USE default_catalog.example_db;
```

或

```SQL
USE example_db;
```

示例二：使用 `hive_catalog` 下的 `example_db` 作为会话当前数据库。

```SQL
USE hive_catalog.example_db;
```

示例三：把会话使用的数据库从 `hive_catalog.example_table1` 切换到 `iceberg_catalog.example_table2`。

```SQL
USE iceberg_catalog.example_table2;
```
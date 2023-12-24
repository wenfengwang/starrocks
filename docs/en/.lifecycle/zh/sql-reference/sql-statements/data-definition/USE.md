---
displayed_sidebar: English
---

# 使用

## 描述

指定您的会话的活动数据库。然后，您可以执行操作，比如创建表和执行查询。

## 语法

```SQL
USE [<catalog_name>.]<db_name>
```

## 参数

| **参数** | **必填** | **描述**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| catalog_name  | 否           | 目录名称。<ul><li>如果未指定此参数，则默认使用`default_catalog`中的数据库。</li><li>当您使用外部目录中的数据库时，必须指定此参数。有关详细信息，请参见示例 2。</li><li>在不同目录之间切换数据库时，必须指定此参数。有关详细信息，请参见示例 3。</li></ul>有关目录的更多信息，请参见[概述](../../../data_source/catalog/catalog_overview.md)。 |
| db_name       | 是          | 数据库名称。数据库必须存在。                  |

## 例子

示例 1：将 `example_db` 从 `default_catalog` 作为您会话的活动数据库。

```SQL
USE default_catalog.example_db;
```

或

```SQL
USE example_db;
```

示例 2：将 `example_db` 从 `hive_catalog` 作为您会话的活动数据库。

```SQL
USE hive_catalog.example_db;
```

示例 3：将您会话的活动数据库从 `hive_catalog.example_table1` 切换到 `iceberg_catalog.example_table2`。

```SQL
USE iceberg_catalog.example_table2;
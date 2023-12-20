---
displayed_sidebar: English
---

# 使用

## 描述

指定您会话中的活跃数据库。之后，您可以进行各种操作，例如创建表格和执行查询。

## 语法

```SQL
USE [<catalog_name>.]<db_name>
```

## 参数

|参数|必填|说明|
|---|---|---|
|catalog_name|No|目录名称。如果不指定该参数，则默认使用default_catalog 中的数据库。使用外部目录中的数据库时必须指定该参数。详细信息请参见示例2。在不同目录之间切换数据库时必须指定此参数。有关详细信息，请参阅示例 3。有关目录的详细信息，请参阅概述。|
|db_name|是|数据库名称。数据库必须存在。|

## 示例

示例 1：将 default_catalog 中的 example_db 设置为您会话的活跃数据库。

```SQL
USE default_catalog.example_db;
```

或

```SQL
USE example_db;
```

示例 2：将 hive_catalog 中的 example_db 设置为您会话的活跃数据库。

```SQL
USE hive_catalog.example_db;
```

示例 3：将会话的活跃数据库从 hive_catalog.example_table1 切换至 iceberg_catalog.example_table2。

```SQL
USE iceberg_catalog.example_table2;
```

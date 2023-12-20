---
displayed_sidebar: English
---

# 目录

## 描述

返回当前目录的名称。该目录既可以是 StarRocks 的内部目录，也可以是映射到外部数据源的外部目录。欲了解更多关于目录的信息，请参见[目录概览](../../../data_source/catalog/catalog_overview.md)。

如果没有选定目录，将返回 StarRocks 的内部默认目录 default_catalog。

## 语法

```Haskell
catalog()
```

## 参数

此函数不需要参数。

## 返回值

返回当前目录名称的字符串。

## 示例

示例 1：当前目录是 StarRocks 的内部默认目录 default_catalog。

```plaintext
select catalog();
+-----------------+
| CATALOG()       |
+-----------------+
| default_catalog |
+-----------------+
1 row in set (0.01 sec)
```

示例 2：当前目录是外部目录 hudi_catalog。

```sql
-- Switch to an external catalog.
set catalog hudi_catalog;

-- Return the name of the current catalog.
select catalog();
+--------------+
| CATALOG()    |
+--------------+
| hudi_catalog |
+--------------+
```

## 另请参阅

[SET CATALOG](../../sql-statements/data-definition/SET_CATALOG.md)：切换至指定目录。

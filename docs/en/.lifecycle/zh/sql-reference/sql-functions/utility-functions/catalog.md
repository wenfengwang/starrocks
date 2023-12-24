---
displayed_sidebar: English
---

# 目录

## 描述

返回当前目录的名称。目录可以是 StarRocks 内部目录，也可以是映射到外部数据源的外部目录。有关目录的详细信息，请参阅 [目录概述](../../../data_source/catalog/catalog_overview.md)。

如果未选择目录，则返回 StarRocks 内部目录 `default_catalog` 。

## 语法

```Haskell
catalog()
```

## 参数

此函数不需要参数。

## 返回值

以字符串形式返回当前目录的名称。

## 例子

例子 1：当前目录为 StarRocks 内部目录 `default_catalog`。

```plaintext
select catalog();
+-----------------+
| CATALOG()       |
+-----------------+
| default_catalog |
+-----------------+
1 行受影响 (0.01 秒)
```

例子 2：当前目录是外部目录 `hudi_catalog`。

```sql
-- 切换到外部目录。
set catalog hudi_catalog;

-- 返回当前目录的名称。
select catalog();
+--------------+
| CATALOG()    |
+--------------+
| hudi_catalog |
+--------------+
```

## 另请参阅

[SET CATALOG](../../sql-statements/data-definition/SET_CATALOG.md)：切换到目标目录。

---
displayed_sidebar: "Chinese"
---

# 目录

## 描述

返回当前目录的名称。目录可以是StarRocks内部目录，也可以是映射到外部数据源的外部目录。有关目录的更多信息，请参见[目录概述](../../../data_source/catalog/catalog_overview.md)。

如果没有选择目录，则返回StarRocks内部目录`default_catalog`。

## 语法

```Haskell
catalog()
```

## 参数

此函数不需要参数。

## 返回值

以字符串形式返回当前目录的名称。

## 示例

示例1：当前目录为StarRocks内部目录`default_catalog`。

```plaintext
select catalog();
+-----------------+
| CATALOG()       |
+-----------------+
| default_catalog |
+-----------------+
1行结果 (0.01秒)
```

示例2：当前目录为外部目录`hudi_catalog`。

```sql
-- 切换至外部目录。
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
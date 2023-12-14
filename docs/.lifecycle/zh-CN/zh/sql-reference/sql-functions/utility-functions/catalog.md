---
displayed_sidebar: "Chinese"
---

# 目录

## 功能

查询当前会话所在的目录。可以是内部目录或外部目录。有关目录的详细信息，请参见 [](../../../data_source/catalog/catalog_overview.md)。

如果未选定目录，默认显示 StarRocks 系统内的内部目录 `default_catalog`。

## 语法

```Haskell
catalog()
```

## 参数说明

该函数不需要传入参数。

## 返回值说明

返回当前会话所在的目录名称。

## 示例

示例一：当前目录为 StarRocks 系统内的内部目录。

```sql
select catalog();
+-----------------+
| CATALOG()       |
+-----------------+
| default_catalog |
+-----------------+
1 row in set (0.01 sec)
```

示例二：当前目录为外部目录 `hudi_catalog`。

```sql
-- 切换到目标外部目录。
set catalog hudi_catalog;

-- 返回当前目录名称。
select catalog();
+--------------+
| CATALOG()    |
+--------------+
| hudi_catalog |
+--------------+
```

## 相关 SQL

[SET CATALOG](../../sql-statements/data-definition/SET_CATALOG.md)：切换到指定目录。
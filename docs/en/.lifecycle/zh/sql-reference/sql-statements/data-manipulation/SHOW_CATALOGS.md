---
displayed_sidebar: English
---

# 显示目录

## 描述

查询当前 StarRocks 集群中的所有目录，包括内部目录和外部目录。

> **注意**
> SHOW CATALOGS 仅向对该外部目录具有 USAGE 权限的用户返回外部目录。如果用户或角色对任何外部目录都没有此权限，则此命令只返回 `default_catalog`。

## 语法

```SQL
SHOW CATALOGS
```

## 输出

```SQL
+----------+--------+----------+
| Catalog  | Type   | Comment  |
+----------+--------+----------+
```

下表描述了此语句返回的字段。

|**字段**|**描述**|
|---|---|
|Catalog|目录名称。|
|Type|目录类型。如果目录是 `default_catalog`，则返回 `Internal`。如果目录是外部目录，例如 `Hive`、`Hudi` 或 `Iceberg`，则返回相应的目录类型。|
|Comment|目录的注释。StarRocks 不支持为外部目录添加注释。因此，对于外部目录，该值为 `NULL`。如果目录是 `default_catalog`，默认情况下注释为 `一个内部目录包含此集群的自我管理表。` `default_catalog` 是 StarRocks 集群中唯一的内部目录。|

## 示例

查询当前集群中的所有目录。

```SQL
SHOW CATALOGS\G
*************************** 1. row ***************************
Catalog: default_catalog
   Type: Internal
Comment: 一个内部目录包含此集群的自我管理表。
*************************** 2. row ***************************
Catalog: hudi_catalog
   Type: Hudi
Comment: NULL
*************************** 3. row ***************************
Catalog: iceberg_catalog
   Type: Iceberg
Comment: NULL
```
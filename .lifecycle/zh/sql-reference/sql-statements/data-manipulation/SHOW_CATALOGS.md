---
displayed_sidebar: English
---

# 显示目录

## 描述

查询当前 StarRocks 集群中的所有目录，包括内部目录和外部目录。

> **注意**
> SHOW CATALOGS 命令会将拥有相应外部目录 **USAGE** 权限的用户的外部目录信息返回。如果用户或角色对任何外部目录都没有此权限，那么该命令只会返回 default_catalog。

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

以下表格描述了该语句返回的字段信息。

|字段|描述|
|---|---|
|目录|目录名称。|
|类型|目录类型。如果目录是default_catalog，则返回Internal。如果目录是外部目录，例如Hive、Hudi或Iceberg，则返回相应的目录类型。|
|评论|目录的评论。 StarRocks 不支持向外部目录添加评论。因此，对于外部目录，该值为 NULL。如果目录是default_catalog，则注释是内部目录包含此集群的自我管理表。默认情况下。 default_catalog 是 StarRocks 集群中唯一的内部目录。|

## 示例

查询当前集群中的所有目录。

```SQL
SHOW CATALOGS\G
*************************** 1. row ***************************
Catalog: default_catalog
   Type: Internal
Comment: An internal catalog contains this cluster's self-managed tables.
*************************** 2. row ***************************
Catalog: hudi_catalog
   Type: Hudi
Comment: NULL
*************************** 3. row ***************************
Catalog: iceberg_catalog
   Type: Iceberg
Comment: NULL
```

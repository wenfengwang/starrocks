---
displayed_sidebar: English
---

# 显示表格

## 描述

显示 StarRocks 数据库或外部数据源（例如 Hive、Iceberg、Hudi 或 Delta Lake）中的所有表。

> **注意**
> 要查看外部数据源中的表，您必须拥有对应该数据源的外部目录的 **USAGE** 权限。

## 语法

```sql
SHOW TABLES [FROM <catalog_name>.<db_name>]
```

## 参数

|参数|必填|说明|
|---|---|---|
|catalog_name|否|内部目录或外部目录的名称。<ul><li>如果不指定此参数或将其设置为 `default_catalog`，则返回 StarRocks 数据库中的表。</li><li>如果将此参数设置为外部目录的名称，则返回外部数据源数据库中的表。</li></ul> 您可以运行 [SHOW CATALOGS](SHOW_CATALOGS.md) 查看内部和外部目录。|
|db_name|否|数据库名称。如果未指定，默认使用当前数据库。|

## 示例

示例 1：连接到 StarRocks 集群后，查看 `default_catalog` 下的 `example_db` 数据库中的表。以下两个语句是等效的。

```plain
show tables from example_db;
+----------------------------+
| Tables_in_example_db       |
+----------------------------+
| depts                      |
| depts_par                  |
| emps                       |
| emps2                      |
+----------------------------+

show tables from default_catalog.example_db;
+----------------------------+
| Tables_in_example_db       |
+----------------------------+
| depts                      |
| depts_par                  |
| emps                       |
| emps2                      |
+----------------------------+
```

示例 2：连接到当前数据库 `example_db` 后查看该数据库中的表。

```plain
show tables;
+----------------------------+
| Tables_in_example_db       |
+----------------------------+
| depts                      |
| depts_par                  |
| emps                       |
| emps2                      |
+----------------------------+
```

示例 3：查看外部目录 `hudi_catalog` 下的 `hudi_db` 数据库中的表。

```plain
show tables from hudi_catalog.hudi_db;
+----------------------------+
| Tables_in_hudi_db          |
+----------------------------+
| hudi_sync_mor              |
| hudi_table1                |
+----------------------------+
```

或者，您可以运行 `SET CATALOG` 切换到外部目录 `hudi_catalog`，然后运行 `SHOW TABLES FROM hudi_db;`。

## 参考资料

- [SHOW CATALOGS](SHOW_CATALOGS.md)：查看 StarRocks 集群中的所有目录。
- [SHOW DATABASES](SHOW_DATABASES.md)：查看内部目录或外部目录中的所有数据库。
- [SET CATALOG](../data-definition/SET_CATALOG.md)：在目录之间切换。
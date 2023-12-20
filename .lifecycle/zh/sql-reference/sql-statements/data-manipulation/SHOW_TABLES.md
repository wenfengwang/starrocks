---
displayed_sidebar: English
---

# 显示表格

## 描述

展示 StarRocks 数据库或外部数据源中的所有表格，例如 Hive、Iceberg、Hudi 或 Delta Lake。

> **注意**
> 要查看外部数据源中的表格，您必须拥有相应外部目录的 **USAGE** 权限。

## 语法

```sql
SHOW TABLES [FROM <catalog_name>.<db_name>]
```

## 参数

|参数|必填|说明|
|---|---|---|
|catalog_name|No|内部目录或外部目录的名称。如果不指定该参数或将其设置为default_catalog，则返回 StarRocks 数据库中的表。如果将此参数设置为外部目录的名称，则返回表返回外部数据源的数据库中。您可以运行 SHOW CATALOGS 查看内部和外部目录。|
|db_name|否|数据库名称。如果不指定，则默认使用当前数据库。|

## 示例

示例 1：连接到 StarRocks 集群后，查看 default_catalog 中的 example_db 数据库的表格。以下两条语句具有相同的效果。

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

示例 2：在连接到该数据库后，查看当前数据库 example_db 中的表格。

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

示例 3：查看外部目录 hudi_catalog 中的 hudi_db 数据库的表格。

```plain
show tables from hudi_catalog.hudi_db;
+----------------------------+
| Tables_in_hudi_db          |
+----------------------------+
| hudi_sync_mor              |
| hudi_table1                |
+----------------------------+
```

另外，您可以执行 SET CATALOG 切换到外部目录 hudi_catalog，然后执行 SHOW TABLES FROM hudi_db。

## 参考资料

- [SHOW CATALOGS](SHOW_CATALOGS.md)：查看所有目录在一个StarRocks集群中。
- [SHOW DATABASES](SHOW_DATABASES.md)：查看内部目录或外部目录中的所有数据库。
- [SET CATALOG](../data-definition/SET_CATALOG.md)：在目录之间进行切换。

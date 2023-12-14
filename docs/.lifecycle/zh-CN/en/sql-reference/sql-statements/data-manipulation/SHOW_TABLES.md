---
displayed_sidebar: "Chinese"
---

# 显示表格

## 描述

显示StarRocks数据库中的所有表格或外部数据源中的数据库，例如Hive、Iceberg、Hudi或Delta Lake中的表格。

> **注意**
>
> 要查看外部数据源中的表格，您必须在对应该数据源的外部目录上具有USAGE权限。

## 语法

```sql
SHOW TABLES [FROM <catalog_name>.<db_name>]
```

## 参数

 **参数**          | **必需** | **描述**                                                     |
| ----------------- | -------- | ------------------------------------------------------------ |
| catalog_name | 否       | 内部目录或外部目录的名称。<ul><li>如果您不指定此参数或将其设置为 `default_catalog`，则返回StarRocks数据库中的表格。</li><li>如果将此参数设置为外部目录的名称，将返回外部数据源数据库中的表格。</li></ul> 您可以运行[SHOW CATALOGS](SHOW_CATALOGS.md)查看内部和外部目录。|
| db_name | 否       | 数据库名称。如果未指定，则默认使用当前数据库。 |

## 示例

示例1：连接到StarRocks集群后，查看`default_catalog`中数据库`example_db`中的表格。以下两个语句是等效的。

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

示例2：连接到当前数据库后，查看数据库`example_db`中的表格。

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

示例3：查看外部目录`hudi_catalog`中`hudi_db`数据库的表格。

```plain
show tables from hudi_catalog.hudi_db;
+----------------------------+
| Tables_in_hudi_db          |
+----------------------------+
| hudi_sync_mor              |
| hudi_table1                |
+----------------------------+
```

或者，您可以运行SET CATALOG切换到外部目录`hudi_catalog`，然后运行 `SHOW TABLES FROM hudi_db;`。

## 参考

- [SHOW CATALOGS](SHOW_CATALOGS.md)：查看StarRocks集群中的所有目录。
- [SHOW DATABASES](SHOW_DATABASES.md)：查看内部目录或外部目录中的所有数据库。
- [SET CATALOG](../data-definition/SET_CATALOG.md)：在目录之间切换。
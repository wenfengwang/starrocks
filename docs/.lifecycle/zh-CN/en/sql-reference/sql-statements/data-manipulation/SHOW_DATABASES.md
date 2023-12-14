---
displayed_sidebar: "Chinese"
---

# 展示数据库

## 描述

查看当前StarRocks集群或外部数据源中的数据库。StarRocks从v2.3开始支持查看外部数据源的数据库。

## 语法

```SQL
SHOW DATABASES [FROM <catalog_name>]
```

## 参数

| **参数**          | **必需**      | **描述**                                   |
| ----------------- | ------------ | ---------------------------------------- |
| catalog_name      | 否           | 内部目录或外部目录的名称。<ul><li>如果您不指定该参数或指定内部目录的名称（`default_catalog`），则可以查看当前StarRocks集群中的数据库。</li><li>如果将参数的值设置为外部目录的名称，则可以查看相应外部数据源中的数据库。您可以运行[SHOW CATALOGS](SHOW_CATALOGS.md)来查看内部和外部目录。</li></ul> |

## 示例

示例1：查看当前StarRocks集群中的数据库。

```SQL
SHOW DATABASES;
```

或者

```SQL
SHOW DATABASES FROM default_catalog;
```

上述语句的输出结果如下。

```SQL
+----------+
| Database |
+----------+
| db1      |
| db2      |
| db3      |
+----------+
```

示例2：通过使用外部目录`Hive1`查看Hive集群中的数据库。

```SQL
SHOW DATABASES FROM hive1;

+-----------+
| Database  |
+-----------+
| hive_db1  |
| hive_db2  |
| hive_db3  |
+-----------+
```

## 参考

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)
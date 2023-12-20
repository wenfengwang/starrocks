---
displayed_sidebar: English
---

# 显示数据库

## 描述

查看当前 StarRocks 集群或外部数据源中的数据库。StarRocks 从 v2.3 版本开始支持查看外部数据源的数据库。

## 语法

```SQL
SHOW DATABASES [FROM <catalog_name>]
```

## 参数

|**参数**|**必填**|**说明**|
|---|---|---|
|catalog_name|否|内部目录或外部目录的名称。<ul><li>如果不指定该参数或指定内部目录的名称（即 `default_catalog`），则可以查看当前 StarRocks 集群中的数据库。</li><li>如果将参数值设置为外部目录的名称，你可以查看对应外部数据源中的数据库。你可以运行 [SHOW CATALOGS](SHOW_CATALOGS.md) 来查看内部和外部目录。</li></ul>|

## 示例

示例 1：查看当前 StarRocks 集群中的数据库。

```SQL
SHOW DATABASES;
```

或

```SQL
SHOW DATABASES FROM default_catalog;
```

上述语句的输出如下所示。

```SQL
+----------+
| Database |
+----------+
| db1      |
| db2      |
| db3      |
+----------+
```

示例 2：使用 `Hive1` 外部目录查看 Hive 集群中的数据库。

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

## 参考资料

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)
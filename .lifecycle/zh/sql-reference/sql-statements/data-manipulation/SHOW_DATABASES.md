---
displayed_sidebar: English
---

# 查看数据库

## 描述

在当前 StarRocks 集群或外部数据源中查看数据库。从 v2.3 版本开始，StarRocks 支持查看外部数据源中的数据库。

## 语法

```SQL
SHOW DATABASES [FROM <catalog_name>]
```

## 参数

|参数|必填|说明|
|---|---|---|
|catalog_name|No|内部目录或外部目录的名称。如果不指定该参数或指定内部目录的名称（default_catalog），则可以查看当前 StarRocks 集群中的数据库。如果设置将参数的值设置为外部目录的名称，即可查看对应外部数据源中的数据库。您可以运行 SHOW CATALOGS 查看内部和外部目录。|

## 示例

示例 1：查看当前 StarRocks 集群中的数据库。

```SQL
SHOW DATABASES;
```

或

```SQL
SHOW DATABASES FROM default_catalog;
```

前述语句的输出结果如下。

```SQL
+----------+
| Database |
+----------+
| db1      |
| db2      |
| db3      |
+----------+
```

示例 2：通过使用 Hive1 外部目录来查看 Hive 集群中的数据库。

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

- [CREATE DATABASE（创建数据库）](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](SHOW_CREATE_DATABASE.md)（显示创建数据库的语句）
- [使用数据库](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)（描述数据库）
- [删除数据库](../data-definition/DROP_DATABASE.md)

---
displayed_sidebar: "Chinese"
---

# 显示数据库

## 功能

用于查看当前StarRocks集群或外部数据源中的数据库。

## 语法

```SQL
SHOW DATABASES [FROM <catalog_name>]
```

## 参数说明

| **参数**          | **必选** | **说明**                                                     |
| ----------------- | -------- | ------------------------------------------------------------ |
| catalog_name | 否       | 内部目录或外部目录的名称。<ul><li>如不指定或指定为内部目录名称，即 `default_catalog`，则查看当前StarRocks集群中的数据库。</li><li>如指定外部目录名称，则查看外部数据源中的数据库。您可以使用 [SHOW CATALOGS](SHOW_CATALOGS.md) 命令查看当前集群的内外部目录。</li></ul> |

## 示例

示例一：查看当前StarRocks集群中的数据库。

```SQL
SHOW DATABASES;
```

或

```SQL
SHOW DATABASES FROM default_catalog;
```

返回信息如下：

```SQL
+----------+
| 数据库   |
+----------+
| db1      |
| db2      |
| db3      |
+----------+
```

示例二：通过外部目录 `hive1` 查看Apache Hive™中的数据库。

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

## 相关参考

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)
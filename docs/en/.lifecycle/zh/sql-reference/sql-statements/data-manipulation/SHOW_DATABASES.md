---
displayed_sidebar: English
---

# 显示数据库

## 描述

查看您当前的 StarRocks 集群或外部数据源中的数据库。StarRocks 从 v2.3 开始支持查看外部数据源的数据库。

## 语法

```SQL
SHOW DATABASES [FROM <catalog_name>]
```

## 参数

| **参数**     | **必填** | **描述**                                              |
| ----------------- | ------------ | ------------------------------------------------------------ |
| catalog_name      | 否           | 内部目录或外部目录的名称。<ul><li>如果您不指定参数或指定内部目录的名称，默认为 `default_catalog`，则可以查看当前 StarRocks 集群中的数据库。</li><li>如果将参数的值设置为外部目录的名称，则可以查看相应外部数据源中的数据库。您可以运行 [SHOW CATALOGS](SHOW_CATALOGS.md) 来查看内部和外部目录。</li></ul> |

## 例子

示例 1：查看当前 StarRocks 集群中的数据库。

```SQL
SHOW DATABASES;
```

或

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

示例 2：通过使用 `Hive1` 外部目录查看 Hive 集群中的数据库。

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

## 引用

- [创建数据库](../data-definition/CREATE_DATABASE.md)
- [显示创建数据库](SHOW_CREATE_DATABASE.md)
- [使用](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [删除数据库](../data-definition/DROP_DATABASE.md)

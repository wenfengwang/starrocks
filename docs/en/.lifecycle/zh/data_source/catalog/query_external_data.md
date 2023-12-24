---
displayed_sidebar: English
---

# 查询外部数据

本主题将指导您通过使用外部目录从外部数据源查询数据。

## 先决条件

外部目录是基于外部数据源创建的。有关受支持的外部目录类型的信息，请参阅 [目录](../catalog/catalog_overview.md#catalog)。

## 程序

1. 连接到 StarRocks 集群。
   - 如果您使用 MySQL 客户端连接到 StarRocks 集群，连接后默认进入 `default_catalog`。
   - 如果您使用 JDBC 连接到 StarRocks 集群，可以在连接时通过指定 `default_catalog.db_name` 直接进入默认目录下的目标数据库。

2. （可选）执行以下语句以查看所有目录，并找到您已创建的外部目录。请参阅 [SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) 以检查此语句的输出。

      ```SQL
      SHOW CATALOGS;
      ```

3. （可选）执行以下语句以查看外部目录中的所有数据库。请参阅 [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) 以检查此语句的输出。

      ```SQL
      SHOW DATABASES FROM catalog_name;
      ```

4. （可选）执行以下语句以进入外部目录中的目标数据库。

      ```SQL
      USE catalog_name.db_name;
      ```

5. 查询外部数据。有关 SELECT 语句的更多用法，请参阅 [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)。

      ```SQL
      SELECT * FROM table_name;
      ```

      如果在前面的步骤中未指定外部目录和数据库，您可以在 select 查询中直接指定它们。

      ```SQL
      SELECT * FROM catalog_name.db_name.table_name;
      ```

## 例子

如果您已经创建了名为 `hive1` 的 Hive 目录，并希望使用 `hive1` 从 Apache Hive™ 集群中查询 `hive_db.hive_table` 的数据，则可以执行以下操作之一：

```SQL
USE hive1.hive_db;
SELECT * FROM hive_table limit 1;
```

或者

```SQL
SELECT * FROM hive1.hive_db.hive_table limit 1;
```

## 引用

要查询您的 StarRocks 集群中的数据，请参阅 [默认目录](../catalog/default_catalog.md)。

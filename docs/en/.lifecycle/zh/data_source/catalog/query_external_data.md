---
displayed_sidebar: English
---

# 查询外部数据

本主题将指导您如何使用外部目录从外部数据源查询数据。

## 先决条件

外部目录基于外部数据源创建。有关支持的外部目录类型的信息，请参阅[目录](../catalog/catalog_overview.md#catalog)。

## 程序

1. 连接您的 StarRocks 集群。
   - 如果您使用 MySQL 客户端连接 StarRocks 集群，连接后默认进入 `default_catalog`。
   - 如果您使用 JDBC 连接 StarRocks 集群，可以通过在连接时指定 `default_catalog.db_name` 直接进入默认目录中的目标数据库。

2. （可选）执行以下 SQL 语句以查看所有目录，并找到您创建的外部目录。请参阅 [SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) 以检查此语句的输出。

   ```SQL
   SHOW CATALOGS;
   ```

3. （可选）执行以下 SQL 语句以查看外部目录中的所有数据库。请参阅 [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) 以检查此语句的输出。

   ```SQL
   SHOW DATABASES FROM catalog_name;
   ```

4. （可选）执行以下 SQL 语句以切换到外部目录中的目标数据库。

   ```SQL
   USE catalog_name.db_name;
   ```

5. 查询外部数据。有关 SELECT 语句的更多用法，请参见 [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)。

   ```SQL
   SELECT * FROM table_name;
   ```

   如果您在前面的步骤中未指定外部目录和数据库，可以在 SELECT 查询中直接指定它们。

   ```SQL
   SELECT * FROM catalog_name.db_name.table_name;
   ```

## 示例

如果您已经创建了一个名为 `hive1` 的 Hive 目录，并且想要使用 `hive1` 来查询 Apache Hive™ 集群中的 `hive_db.hive_table` 数据，您可以执行以下操作之一：

```SQL
USE hive1.hive_db;
SELECT * FROM hive_table LIMIT 1;
```

或者

```SQL
SELECT * FROM hive1.hive_db.hive_table LIMIT 1;
```

## 参考资料

要从您的 StarRocks 集群查询数据，请参阅[默认目录](../catalog/default_catalog.md)。
---
displayed_sidebar: English
---

# 查询外部数据

本主题将指导您如何使用外部目录从外部数据源中查询数据。

## 先决条件

基于外部数据源创建外部目录。有关支持的外部目录类型的信息，请参考[目录](../catalog/catalog_overview.md#catalog)部分。

## 操作步骤

1. 连接到您的StarRocks集群。
   - 如果您使用MySQL客户端连接StarRocks集群，连接成功后会默认进入default_catalog。
   - 如果您使用JDBC连接StarRocks集群，可以在连接时通过指定default_catalog.db_name直接进入默认目录中的目标数据库。

2. （可选）执行以下语句以查看所有目录，并找到您所创建的外部目录。详细信息请查看[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)命令的输出结果。

   ```SQL
   SHOW CATALOGS;
   ```

3. （可选）执行以下语句以查看外部目录中的所有数据库。详细信息请查看[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)命令的输出结果。

   ```SQL
   SHOW DATABASES FROM catalog_name;
   ```

4. （可选）执行以下语句以切换到外部目录中的目标数据库。

   ```SQL
   USE catalog_name.db_name;
   ```

5. 查询外部数据。更多SELECT语句的使用方法，请参见[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)部分。

   ```SQL
   SELECT * FROM table_name;
   ```

   如果在之前的步骤中您未指定外部目录和数据库，您可以在SELECT查询中直接指定它们。

   ```SQL
   SELECT * FROM catalog_name.db_name.table_name;
   ```

## 示例

如果您已经创建了一个名为hive1的Hive目录，并且想要使用hive1来查询Apache Hive™集群中的hive_db.hive_table里的数据，您可以进行以下操作之一：

```SQL
USE hive1.hive_db;
SELECT * FROM hive_table limit 1;
```

或

```SQL
SELECT * FROM hive1.hive_db.hive_table limit 1;
```

## 参考资料

要查询数据从你的StarRocks集群，请参阅[默认目录](../catalog/default_catalog.md)。

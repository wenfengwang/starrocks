---
displayed_sidebar: English
---

# 默认目录

本文介绍了默认目录是什么，以及如何使用默认目录来查询 StarRocks 的内部数据。

StarRocks 2.3 及之后的版本提供了一个内部目录，用于管理 StarRocks 的内部数据。每个 StarRocks 集群只有一个名为 default_catalog 的内部目录。目前，您不能更改内部目录的名称或创建新的内部目录。

## 查询内部数据

1. 连接到您的 StarRocks 集群。
   - 如果您使用 MySQL 客户端连接 StarRocks 集群，连接成功后会默认进入 default_catalog。
   - 如果您使用 JDBC 连接 StarRocks 集群，可以在连接时通过指定 default_catalog.db_name 直接进入默认目录中的目标数据库。

2. （可选）使用[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)命令查看数据库列表：

   ```SQL
   SHOW DATABASES;
   ```

   或者

   ```SQL
   SHOW DATABASES FROM <catalog_name>;
   ```

3. （可选）使用 [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 命令在当前会话中切换到指定的目录：

   ```SQL
   SET CATALOG <catalog_name>;
   ```

   然后，使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 命令在当前会话中设置活动数据库：

   ```SQL
   USE <db_name>;
   ```

   或者，您也可以使用[USE](../../sql-reference/sql-statements/data-definition/USE.md)命令直接进入目标目录中的活动数据库：

   ```SQL
   USE <catalog_name>.<db_name>;
   ```

4. 使用 [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) 命令查询内部数据：

   ```SQL
   SELECT * FROM <table_name>;
   ```

   如果在前面的步骤中您没有指定活动数据库，那么您可以在 SELECT 查询中直接指定：

   ```SQL
   SELECT * FROM <db_name>.<table_name>;
   ```

   或者

   ```SQL
   SELECT * FROM default_catalog.<db_name>.<table_name>;
   ```

## 示例

要查询 olap_db.olap_table 中的数据，您可以执行以下任一操作：

```SQL
USE olap_db;
SELECT * FROM olap_table limit 1;
```

或者

```SQL
SELECT * FROM olap_db.olap_table limit 1;     
```

或者

```SQL
SELECT * FROM default_catalog.olap_db.olap_table limit 1;      
```

## 参考资料

要查询外部数据源的数据，请参考[Query external data](../catalog/query_external_data.md)。

---
displayed_sidebar: "Chinese"
---

# 默认目录

本文介绍什么是默认目录，以及如何使用默认目录查询StarRocks内部数据。

StarRocks 2.3及以上版本提供了内部目录，用于管理StarRocks的[内部数据](../catalog/catalog_overview.md#基本概念)。每个StarRocks集群都有且只有一个内部目录，名为 `default_catalog`。StarRocks暂不支持修改内部目录的名称，也不支持创建新的内部目录。

## 查询内部数据

1. 连接StarRocks。
   - 如从MySQL客户端连接到StarRocks。连接后，默认进入到 `default_catalog`。
   - 如使用JDBC连接到StarRocks，连接时即可通过 `default_catalog.db_name` 的方式指定要连接的数据库。
2. （可选）通过 [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) 查看数据库：

   ```SQL
   SHOW DATABASES;
   ```

   或

   ```SQL
   SHOW DATABASES FROM default_catalog;
   ```

3. （可选）通过 [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 切换当前会话生效的目录：

   ```SQL
   SET CATALOG <catalog_name>;
   ```

   再通过 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 指定当前会话生效的数据库：

   ```SQL
   USE <db_name>;
   ```

   或者，也可以通过 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 直接将会话切换到目标目录下的指定数据库：

   ```SQL
   USE <catalog_name>.<db_name>;
   ```

4. 通过 [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询内部数据：

   ```SQL
   SELECT * FROM <table_name>;
   ```

   如在以上步骤中未指定数据库，则可以在查询语句中直接指定。

   ```SQL
   SELECT * FROM <db_name>.<table_name>;
   ```

   或

   ```SQL
   SELECT * FROM default_catalog.<db_name>.<table_name>;
   ```

## 示例

如要查询`olap_db.olap_table`中的数据，操作如下：

 ```SQL
USE olap_db;
SELECT * FROM olap_table limit 1;
```

或

```SQL
SELECT * FROM olap_db.olap_table limit 1;   
```

或

```SQL
SELECT * FROM default_catalog.olap_db.olap_table limit 1;
```

## 更多操作

如要查询外部数据，请参见[查询外部数据](./query_external_data.md)。
---
displayed_sidebar: English
---

# 默认目录

本主题描述了默认目录是什么，以及如何通过使用默认目录查询 StarRocks的内部数据。

StarRocks 2.3及更高版本提供了一个内部目录来管理StarRocks的内部数据。每个StarRocks集群只有一个名为`default_catalog`的内部目录。目前，您无法修改内部目录的名称或创建新的内部目录。

## 查询内部数据

1. 连接到您的StarRocks集群。
   - 如果您使用MySQL客户端连接到StarRocks集群，则连接后默认进入`default_catalog`。
   - 如果您使用JDBC连接到StarRocks集群，可以在连接时通过指定`default_catalog.db_name`直接进入默认目录中的目标数据库。

2. （可选）使用[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)查看数据库：

      ```SQL
      SHOW DATABASES;
      ```

      或者

      ```SQL
      SHOW DATABASES FROM <catalog_name>;
      ```

3. （可选）使用[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)切换到当前会话中的目标目录：

    ```SQL
    SET CATALOG <catalog_name>;
    ```

    然后，使用[USE](../../sql-reference/sql-statements/data-definition/USE.md)指定当前会话中的活动数据库：

    ```SQL
    USE <db_name>;
    ```

    或者，您可以使用[USE](../../sql-reference/sql-statements/data-definition/USE.md)直接进入目标目录中的活动数据库：

    ```SQL
    USE <catalog_name>.<db_name>;
    ```

4. 使用[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)查询内部数据：

      ```SQL
      SELECT * FROM <table_name>;
      ```

      如果您在前面的步骤中未指定活动数据库，您可以在select查询中直接指定：

      ```SQL
      SELECT * FROM <db_name>.<table_name>;
      ```

      或者

      ```SQL
      SELECT * FROM default_catalog.<db_name>.<table_name>;
      ```

## 示例

要查询`olap_db.olap_table`中的数据，您可以执行以下操作之一：

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

## 参考

要查询外部数据源的数据，请参阅[查询外部数据](../catalog/query_external_data.md)。

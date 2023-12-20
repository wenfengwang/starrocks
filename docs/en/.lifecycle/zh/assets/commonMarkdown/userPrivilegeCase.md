
我们建议您自定义角色来管理权限和用户。以下示例对一些常见场景的几种权限组合进行了分类。

#### 授予 StarRocks 表的全局只读权限

```SQL
-- 创建一个角色。
CREATE ROLE read_only;
-- 授予该角色所有目录的 USAGE 权限。
GRANT USAGE ON ALL CATALOGS TO ROLE read_only;
-- 授予该角色查询所有表的权限。
GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE read_only;
-- 授予该角色查询所有视图的权限。
GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE read_only;
-- 授予该角色查询所有物化视图以及使用它们加速查询的权限。
GRANT SELECT ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE read_only;
```

您还可以进一步授予在查询中使用 UDF 的权限：

```SQL
-- 授予该角色所有数据库级别 UDF 的 USAGE 权限。
GRANT USAGE ON ALL FUNCTIONS IN ALL DATABASES TO ROLE read_only;
-- 授予该角色所有全局 UDF 的 USAGE 权限。
GRANT USAGE ON ALL GLOBAL FUNCTIONS TO ROLE read_only;
```

#### 授予 StarRocks 表的全局写入权限

```SQL
-- 创建一个角色。
CREATE ROLE write_only;
-- 授予该角色所有目录的 USAGE 权限。
GRANT USAGE ON ALL CATALOGS TO ROLE write_only;
-- 授予该角色所有表的 INSERT 和 UPDATE 权限。
GRANT INSERT, UPDATE ON ALL TABLES IN ALL DATABASES TO ROLE write_only;
-- 授予该角色所有物化视图的 REFRESH 权限。
GRANT REFRESH ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE write_only;
```

#### 授予特定外部目录的只读权限

```SQL
-- 创建一个角色。
CREATE ROLE read_catalog_only;
-- 授予该角色目标目录的 USAGE 权限。
GRANT USAGE ON CATALOG hive_catalog TO ROLE read_catalog_only;
-- 切换到相应的目录。
SET CATALOG hive_catalog;
-- 授予该角色查询所有数据库中所有表和视图的权限。
GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE read_catalog_only;
GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE read_catalog_only;
```

注意：您只能查询 Hive 表视图（自 v3.1 起）。

#### 授予特定外部目录的只写权限

您只能将数据写入 Iceberg 表（自 v3.1 起）和 Hive 表（自 v3.2 起）。

```SQL
-- 创建一个角色。
CREATE ROLE write_catalog_only;
-- 授予该角色目标目录的 USAGE 权限。
GRANT USAGE ON CATALOG iceberg_catalog TO ROLE write_catalog_only;
-- 切换到相应的目录。
SET CATALOG iceberg_catalog;
-- 授予该角色向 Iceberg 表写入数据的权限。
GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE write_catalog_only;
```

#### 授予在全局、数据库、表和分区级别执行备份和恢复操作的权限

- 授予执行全局备份和恢复操作的权限：

  执行全局备份和恢复操作的权限允许角色备份和恢复任何数据库、表或分区。它需要在 SYSTEM 级别的 REPOSITORY 权限、在默认目录中创建数据库的权限、在任何数据库中创建表的权限，以及在任何表上加载和导出数据的权限。

  ```SQL
  -- 创建一个角色。
  CREATE ROLE recover;
  -- 授予该角色 SYSTEM 级别的 REPOSITORY 权限。
  GRANT REPOSITORY ON SYSTEM TO ROLE recover;
  -- 授予该角色在默认目录中创建数据库的权限。
  GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE recover;
  -- 授予该角色在任何数据库中创建表的权限。
  GRANT CREATE TABLE ON ALL DATABASES TO ROLE recover;
  -- 授予该角色在任何表上加载和导出数据的权限。
  GRANT INSERT, EXPORT ON ALL TABLES IN ALL DATABASES TO ROLE recover;
  ```

- 授予执行数据库级备份和恢复操作的权限：

  执行数据库级备份和恢复操作的权限需要 SYSTEM 级别的 REPOSITORY 权限、在默认目录中创建数据库的权限、在任何数据库中创建表的权限、将数据加载到任何表中的权限，以及从要备份的数据库中的任何表导出数据的权限。

  ```SQL
  -- 创建一个角色。
  CREATE ROLE recover_db;
  -- 授予该角色 SYSTEM 级别的 REPOSITORY 权限。
  GRANT REPOSITORY ON SYSTEM TO ROLE recover_db;
  -- 授予该角色创建数据库的权限。
  GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE recover_db;
  -- 授予该角色创建表的权限。
  GRANT CREATE TABLE ON ALL DATABASES TO ROLE recover_db;
  -- 授予该角色向任何表中加载数据的权限。
  GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE recover_db;
  -- 授予该角色从要备份的数据库中的任何表导出数据的权限。
  GRANT EXPORT ON ALL TABLES IN DATABASE <db_name> TO ROLE recover_db;
  ```

- 授予执行表级备份和恢复操作的权限：

  执行表级备份和恢复操作的权限需要 SYSTEM 级别的 REPOSITORY 权限、在相应数据库中创建表的权限、向数据库中任意表加载数据的权限，以及从要备份的表中导出数据的权限。

  ```SQL
  -- 创建一个角色。
  CREATE ROLE recover_tbl;
  -- 授予该角色 SYSTEM 级别的 REPOSITORY 权限。
  GRANT REPOSITORY ON SYSTEM TO ROLE recover_tbl;
  -- 授予该角色在相应数据库中创建表的权限。
  GRANT CREATE TABLE ON DATABASE <db_name> TO ROLE recover_tbl;
  -- 授予该角色向数据库中任意表加载数据的权限。
  GRANT INSERT ON ALL TABLES IN DATABASE <db_name> TO ROLE recover_tbl;
  -- 授予该角色从要备份的表中导出数据的权限。
  GRANT EXPORT ON TABLE <table_name> TO ROLE recover_tbl;     
  ```

- 授予执行分区级备份和恢复操作的权限：

  执行分区级备份和恢复操作的权限需要 SYSTEM 级别的 REPOSITORY 权限，以及在相应表上加载和导出数据的权限。

  ```SQL
  -- 创建一个角色。
  CREATE ROLE recover_par;
  -- 授予该角色 SYSTEM 级别的 REPOSITORY 权限。
  GRANT REPOSITORY ON SYSTEM TO ROLE recover_par;
  -- 授予该角色在相应表上加载和导出数据的权限。
  GRANT INSERT, EXPORT ON TABLE <table_name> TO ROLE recover_par;
  ```
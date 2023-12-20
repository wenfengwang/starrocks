
我们建议您自定义角色以管理权限和用户。下面的例子为一些常见场景下的权限组合进行了分类。

#### 为 StarRocks 表授予全局只读权限

```SQL
-- Create a role.
CREATE ROLE read_only;
-- Grant the USAGE privilege on all catalogs to the role.
GRANT USAGE ON ALL CATALOGS TO ROLE read_only;
-- Grant the privilege to query all tables to the role.
GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE read_only;
-- Grant the privilege to query all views to the role.
GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE read_only;
-- Grant the privilege to query all materialized views and the privilege to accelerate queries with them to the role.
GRANT SELECT ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE read_only;
```

您还可以进一步授予查询中使用 UDF 的权限：

```SQL
-- Grant the USAGE privilege on all database-level UDF to the role.
GRANT USAGE ON ALL FUNCTIONS IN ALL DATABASES TO ROLE read_only;
-- Grant the USAGE privilege on global UDF to the role.
GRANT USAGE ON ALL GLOBAL FUNCTIONS TO ROLE read_only;
```

#### 为 StarRocks 表授予全局写入权限

```SQL
-- Create a role.
CREATE ROLE write_only;
-- Grant the USAGE privilege on all catalogs to the role.
GRANT USAGE ON ALL CATALOGS TO ROLE write_only;
-- Grant the INSERT and UPDATE privileges on all tables to the role.
GRANT INSERT, UPDATE ON ALL TABLES IN ALL DATABASES TO ROLE write_only;
-- Grant the REFRESH privilege on all materialized views to the role.
GRANT REFRESH ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE write_only;
```

#### 为特定外部目录授予只读权限

```SQL
-- Create a role.
CREATE ROLE read_catalog_only;
-- Grant the USAGE privilege on the destination catalog to the role.
GRANT USAGE ON CATALOG hive_catalog TO ROLE read_catalog_only;
-- Switch to the corresponding catalog.
SET CATALOG hive_catalog;
-- Grant the privileges to query all tables and all views in all databases.
GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE read_catalog_only;
GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE read_catalog_only;
```

注意：您只能查询 Hive 表的视图（3.1 版本之后）。

#### 为特定外部目录授予只写权限

您仅能向 Iceberg 表（3.1 版本之后）和 Hive 表（3.2 版本之后）写入数据。

```SQL
-- Create a role.
CREATE ROLE write_catalog_only;
-- Grant the USAGE privilege on the destination catalog to the role.
GRANT USAGE ON CATALOG iceberg_catalog TO ROLE read_catalog_only;
-- Switch to the corresponding catalog.
SET CATALOG iceberg_catalog;
-- Grant the privilege to write data into Iceberg tables.
GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE write_catalog_only;
```

#### 授予在全局、数据库、表和分区级别进行备份和恢复操作的权限

- 授予进行全局备份和恢复操作的权限：

  进行全局备份和恢复操作的权限允许角色备份和恢复任意数据库、表或分区。这需要在系统级别上具有仓库（REPOSITORY）权限，在默认目录下创建数据库的权限，在任意数据库中创建表的权限，以及对任意表进行数据加载和导出的权限。

  ```SQL
  -- Create a role.
  CREATE ROLE recover;
  -- Grant the REPOSITORY privilege on the SYSTEM level.
  GRANT REPOSITORY ON SYSTEM TO ROLE recover;
  -- Grant the privilege to create databases in the default catalog.
  GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE recover;
  -- Grant the privilege to create tables in any database.
  GRANT CREATE TABLE ON ALL DATABASE TO ROLE recover;
  -- Grant the privilege to load and export data on any table.
  GRANT INSERT, EXPORT ON ALL TABLES IN ALL DATABASES TO ROLE recover;
  ```

- 授予进行数据库级别备份和恢复操作的权限：

  进行数据库级别备份和恢复操作的权限需要在系统级别上具有仓库（REPOSITORY）权限，在默认目录下创建数据库的权限，在任何数据库中创建表的权限，将数据加载到任意表中的权限，以及将要备份的数据库中任何表的数据导出的权限。

  ```SQL
  -- Create a role.
  CREATE ROLE recover_db;
  -- Grant the REPOSITORY privilege on the SYSTEM level.
  GRANT REPOSITORY ON SYSTEM TO ROLE recover_db;
  -- Grant the privilege to create databases.
  GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE recover_db;
  -- Grant the privilege to create tables.
  GRANT CREATE TABLE ON ALL DATABASE TO ROLE recover_db;
  -- Grant the privilege to load data into any table.
  GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE recover_db;
  -- Grant the privilege to export data from any table in the database to be backed up.
  GRANT EXPORT ON ALL TABLES IN DATABASE <db_name> TO ROLE recover_db;
  ```

- 授予进行表级备份和恢复操作的权限：

  进行表级备份和恢复操作的权限需要在系统级别上具有仓库（REPOSITORY）权限，在相应数据库中创建表的权限，向该数据库中任意表加载数据的权限，以及将要备份的表的数据导出的权限。

  ```SQL
  -- Create a role.
  CREATE ROLE recover_tbl;
  -- Grant the REPOSITORY privilege on the SYSTEM level.
  GRANT REPOSITORY ON SYSTEM TO ROLE recover_tbl;
  -- Grant the privilege to create tables in corresponding databases.
  GRANT CREATE TABLE ON DATABASE <db_name> TO ROLE recover_tbl;
  -- Grant the privilege to load data into any table in a database.
  GRANT INSERT ON ALL TABLES IN DATABASE <db_name> TO ROLE recover_db;
  -- Grant the privilege to export data from the table you want to back up.
  GRANT EXPORT ON TABLE <table_name> TO ROLE recover_tbl;     
  ```

- 授予进行分区级备份和恢复操作的权限：

  进行分区级备份和恢复操作的权限需要在系统级别上具有仓库（REPOSITORY）权限，以及对应表上进行数据加载和导出的权限。

  ```SQL
  -- Create a role.
  CREATE ROLE recover_par;
  -- Grant the REPOSITORY privilege on the SYSTEM level.
  GRANT REPOSITORY ON SYSTEM TO ROLE recover_par;
  -- Grant the privilege to load and export data on the corresponding table.
  GRANT INSERT, EXPORT ON TABLE <table_name> TO ROLE recover_par;
  ```

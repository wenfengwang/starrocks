
我们建议您自定义角色以管理权限和用户。以下示例对一些常见场景的权限组合进行了分类。

#### 授予 StarRocks 表的全局只读权限

   ```SQL
   -- 创建一个角色。
   CREATE ROLE 只读角色;
   -- 授予角色对所有目录的 USAGE 权限。
   GRANT USAGE ON ALL CATALOGS TO ROLE 只读角色;
   -- 授予角色对所有数据库中的所有表的查询权限。
   GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE 只读角色;
   -- 授予角色对所有数据库中的所有视图的查询权限。
   GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE 只读角色;
   -- 授予角色对所有数据库中的所有物化视图的查询权限以及使用加速查询的权限。
   GRANT SELECT ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE 只读角色;
   ```

   您还可以进一步授予角色在查询中使用UDF的权限：

   ```SQL
   -- 授予角色对所有数据库级别的UDF的 USAGE 权限。
   GRANT USAGE ON ALL FUNCTIONS IN ALL DATABASES TO ROLE 只读角色;
   -- 授予角色对全局UDF的 USAGE 权限。
   GRANT USAGE ON ALL GLOBAL FUNCTIONS TO ROLE 只读角色;
   ```

#### 授予 StarRocks 表的全局写入权限

   ```SQL
   -- 创建一个角色。
   CREATE ROLE 只写角色;
   -- 授予角色对所有目录的 USAGE 权限。
   GRANT USAGE ON ALL CATALOGS TO ROLE 只写角色;
   -- 授予角色对所有数据库中的所有表的 INSERT 和 UPDATE 权限。
   GRANT INSERT, UPDATE ON ALL TABLES IN ALL DATABASES TO ROLE 只写角色;
   -- 授予角色对所有数据库中的所有物化视图的 REFRESH 权限。
   GRANT REFRESH ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE 只写角色;
   ```

#### 授予对特定外部目录的只读权限

   ```SQL
   -- 创建一个角色。
   CREATE ROLE 只读目录角色;
   -- 授予角色对目标目录的 USAGE 权限。
   GRANT USAGE ON CATALOG hive_catalog TO ROLE 只读目录角色;
   -- 切换到相应的目录。
   SET CATALOG hive_catalog;
   -- 授予角色对所有数据库中的所有表和视图的查询权限。
   GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE 只读目录角色;
   GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE 只读目录角色;
   ```

   注意：您只能查询Hive表视图（自v3.1起）。

#### 授予对特定外部目录的只写权限

您只能向Iceberg表（自v3.1起）和Hive表（自v3.2起）写入数据。

   ```SQL
   -- 创建一个角色。
   CREATE ROLE 只写目录角色;
   -- 授予角色对目标目录的 USAGE 权限。
   GRANT USAGE ON CATALOG iceberg_catalog TO ROLE 只写目录角色;
   -- 切换到相应的目录。
   SET CATALOG iceberg_catalog;
   -- 授予角色向所有数据库中的所有表写入数据的权限。
   GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE 只写目录角色;
   ```

#### 授予在全局、数据库、表和分区级别执行备份和还原操作的权限

- 授予执行全局备份和还原操作的权限：

     执行全局备份和还原操作的权限允许该角色备份和还原任何数据库、表或分区。它需要在SYSTEM级别上具有REPOSITORY权限，在默认目录中创建数据库的权限，在任何数据库中创建表的权限，并在任何表上加载和导出数据的权限。

     ```SQL
     -- 创建一个角色。
     CREATE ROLE 恢复角色;
     -- 授予在SYSTEM级别上的REPOSITORY权限。
     GRANT REPOSITORY ON SYSTEM TO ROLE 恢复角色;
     -- 授予在默认目录中创建数据库的权限。
     GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE 恢复角色;
     -- 授予在任何数据库中创建表的权限。
     GRANT CREATE TABLE ON ALL DATABASE TO ROLE 恢复角色;
     -- 授予在任何表上加载和导出数据的权限。
     GRANT INSERT, EXPORT ON ALL TABLES IN ALL DATABASES TO ROLE 恢复角色;
     ```

- 授予执行数据库级备份和还原操作的权限：

     执行数据库级备份和还原操作的权限需要在SYSTEM级别上具有REPOSITORY权限，在默认目录中创建数据库的权限，在任何数据库中创建表的权限，在任何表中加载数据的权限，并从要备份的数据库中导出数据的权限。

     ```SQL
     -- 创建一个角色。
     CREATE ROLE 恢复数据库角色;
     -- 授予在SYSTEM级别上的REPOSITORY权限。
     GRANT REPOSITORY ON SYSTEM TO ROLE 恢复数据库角色;
     -- 授予创建数据库的权限。
     GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE 恢复数据库角色;
     -- 授予创建表的权限。
     GRANT CREATE TABLE ON ALL DATABASE TO ROLE 恢复数据库角色;
     -- 授予在任何表中加载数据的权限。
     GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE 恢复数据库角色;
     -- 授予从要备份的数据库中导出数据的权限。
     GRANT EXPORT ON ALL TABLES IN DATABASE <db_name> TO ROLE 恢复数据库角色;
     ```

- 授予执行表级备份和还原操作的权限：

     执行表级备份和还原操作的权限需要在SYSTEM级别上具有REPOSITORY权限，在相应数据库中创建表的权限，在数据库中的任何表中加载数据的权限，并从要备份的表中导出数据的权限。

     ```SQL
     -- 创建一个角色。
     CREATE ROLE 恢复表角色;
     -- 授予在SYSTEM级别上的REPOSITORY权限。
     GRANT REPOSITORY ON SYSTEM TO ROLE 恢复表角色;
     -- 授予在相应数据库中创建表的权限。
     GRANT CREATE TABLE ON DATABASE <db_name> TO ROLE 恢复表角色;
     -- 授予在数据库中的任何表中加载数据的权限。
     GRANT INSERT ON ALL TABLES IN DATABASE <db_name> TO ROLE 恢复数据库角色;
     -- 授予从要备份的表中导出数据的权限。
     GRANT EXPORT ON TABLE <table_name> TO ROLE 恢复表角色;     
     ```

- 授予执行分区级备份和还原操作的权限：

     执行分区级备份和还原操作的权限需要在SYSTEM级别上具有REPOSITORY权限，并在相应表上加载和导出数据的权限。

     ```SQL
     -- 创建一个角色。
     CREATE ROLE 恢复分区角色;
     -- 授予在SYSTEM级别上的REPOSITORY权限。
     GRANT REPOSITORY ON SYSTEM TO ROLE 恢复分区角色;
     -- 授予在相应表上加载和导出数据的权限。
     GRANT INSERT, EXPORT ON TABLE <table_name> TO ROLE 恢复分区角色;
     ```

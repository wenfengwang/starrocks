
我们建议您自定义角色来管理特权和用户。以下示例列举了一些常见情景下的一些权限组合。

#### 授予StarRocks表的全局只读权限

   ```SQL
   -- 创建一个角色。
   CREATE ROLE 只读;
   -- 授予角色对所有目录的使用权限。
   GRANT 在所有目录上的使用权限 TO ROLE 只读;
   -- 授予角色对所有数据库中的所有表的查询权限。
   GRANT 在所有数据库中的所有表上的SELECT权限 TO ROLE 只读;
   -- 授予角色对所有数据库中的所有视图的查询权限。
   GRANT 在所有数据库中的所有视图上的SELECT权限 TO ROLE 只读;
   -- 授予角色对所有数据库中的所有物化视图的查询权限和用于加速查询的权限。
   GRANT 在所有数据库中的所有物化视图上的SELECT权限 TO ROLE 只读;
   ```

   您还可以进一步授予角色在查询中使用UDF的权限：

   ```SQL
   -- 授予角色对所有数据库级别UDF的使用权限。
   GRANT 在所有数据库中的所有UDF上的使用权限 TO ROLE 只读;
   -- 授予角色对全局UDF的使用权限。
   GRANT 对所有全局UDF的使用权限 TO ROLE 只读;
   ```

#### 授予StarRocks表的全局写权限

   ```SQL
   -- 创建一个角色。
   CREATE ROLE 只写;
   -- 授予角色对所有目录的使用权限。
   GRANT 在所有目录上的使用权限 TO ROLE 只写;
   -- 授予角色对所有数据库中的所有表的插入和更新权限。
   GRANT 在所有数据库中的所有表上的INSERT, UPDATE权限 TO ROLE 只写;
   -- 授予角色对所有数据库中的所有物化视图的刷新权限。
   GRANT 在所有数据库中的所有物化视图上的REFRESH权限 TO ROLE 只写;
   ```

#### 授予特定外部目录的只读权限

   ```SQL
   -- 创建一个角色。
   CREATE ROLE 只读目录;
   -- 授予角色对目标目录的使用权限。
   GRANT 对目录hive_catalog上的使用权限 TO ROLE 只读目录;
   -- 切换到相应的目录。
   SET 目录 hive_catalog;
   -- 授予角色在所有数据库中查询所有表和视图的权限。
   GRANT 在所有数据库中的所有表上的SELECT权限 TO ROLE 只读目录;
   GRANT 在所有数据库中的所有视图上的SELECT权限 TO ROLE 只读目录;
   ```

   注意：您只能查询Hive表视图（自v3.1起）。

#### 授予特定外部目录的只写权限

您只能将数据写入Iceberg表（自v3.1起）和Hive表（自v3.2起）。

   ```SQL
   -- 创建一个角色。
   CREATE ROLE 只写目录;
   -- 授予角色对目标目录的使用权限。
   GRANT 对目录iceberg_catalog上的使用权限 TO ROLE 只读目录;
   -- 切换到相应的目录。
   SET 目录 iceberg_catalog;
   -- 授予角色将数据写入Iceberg表的权限。
   GRANT 在所有数据库中的所有表上的INSERT权限 TO ROLE 只写目录;
   ```

#### 授予执行全局、数据库、表和分区级别备份和恢复操作的权限

- 授予执行全局备份和恢复操作的权限：

     执行全局备份和恢复操作的权限允许角色备份和恢复任何数据库、表或分区。它需要在SYSTEM级别具有REPOSITORY权限，在默认目录创建数据库的权限，在任何数据库创建表的权限，以及在任何表上加载和导出数据的权限。

     ```SQL
     -- 创建一个角色。
     CREATE ROLE 恢复;
     -- 授予在SYSTEM级别的REPOSITORY权限。
     GRANT 在SYSTEM上的REPOSITORY权限 TO ROLE 恢复;
     -- 授予在默认目录中创建数据库的权限。
     GRANT 在默认目录上的创建数据库权限 TO ROLE 恢复;
     -- 授予在任何数据库中创建表的权限。
     GRANT 在所有数据库上的创建表权限 TO ROLE 恢复;
     -- 授予在任何数据库中加载和导出数据的权限。
     GRANT 在所有数据库中的所有表上的INSERT, EXPORT权限 TO ROLE 恢复;
     ```

- 授予执行数据库级别备份和恢复操作的权限：

     执行数据库级别备份和恢复操作的权限需要在SYSTEM级别具有REPOSITORY权限，在默认目录创建数据库的权限，在任何数据库创建表的权限，加载数据到任何表的权限，以及在要备份的数据库中从任何表导出数据的权限。

     ```SQL
     -- 创建一个角色。
     CREATE ROLE 恢复_db;
     -- 授予在SYSTEM级别的REPOSITORY权限。
     GRANT 在SYSTEM上的REPOSITORY权限 TO ROLE 恢复_db;
     -- 授予创建数据库的权限。
     GRANT 在默认目录上的创建数据库权限 TO ROLE 恢复_db;
     -- 授予创建表的权限。
     GRANT 在所有数据库上的创建表权限 TO ROLE 恢复_db;
     -- 授予在任何表中加载数据的权限。
     GRANT 在所有数据库中的所有表上的INSERT权限 TO ROLE 恢复_db;
     -- 授予从要备份的数据库中导出数据的权限。
     GRANT 在数据库<db_name>上的所有表的EXPORT权限 TO ROLE 恢复_db;
     ```

- 授予执行表级别备份和恢复操作的权限：

     执行表级别备份和恢复操作的权限需要在SYSTEM级别具有REPOSITORY权限，在相应的数据库中创建表的权限，在数据库中加载数据到任何表的权限，以及从要备份的表中导出数据的权限。

     ```SQL
     -- 创建一个角色。
     CREATE ROLE 恢复_tbl;
     -- 授予在SYSTEM级别的REPOSITORY权限。
     GRANT 在SYSTEM上的REPOSITORY权限 TO ROLE 恢复_tbl;
     -- 授予在相应的数据库中创建表的权限。
     GRANT 在数据库<db_name>上的创建表权限 TO ROLE 恢复_tbl;
     -- 授予在数据库中加载数据到任何表的权限。
     GRANT 在数据库<db_name>上的所有表上的INSERT权限 TO ROLE 恢复_db;
     -- 授予从要备份的表中导出数据的权限。
     GRANT 在表<表名>上的EXPORT权限 TO ROLE 恢复_tbl;     
     ```

- 授予执行分区级别备份和恢复操作的权限：

     执行分区级别备份和恢复操作的权限需要在SYSTEM级别具有REPOSITORY权限，并且可以在相应的表上加载和导出数据。

     ```SQL
     -- 创建一个角色。
     CREATE ROLE 恢复_par;
     -- 授予在SYSTEM级别的REPOSITORY权限。
     GRANT 在SYSTEM上的REPOSITORY权限 TO ROLE 恢复_par;
     -- 授予在相应表上加载和导出数据的权限。
     GRANT 在表<表名>上的INSERT, EXPORT权限 TO ROLE 恢复_par;
     ```
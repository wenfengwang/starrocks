---
displayed_sidebar: "中文"
---

# 所有命令

本文罗列了所有 StarRocks 支持的 SQL 命令，并按照命令的功能进行了分类。

- [所有命令](#所有命令)
  - [用户账户管理](#用户账户管理)
  - [集群管理](#集群管理)
    - [FE，BE，CN，Broker，process](#febecnbrokerprocess)
    - [资源组](#资源组)
    - [存储卷](#存储卷)
    - [表，tablet，副本](#表tablet副本)
    - [文件，索引，变量](#文件索引变量)
    - [SQL 黑名单](#sql-黑名单)
    - [插件](#插件)
  - [导入，导出](#导入导出)
    - [Routine load](#routine-load)
    - [其他导入](#其他导入)
    - [导出](#导出)
    - [ETL 任务](#etl-任务)
  - [数据目录 (Catalog)，数据库，资源](#数据目录-catalog数据库资源)
    - [Catalog](#catalog)
    - [数据库](#数据库)
    - [资源](#资源)
  - [建表，分区](#建表分区)
  - [视图，物化视图](#视图物化视图)
    - [视图](#视图)
    - [物化视图](#物化视图)
  - [函数，SELECT](#函数select)
  - [CBO 统计信息](#cbo-统计信息)
  - [备份与恢复](#备份与恢复)
  - [工具辅助语句](#工具辅助语句)

## 用户账户管理

管理用户、角色、和权限。

- [ALTER USER](./account-management/ALTER_USER.md)
- [CREATE ROLE](./account-management/CREATE_ROLE.md)
- [CREATE USER](./account-management/CREATE_USER.md)
- [DROP ROLE](./account-management/DROP_ROLE.md)
- [DROP USER](./account-management/DROP_USER.md)
- [EXECUTE AS](./account-management/EXECUTE_AS.md)
- [GRANT](./account-management/GRANT.md)
- [REVOKE](./account-management/REVOKE.md)
- [SET DEFAULT ROLE](./account-management/SET_DEFAULT_ROLE.md)
- [SET PASSWORD](./account-management/SET_PASSWORD.md)
- [SET PROPERTY](./account-management/SET_PROPERTY.md)
- [SET ROLE](./account-management/SET_ROLE.md)
- [SHOW AUTHENTICATION](./account-management/SHOW_AUTHENTICATION.md)
- [SHOW GRANTS](./account-management/SHOW_GRANTS.md)
- [SHOW PROPERTY](./account-management/SHOW_PROPERTY.md)
- [SHOW ROLES](./account-management/SHOW_ROLES.md)
- [SHOW USERS](./account-management/SHOW_USERS.md)

## 集群管理

管理集群，包括 FE、BE、Compute Node (CN)、资源组 (Resource Group)、存储卷（Storage Volume）、表、Tablet、副本 (Replica)、文件、索引（Index）、变量（Variable）、插件（Plugin）等。

### FE，BE，CN，Broker，process

- [ADMIN SET CONFIG](./Administration/ADMIN_SET_CONFIG.md)
- [ADMIN SHOW CONFIG](./Administration/ADMIN_SHOW_CONFIG.md)
- [ALTER SYSTEM](./Administration/ALTER_SYSTEM.md)
- [CANCEL DECOMMISSION](./Administration/CANCEL_DECOMMISSION.md)
- [KILL](./Administration/KILL.md)
- [SHOW BACKENDS](./Administration/SHOW_BACKENDS.md)
- [SHOW BROKER](./Administration/SHOW_BROKER.md)
- [SHOW COMPUTE NODES](./Administration/SHOW_COMPUTE_NODES.md)
- [SHOW FRONTENDS](./Administration/SHOW_FRONTENDS.md)
- [SHOW PROC](./Administration/SHOW_PROC.md)
- [SHOW PROCESSLIST](./Administration/SHOW_PROCESSLIST.md)
- [SHOW RUNNING QUERIES](./Administration/SHOW_RUNNING_QUERIES.md)

### 资源组

- [CREATE RESOURCE GROUP](./Administration/CREATE_RESOURCE_GROUP.md)
- [ALTER RESOURCE GROUP](./Administration/ALTER_RESOURCE_GROUP.md)
- [DROP RESOURCE GROUP](./Administration/DROP_RESOURCE_GROUP.md)
- [SHOW RESOURCE GROUP](./Administration/SHOW_RESOURCE_GROUP.md)
- [SHOW USAGE RESOURCE GROUPS](./Administration/SHOW_USAGE_RESOURCE_GROUPS.md)

### 存储卷

- [ALTER STORAGE VOLUME](./Administration/ALTER_STORAGE_VOLUME.md)
- [CREATE STORAGE VOLUME](./Administration/CREATE_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./Administration/DESC_STORAGE_VOLUME.md)
- [DROP STORAGE VOLUME](./Administration/DROP_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./Administration/SET_DEFAULT_STORAGE_VOLUME.md)
- [SHOW STORAGE VOLUMES](./Administration/SHOW_STORAGE_VOLUMES.md)

### 表，tablet，副本

- [ADMIN CANCEL REPAIR TABLE](./Administration/ADMIN_CANCEL_REPAIR.md)
- [ADMIN CHECK TABLET](./Administration/ADMIN_CHECK_TABLET.md)
- [ADMIN REPAIR TABLE](./Administration/ADMIN_REPAIR.md)
- [ADMIN SET REPLICA STATUS](./Administration/ADMIN_SET_REPLICA_STATUS.md)
- [ADMIN SHOW REPLICA DISTRIBUTION](./Administration/ADMIN_SHOW_REPLICA_DISTRIBUTION.md)
- [ADMIN SHOW REPLICA STATUS](./Administration/ADMIN_SHOW_REPLICA_STATUS.md)
- [RECOVER](./data-definition/RECOVER.md)
- [SHOW TABLE STATUS](./Administration/SHOW_TABLE_STATUS.md)

### 文件，索引，变量

- [CREATE FILE](./Administration/CREATE_FILE.md)
- [CREATE INDEX](./data-definition/CREATE_INDEX.md)
- [DROP FILE](./Administration/DROP_FILE.md)
- [DROP INDEX](./data-definition/DROP_INDEX.md)
- [SET (variable)](./Administration/SET.md)
- [SHOW FILE](./Administration/SHOW_FILE.md)
- [SHOW FULL COLUMNS](./Administration/SHOW_FULL_COLUMNS.md)
- [SHOW INDEX](./Administration/SHOW_INDEX.md)
- [SHOW VARIABLES](./Administration/SHOW_VARIABLES.md)

### SQL 黑名单

- [ADD SQLBLACKLIST](./Administration/ADD_SQLBLACKLIST.md)
- [SHOW SQLBLACKLIST](./Administration/SHOW_SQLBLACKLIST.md)
- [DELETE SQLBLACKLIST](./Administration/DELETE_SQLBLACKLIST.md)

### 插件

- [INSTALL PLUGIN](./Administration/INSTALL_PLUGIN.md)
- [SHOW PLUGINS](./Administration/SHOW_PLUGINS.md)
- [UNINSTALL PLUGIN](./Administration/UNINSTALL_PLUGIN.md)

## 导入，导出

### Routine load

- [ALTER ROUTINE LOAD](./data-manipulation/ALTER_ROUTINE_LOAD.md)
- [CREATE ROUTINE LOAD](./data-manipulation/CREATE_ROUTINE_LOAD.md)
- [PAUSE ROUTINE LOAD](./data-manipulation/PAUSE_ROUTINE_LOAD.md)
- [RESUME ROUTINE LOAD](./data-manipulation/RESUME_ROUTINE_LOAD.md)
- [SHOW ROUTINE LOAD](./data-manipulation/SHOW_ROUTINE_LOAD.md)
- [SHOW ROUTINE LOAD TASK](./data-manipulation/SHOW_ROUTINE_LOAD_TASK.md)
- [STOP ROUTINE LOAD](./data-manipulation/STOP_ROUTINE_LOAD.md)

### 其他导入

- [ALTER LOAD](./data-manipulation/ALTER_LOAD.md)
- [BROKER LOAD](./data-manipulation/BROKER_LOAD.md)
- [CANCEL LOAD](./data-manipulation/CANCEL_LOAD.md)
- [INSERT](./data-manipulation/INSERT.md)
- [SHOW LOAD](./data-manipulation/SHOW_LOAD.md)
- [SHOW TRANSACTION](./data-manipulation/SHOW_TRANSACTION.md)
- [SPARK LOAD](./data-manipulation/SPARK_LOAD.md)
- [STREAM LOAD](./data-manipulation/STREAM_LOAD.md)

### 导出

- [EXPORT](./data-manipulation/EXPORT.md)
- [CANCEL EXPORT](./data-manipulation/CANCEL_EXPORT.md)
- [SHOW EXPORT](./data-manipulation/SHOW_EXPORT.md)

### ETL 任务

- [DROP TASK](./data-manipulation/DROP_TASK.md)
- [SUBMIT TASK](./data-manipulation/SUBMIT_TASK.md)

## 数据目录 (Catalog)，数据库，资源

### Catalog

- [CREATE EXTERNAL CATALOG](./data-definition/CREATE_EXTERNAL_CATALOG.md)
- [DROP CATALOG](./data-definition/DROP_CATALOG.md)
- [SET CATALOG](./data-definition/SET_CATALOG.md)
- [SHOW CATALOGS](./data-manipulation/SHOW_CATALOGS.md)
- [SHOW CREATE CATALOG](./data-manipulation/SHOW_CREATE_CATALOG.md)

### 数据库

- [ALTER DATABASE](./data-definition/ALTER_DATABASE.md)
- [CREATE DATABASE](./data-definition/CREATE_DATABASE.md)
- [DROP DATABASE](./data-definition/DROP_DATABASE.md)
- [显示创建数据库](./data-manipulation/SHOW_CREATE_DATABASE.md)
- [显示数据](./data-manipulation/SHOW_DATA.md)
- [显示数据库](./data-manipulation/SHOW_DATABASES.md)

### 资源

- [修改资源](./data-definition/ALTER_RESOURCE.md)
- [创建资源](./data-definition/CREATE_RESOURCE.md)
- [删除资源](./data-definition/DROP_RESOURCE.md)
- [显示资源](./data-definition/SHOW_RESOURCES.md)

## 建表，分区

- [修改表](./data-definition/ALTER_TABLE.md)
- [取消修改表](./data-definition/CANCEL_ALTER_TABLE.md)
- [创建表](./data-definition/CREATE_TABLE.md)
- [创建表为选择](./data-definition/CREATE_TABLE_AS_SELECT.md)
- [类似创建表](./data-definition/CREATE_TABLE_LIKE.md)
- [删除表](./data-definition/DROP_TABLE.md)
- [刷新外部表](./data-definition/REFRESH_EXTERNAL_TABLE.md)
- [截断表](./data-definition/TRUNCATE_TABLE.md)
- [删除](./data-manipulation/DELETE.md)
- [显示修改表](./data-manipulation/SHOW_ALTER.md)
- [显示创建表](./data-manipulation/SHOW_CREATE_TABLE.md)
- [显示删除](./data-manipulation/SHOW_DELETE.md)
- [显示动态分区表](./data-manipulation/SHOW_DYNAMIC_PARTITION_TABLES.md)
- [显示分区](./data-manipulation/SHOW_PARTITIONS.md)
- [显示表](./data-manipulation/SHOW_TABLES.md)
- [显示表格](./data-manipulation/SHOW_TABLET.md)
- [更新](./data-manipulation/UPDATE.md)

## 视图，物化视图

### 视图

- [修改视图](./data-definition/ALTER_VIEW.md)
- [创建视图](./data-definition/CREATE_VIEW.md)
- [显示创建视图](./data-manipulation/SHOW_CREATE_VIEW.md)
- [删除视图](./data-definition/DROP_VIEW.md)

### 物化视图

- [修改物化视图](./data-definition/ALTER_MATERIALIZED_VIEW.md)
- [创建物化视图](./data-definition/CREATE_MATERIALIZED_VIEW.md)
- [删除物化视图](./data-definition/DROP_MATERIALIZED_VIEW.md)
- [取消刷新物化视图](./data-manipulation/CANCEL_REFRESH_MATERIALIZED_VIEW.md)
- [刷新物化视图](./data-manipulation/REFRESH_MATERIALIZED_VIEW.md)
- [显示修改物化视图](./data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md)
- [显示创建物化视图](./data-manipulation/SHOW_CREATE_MATERIALIZED_VIEW.md)
- [显示物化视图](./data-manipulation/SHOW_MATERIALIZED_VIEW.md)

## 函数，选择

- [创建函数](./data-definition/CREATE_FUNCTION.md)
- [删除函数](./data-definition/DROP_FUNCTION.md)
- [显示函数](./data-definition/SHOW_FUNCTIONS.md)
- [选择](./data-manipulation/SELECT.md)

## CBO 统计信息

- [分析表](./data-definition/ANALYZE_TABLE.md)
- [创建分析](./data-definition/CREATE_ANALYZE.md)
- [删除分析](./data-definition/DROP_ANALYZE.md)
- [删除统计信息](./data-definition/DROP_STATS.md)
- [终止分析](./data-definition/KILL_ANALYZE.md)
- [显示分析作业](./data-definition/SHOW_ANALYZE_JOB.md)
- [显示分析状态](./data-definition/SHOW_ANALYZE_STATUS.md)
- [显示元数据](./data-definition/SHOW_META.md)

## 备份与恢复

- [备份](./data-definition/BACKUP.md)
- [取消备份](./data-definition/CANCEL_BACKUP.md)
- [取消恢复](./data-definition/CANCEL_RESTORE.md)
- [创建存储库](./data-definition/CREATE_REPOSITORY.md)
- [删除存储库](./data-definition/DROP_REPOSITORY.md)
- [恢复](./data-definition/RECOVER.md)
- [恢复](./data-definition/RESTORE.md)
- [显示备份](./data-manipulation/SHOW_BACKUP.md)
- [显示存储库](./data-manipulation/SHOW_REPOSITORIES.md)
- [显示恢复](./data-manipulation/SHOW_RESTORE.md)
- [显示快照](./data-manipulation/SHOW_SNAPSHOT.md)

## 工具辅助语句

- [DESC](./Utility/DESCRIBE.md)
- [EXPLAIN](./Administration/EXPLAIN.md)
- [USE](./data-definition/USE.md)
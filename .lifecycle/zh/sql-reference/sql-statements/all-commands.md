---
displayed_sidebar: English
---

# 所有声明

本主题列出了 StarRocks 支持的所有 SQL 语句，并按照它们的功能进行了分类。

- [所有声明](#all-statements)
  - [#用户账户管理](#user-account-management)
  - [#集群管理](#cluster-management)
    - [FE、BE、CN、Broker、process](#fe-be-cn-broker-process)
    - [资源组](#resource-group)
    - [存储卷](#storage-volume)
    - [表格, tablet, replica](#table-tablet-replica)
    - [文件, 索引, 变量](#file-index-variable)
    - [SQL 黑名单](#sql-blacklist)
    - [插件](#plugin)
  - [加载、卸载](#loading-unloading)
    - [常规加载](#routine-load)
    - [其他加载](#other-load)
    - [卸载](#unloading)
    - [ETL 任务](#etl-task)
  - [目录、数据库、资源](#catalog-database-resource)
    - [目录](#catalog)
    - [数据库](#database)
    - [#资源](#resource)
  - [创建表格、分区](#create-table-partition)
  - [视图, 物化视图](#view-materialized-view)
    - [视图](#view)
    - [物化视图](#materialized-view)
  - [函数, SELECT](#function-select)
  - [CBO 统计](#cbo-statistics)
  - [备份和还原](#backup-and-restore)
  - [实用工具命令](#utility-commands)

## 用户账户管理

管理用户、角色和权限。

- [修改用户](./account-management/ALTER_USER.md)
- [创建角色](./account-management/CREATE_ROLE.md)
- [创建用户](./account-management/CREATE_USER.md)
- [DROP ROLE](./account-management/DROP_ROLE.md)
- [DROP USER](./account-management/DROP_USER.md)
- [执行为](./account-management/EXECUTE_AS.md)
- [GRANT](./account-management/GRANT.md)
- [REVOKE](./account-management/REVOKE.md)
- [SET DEFAULT ROLE](./account-management/SET_DEFAULT_ROLE.md)
- [设置密码](./account-management/SET_PASSWORD.md)
- [SET PROPERTY](./account-management/SET_PROPERTY.md)
- [SET ROLE](./account-management/SET_ROLE.md)
- [显示认证](./account-management/SHOW_AUTHENTICATION.md)
- [SHOW GRANTS](./account-management/SHOW_GRANTS.md)
- [显示属性](./account-management/SHOW_PROPERTY.md)
- [显示角色](./account-management/SHOW_ROLES.md)
- [显示用户](./account-management/SHOW_USERS.md)

## 集群管理

管理集群，包括 FE、BE、计算节点、Broker、资源组、存储卷、表格、Tablet、副本、文件、索引、变量和插件。

### FE、BE、CN、Broker、进程

- [ADMIN SET CONFIG](./Administration/ADMIN_SET_CONFIG.md)
- [ADMIN SHOW CONFIG](./Administration/ADMIN_SHOW_CONFIG.md)
- [修改系统](./Administration/ALTER_SYSTEM.md)
- [取消停用](./Administration/CANCEL_DECOMMISSION.md)
- [KILL](./Administration/KILL.md)
- [显示后端](./Administration/SHOW_BACKENDS.md)
- [SHOW BROKER](./Administration/SHOW_BROKER.md)
- [显示计算节点](./Administration/SHOW_COMPUTE_NODES.md)
- [显示前端](./Administration/SHOW_FRONTENDS.md)
- [显示进程](./Administration/SHOW_PROC.md)
- [SHOW PROCESSLIST](./Administration/SHOW_PROCESSLIST.md)
- [显示运行的查询](./Administration/SHOW_RUNNING_QUERIES.md)

### 资源组

- [创建资源组](./Administration/CREATE_RESOURCE_GROUP.md)
- [修改资源组](./Administration/ALTER_RESOURCE_GROUP.md)
- [删除资源组](./Administration/DROP_RESOURCE_GROUP.md)
- [显示资源组](./Administration/SHOW_RESOURCE_GROUP.md)
- [显示使用资源组](./Administration/SHOW_USAGE_RESOURCE_GROUPS.md)

### 存储卷

- [修改存储卷](./Administration/ALTER_STORAGE_VOLUME.md)
- [创建存储卷](./Administration/CREATE_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./Administration/DESC_STORAGE_VOLUME.md)
- [删除存储卷](./Administration/DROP_STORAGE_VOLUME.md)
- [设置默认存储卷](./Administration/SET_DEFAULT_STORAGE_VOLUME.md)
- [显示存储卷](./Administration/SHOW_STORAGE_VOLUMES.md)

### 表格、Tablet、副本

- [ADMIN CANCEL REPAIR TABLE](./Administration/ADMIN_CANCEL_REPAIR.md)
- [ADMIN CHECK TABLET](./Administration/ADMIN_CHECK_TABLET.md)
- [ADMIN REPAIR TABLE](./Administration/ADMIN_REPAIR.md)
- [ADMIN SET REPLICA STATUS](./Administration/ADMIN_SET_REPLICA_STATUS.md)
- [ADMIN SHOW REPLICA DISTRIBUTION](./Administration/ADMIN_SHOW_REPLICA_DISTRIBUTION.md)
- [ADMIN SHOW REPLICA STATUS](./Administration/ADMIN_SHOW_REPLICA_STATUS.md)
- [RECOVER](./data-definition/RECOVER.md)
- [显示表状态](./Administration/SHOW_TABLE_STATUS.md)

### 文件、索引、变量

- [创建文件](./Administration/CREATE_FILE.md)
- [创建索引](./data-definition/CREATE_INDEX.md)
- [DROP FILE](./Administration/DROP_FILE.md)
- [删除索引](./data-definition/DROP_INDEX.md)
- [SET (variable)](./Administration/SET.md)
- [显示文件](./Administration/SHOW_FILE.md)
- [显示完整列](./Administration/SHOW_FULL_COLUMNS.md)
- [显示索引](./Administration/SHOW_INDEX.md)
- [显示变量](./Administration/SHOW_VARIABLES.md)

### SQL 黑名单

- [添加SQL黑名单](./Administration/ADD_SQLBLACKLIST.md)
- [显示SQL黑名单](./Administration/SHOW_SQLBLACKLIST.md)
- [DELETE SQLBLACKLIST](./Administration/DELETE_SQLBLACKLIST.md)

### 插件

- [安装插件](./Administration/INSTALL_PLUGIN.md)
- [显示插件](./Administration/SHOW_PLUGINS.md)
- [卸载插件](./Administration/UNINSTALL_PLUGIN.md)

## 加载、卸载

### 常规加载

- [修改例程加载](./data-manipulation/ALTER_ROUTINE_LOAD.md)
- [创建例行加载](./data-manipulation/CREATE_ROUTINE_LOAD.md)
- [PAUSE ROUTINE LOAD](./data-manipulation/PAUSE_ROUTINE_LOAD.md)
- [继续例行加载](./data-manipulation/RESUME_ROUTINE_LOAD.md)
- [显示例行负载](./data-manipulation/SHOW_ROUTINE_LOAD.md)
- [显示例行加载任务](./data-manipulation/SHOW_ROUTINE_LOAD_TASK.md)
- [停止例行加载](./data-manipulation/STOP_ROUTINE_LOAD.md)

### 其他加载

- [修改加载](./data-manipulation/ALTER_LOAD.md)
- [BROKER LOAD](./data-manipulation/BROKER_LOAD.md)
- [取消加载](./data-manipulation/CANCEL_LOAD.md)
- [插入](./data-manipulation/INSERT.md)
- [SHOW LOAD](./data-manipulation/SHOW_LOAD.md)
- [显示交易](./data-manipulation/SHOW_TRANSACTION.md)
- [SPARK LOAD](./data-manipulation/SPARK_LOAD.md)
- [STREAM LOAD](./data-manipulation/STREAM_LOAD.md)

### 卸载

- [./data-manipulation/EXPORT.md](EXPORT)
- [取消导出](./data-manipulation/CANCEL_EXPORT.md)
- [显示导出](./data-manipulation/SHOW_EXPORT.md)

### ETL 任务

- [删除任务](./data-manipulation/DROP_TASK.md)
- [提交任务](./data-manipulation/SUBMIT_TASK.md)

## 目录、数据库、资源

### 目录

- [创建外部目录](./data-definition/CREATE_EXTERNAL_CATALOG.md)
- [删除目录](./data-definition/DROP_CATALOG.md)
- [SET CATALOG](./data-definition/SET_CATALOG.md)
- [显示目录](./data-manipulation/SHOW_CATALOGS.md)
- [显示创建目录](./data-manipulation/SHOW_CREATE_CATALOG.md)

### 数据库

- [修改数据库](./data-definition/ALTER_DATABASE.md)
- [创建数据库](./data-definition/CREATE_DATABASE.md)
- [删除数据库](./data-definition/DROP_DATABASE.md)
- [显示创建数据库](./data-manipulation/SHOW_CREATE_DATABASE.md)
- [显示数据](./data-manipulation/SHOW_DATA.md)
- [显示数据库](./data-manipulation/SHOW_DATABASES.md)

### 资源

- [修改资源](./data-definition/ALTER_RESOURCE.md)
- [创建资源](./data-definition/CREATE_RESOURCE.md)
- [删除资源](./data-definition/DROP_RESOURCE.md)
- [显示资源](./data-definition/SHOW_RESOURCES.md)

## 创建表格、分区

- [修改表](./data-definition/ALTER_TABLE.md)
- [取消修改表](./data-definition/CANCEL_ALTER_TABLE.md)
- [创建表](./data-definition/CREATE_TABLE.md)
- [CREATE TABLE AS SELECT](./data-definition/CREATE_TABLE_AS_SELECT.md)
- [创建表格类似](./data-definition/CREATE_TABLE_LIKE.md)
- [DROP TABLE](./data-definition/DROP_TABLE.md)
- [刷新外部表](./data-definition/REFRESH_EXTERNAL_TABLE.md)
- [TRUNCATE TABLE](./data-definition/TRUNCATE_TABLE.md)
- [删除](./data-manipulation/DELETE.md)
- [SHOW ALTER TABLE](./data-manipulation/SHOW_ALTER.md)
- [显示创建表](./data-manipulation/SHOW_CREATE_TABLE.md)
- [显示删除](./data-manipulation/SHOW_DELETE.md)
- [显示动态分区表](./data-manipulation/SHOW_DYNAMIC_PARTITION_TABLES.md)
- [显示分区](./data-manipulation/SHOW_PARTITIONS.md)
- [显示表格](./data-manipulation/SHOW_TABLES.md)
- [显示平板电脑](./data-manipulation/SHOW_TABLET.md)
- [更新](./data-manipulation/UPDATE.md)

## 视图、物化视图

### 视图

- [修改视图](./data-definition/ALTER_VIEW.md)
- [创建视图](./data-definition/CREATE_VIEW.md)
- [显示创建视图](./data-manipulation/SHOW_CREATE_VIEW.md)
- [DROP VIEW](./data-definition/DROP_VIEW.md)

### 物化视图

- [修改物化视图](./data-definition/ALTER_MATERIALIZED_VIEW.md)
- [创建物化视图](./data-definition/CREATE_MATERIALIZED_VIEW.md)
- [删除物化视图](./data-definition/DROP_MATERIALIZED_VIEW.md)
- [取消刷新物化视图](./data-manipulation/CANCEL_REFRESH_MATERIALIZED_VIEW.md)
- [刷新物化视图](./data-manipulation/REFRESH_MATERIALIZED_VIEW.md)
- [显示修改物化视图](./data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md)
- [显示创建物化视图](./data-manipulation/SHOW_CREATE_MATERIALIZED_VIEW.md)
- [显示物化视图](./data-manipulation/SHOW_MATERIALIZED_VIEW.md)

## 函数、SELECT

- [创建函数](./data-definition/CREATE_FUNCTION.md)
- [删除函数](./data-definition/DROP_FUNCTION.md)
- [显示函数](./data-definition/SHOW_FUNCTIONS.md)
- [选择](./data-manipulation/SELECT.md)

## CBO 统计

- [ANALYZE TABLE](./data-definition/ANALYZE_TABLE.md)
- [创建分析](./data-definition/CREATE_ANALYZE.md)
- [DROP ANALYZE](./data-definition/DROP_ANALYZE.md)
- [DROP STATS](./data-definition/DROP_STATS.md)
- [KILL ANALYZE](./data-definition/KILL_ANALYZE.md)
- [显示分析作业](./data-definition/SHOW_ANALYZE_JOB.md)
- [SHOW ANALYZE STATUS](./data-definition/SHOW_ANALYZE_STATUS.md)
- [显示元数据](./data-definition/SHOW_META.md)

## 备份和还原

- [备份](./data-definition/BACKUP.md)
- [取消备份](./data-definition/CANCEL_BACKUP.md)
- [取消恢复](./data-definition/CANCEL_RESTORE.md)
- [创建存储库](./data-definition/CREATE_REPOSITORY.md)
- [删除仓库](./data-definition/DROP_REPOSITORY.md)
- [RECOVER](./data-definition/RECOVER.md)
- [恢复](./data-definition/RESTORE.md)
- [显示备份](./data-manipulation/SHOW_BACKUP.md)
- [显示存储库](./data-manipulation/SHOW_REPOSITORIES.md)
- [显示恢复](./data-manipulation/SHOW_RESTORE.md)
- [显示快照](./data-manipulation/SHOW_SNAPSHOT.md)

## 实用工具命令

- [DESC](./Utility/DESCRIBE.md)
- [解释](./Administration/EXPLAIN.md)
- [使用](./data-definition/USE.md)

---
displayed_sidebar: English
---

# 所有语句

本主题列出了 StarRocks 支持的所有 SQL 语句，并根据其功能对这些语句进行分类。

- [所有语句](#all-statements)
  - [用户帐户管理](#user-account-management)
  - [集群管理](#cluster-management)
    - [FE、BE、CN、Broker、进程](#fe-be-cn-broker-process)
    - [资源组](#resource-group)
    - [存储卷](#storage-volume)
    - [表、平板、副本](#table-tablet-replica)
    - [文件、索引、变量](#file-index-variable)
    - [SQL 黑名单](#sql-blacklist)
    - [插件](#plugin)
  - [装载、卸载](#loading-unloading)
    - [常规加载](#routine-load)
    - [其他加载](#other-load)
    - [卸载](#unloading)
    - [ETL 任务](#etl-task)
  - [目录、数据库、资源](#catalog-database-resource)
    - [目录](#catalog)
    - [数据库](#database)
    - [资源](#resource)
  - [创建表、分区](#create-table-partition)
  - [视图、物化视图](#view-materialized-view)
    - [视图](#view)
    - [物化视图](#materialized-view)
  - [函数、SELECT](#function-select)
  - [CBO 统计数据](#cbo-statistics)
  - [备份和还原](#backup-and-restore)
  - [实用命令](#utility-commands)

## 用户帐户管理

管理用户、角色和权限。

- [修改用户](./account-management/ALTER_USER.md)
- [创建角色](./account-management/CREATE_ROLE.md)
- [创建用户](./account-management/CREATE_USER.md)
- [删除角色](./account-management/DROP_ROLE.md)
- [删除用户](./account-management/DROP_USER.md)
- [执行 AS](./account-management/EXECUTE_AS.md)
- [授予](./account-management/GRANT.md)
- [撤销](./account-management/REVOKE.md)
- [设置默认角色](./account-management/SET_DEFAULT_ROLE.md)
- [设置密码](./account-management/SET_PASSWORD.md)
- [设置属性](./account-management/SET_PROPERTY.md)
- [设置角色](./account-management/SET_ROLE.md)
- [显示认证信息](./account-management/SHOW_AUTHENTICATION.md)
- [显示授权信息](./account-management/SHOW_GRANTS.md)
- [显示属性](./account-management/SHOW_PROPERTY.md)
- [显示角色](./account-management/SHOW_ROLES.md)
- [显示用户](./account-management/SHOW_USERS.md)

## 集群管理

管理集群，包括 FE、BE、计算节点、Broker、资源组、存储卷、表、平板、副本、文件、索引、变量和插件。

### FE、BE、CN、Broker、进程

- [管理设置配置](./Administration/ADMIN_SET_CONFIG.md)
- [管理显示配置](./Administration/ADMIN_SHOW_CONFIG.md)
- [修改系统](./Administration/ALTER_SYSTEM.md)
- [取消停用](./Administration/CANCEL_DECOMMISSION.md)
- [终止](./Administration/KILL.md)
- [显示后端](./Administration/SHOW_BACKENDS.md)
- [显示 Broker](./Administration/SHOW_BROKER.md)
- [显示计算节点](./Administration/SHOW_COMPUTE_NODES.md)
- [显示前端](./Administration/SHOW_FRONTENDS.md)
- [显示进程](./Administration/SHOW_PROC.md)
- [显示进程列表](./Administration/SHOW_PROCESSLIST.md)
- [显示运行中的查询](./Administration/SHOW_RUNNING_QUERIES.md)

### 资源组

- [创建资源组](./Administration/CREATE_RESOURCE_GROUP.md)
- [修改资源组](./Administration/ALTER_RESOURCE_GROUP.md)
- [删除资源组](./Administration/DROP_RESOURCE_GROUP.md)
- [显示资源组](./Administration/SHOW_RESOURCE_GROUP.md)
- [显示资源组使用情况](./Administration/SHOW_USAGE_RESOURCE_GROUPS.md)

### 存储卷

- [修改存储卷](./Administration/ALTER_STORAGE_VOLUME.md)
- [创建存储卷](./Administration/CREATE_STORAGE_VOLUME.md)
- [描述存储卷](./Administration/DESC_STORAGE_VOLUME.md)
- [删除存储卷](./Administration/DROP_STORAGE_VOLUME.md)
- [设置默认存储卷](./Administration/SET_DEFAULT_STORAGE_VOLUME.md)
- [显示存储卷](./Administration/SHOW_STORAGE_VOLUMES.md)

### 表、平板、副本

- [管理取消修复表](./Administration/ADMIN_CANCEL_REPAIR.md)
- [管理检查平板](./Administration/ADMIN_CHECK_TABLET.md)
- [管理修复表](./Administration/ADMIN_REPAIR.md)
- [管理设置副本状态](./Administration/ADMIN_SET_REPLICA_STATUS.md)
- [管理显示副本分布](./Administration/ADMIN_SHOW_REPLICA_DISTRIBUTION.md)
- [管理显示副本状态](./Administration/ADMIN_SHOW_REPLICA_STATUS.md)
- [恢复](./data-definition/RECOVER.md)
- [显示表状态](./Administration/SHOW_TABLE_STATUS.md)

### 文件、索引、变量

- [创建文件](./Administration/CREATE_FILE.md)
- [创建索引](./data-definition/CREATE_INDEX.md)
- [删除文件](./Administration/DROP_FILE.md)
- [删除索引](./data-definition/DROP_INDEX.md)
- [设置（变量）](./Administration/SET.md)
- [显示文件](./Administration/SHOW_FILE.md)
- [显示完整列](./Administration/SHOW_FULL_COLUMNS.md)
- [显示索引](./Administration/SHOW_INDEX.md)
- [显示变量](./Administration/SHOW_VARIABLES.md)

### SQL 黑名单

- [添加 SQL 黑名单](./Administration/ADD_SQLBLACKLIST.md)
- [显示 SQL 黑名单](./Administration/SHOW_SQLBLACKLIST.md)
- [删除 SQL 黑名单](./Administration/DELETE_SQLBLACKLIST.md)

### 插件

- [安装插件](./Administration/INSTALL_PLUGIN.md)
- [显示插件](./Administration/SHOW_PLUGINS.md)
- [卸载插件](./Administration/UNINSTALL_PLUGIN.md)

## 装载、卸载

### 常规加载

- [修改常规加载](./data-manipulation/ALTER_ROUTINE_LOAD.md)
- [创建常规加载](./data-manipulation/CREATE_ROUTINE_LOAD.md)
- [暂停常规加载](./data-manipulation/PAUSE_ROUTINE_LOAD.md)
- [恢复常规加载](./data-manipulation/RESUME_ROUTINE_LOAD.md)
- [显示常规加载](./data-manipulation/SHOW_ROUTINE_LOAD.md)
- [显示常规加载任务](./data-manipulation/SHOW_ROUTINE_LOAD_TASK.md)
- [停止常规加载](./data-manipulation/STOP_ROUTINE_LOAD.md)

### 其他加载

- [修改加载](./data-manipulation/ALTER_LOAD.md)
- [Broker 加载](./data-manipulation/BROKER_LOAD.md)
- [取消加载](./data-manipulation/CANCEL_LOAD.md)
- [插入](./data-manipulation/INSERT.md)
- [显示加载](./data-manipulation/SHOW_LOAD.md)
- [显示事务](./data-manipulation/SHOW_TRANSACTION.md)
- [Spark 加载](./data-manipulation/SPARK_LOAD.md)
- [流加载](./data-manipulation/STREAM_LOAD.md)

### 卸载

- [导出](./data-manipulation/EXPORT.md)
- [取消导出](./data-manipulation/CANCEL_EXPORT.md)
- [显示导出](./data-manipulation/SHOW_EXPORT.md)

### ETL 任务

- [删除任务](./data-manipulation/DROP_TASK.md)
- [提交任务](./data-manipulation/SUBMIT_TASK.md)

## 目录、数据库、资源

### 目录

- [创建外部目录](./data-definition/CREATE_EXTERNAL_CATALOG.md)
- [删除目录](./data-definition/DROP_CATALOG.md)
- [设置目录](./data-definition/SET_CATALOG.md)
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

## 创建表、分区

- [修改表](./data-definition/ALTER_TABLE.md)
- [取消修改表](./data-definition/CANCEL_ALTER_TABLE.md)
- [创建表](./data-definition/CREATE_TABLE.md)
- [创建表作为选择](./data-definition/CREATE_TABLE_AS_SELECT.md)
- [创建表像](./data-definition/CREATE_TABLE_LIKE.md)
- [删除表](./data-definition/DROP_TABLE.md)
- [刷新外部表](./data-definition/REFRESH_EXTERNAL_TABLE.md)
- [截断表](./data-definition/TRUNCATE_TABLE.md)
- [删除](./data-manipulation/DELETE.md)
- [显示修改表](./data-manipulation/SHOW_ALTER.md)
- [显示创建表](./data-manipulation/SHOW_CREATE_TABLE.md)
- [显示删除](./data-manipulation/SHOW_DELETE.md)
- [显示动态分区表](./data-manipulation/SHOW_DYNAMIC_PARTITION_TABLES.md)
- [显示分区](./data-manipulation/SHOW_PARTITIONS.md)
- [显示表格](./data-manipulation/SHOW_TABLES.md)
- [显示平板](./data-manipulation/SHOW_TABLET.md)
- [更新](./data-manipulation/UPDATE.md)

## 视图、物化视图

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

## 函数、SELECT

- [创建函数](./data-definition/CREATE_FUNCTION.md)
- [删除函数](./data-definition/DROP_FUNCTION.md)
- [显示函数](./data-definition/SHOW_FUNCTIONS.md)
- [SELECT](./data-manipulation/SELECT.md)

## CBO 统计数据

- [分析表](./data-definition/ANALYZE_TABLE.md)
- [创建分析](./data-definition/CREATE_ANALYZE.md)
- [删除分析](./data-definition/DROP_ANALYZE.md)
- [删除统计信息](./data-definition/DROP_STATS.md)
- [终止分析](./data-definition/KILL_ANALYZE.md)
- [显示分析作业](./data-definition/SHOW_ANALYZE_JOB.md)
- [显示分析状态](./data-definition/SHOW_ANALYZE_STATUS.md)
- [显示元数据](./data-definition/SHOW_META.md)

## 备份和还原

- [备份](./data-definition/BACKUP.md)
- [取消备份](./data-definition/CANCEL_BACKUP.md)
- [取消还原](./data-definition/CANCEL_RESTORE.md)
- [创建存储库](./data-definition/CREATE_REPOSITORY.md)
- [删除存储库](./data-definition/DROP_REPOSITORY.md)
- [恢复](./data-definition/RECOVER.md)
- [还原](./data-definition/RESTORE.md)
- [显示备份](./data-manipulation/SHOW_BACKUP.md)
- [显示存储库](./data-manipulation/SHOW_REPOSITORIES.md)
- [显示还原](./data-manipulation/SHOW_RESTORE.md)
- [显示快照](./data-manipulation/SHOW_SNAPSHOT.md)

## 实用命令

- [DESC](./Utility/DESCRIBE.md)
- [解释](./Administration/EXPLAIN.md)
- [使用](./data-definition/USE.md)

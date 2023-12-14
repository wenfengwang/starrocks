---
displayed_sidebar: "Chinese"
---

# StarRocks支持的权限

赋予用户或角色的权限决定了用户或角色可以在某些对象上执行哪些操作。权限可用于实现细粒度访问控制以保障数据安全。

本主题描述了StarRocks在不同对象上提供的权限及其含义。使用[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)可授予和撤销权限。可以授予对象的权限特定于对象类型。例如，表权限与数据库权限不同。

> 注意：本主题中描述的权限仅在v3.0及以上版本可用。v3.0中的权限框架和语法与早期版本不兼容。升级到v3.0后，大部分原始权限仍保留，除了特定操作的权限。有关详细差异，请参阅本主题末尾的[升级说明](#upgrade-notes)。

## 权限列表

本部分描述了可用于不同对象的权限。

### SYSTEM

| 权限                      | 描述                                              |
| ------------------------- | ------------------------------------------------- |
| NODE                      | 操作节点，例如添加、删除或退役节点。为确保集群安全性，不能直接授予用户或角色。`cluster_admin`角色拥有此权限。 |
| GRANT                     | 创建用户或角色，修改用户或角色，或向用户或角色授予权限。不能直接授予用户或角色。`user_admin`角色拥有此权限。 |
| CREATE RESOURCE GROUP     | 创建资源组。                                      |
| CREATE RESOURCE           | 为Spark Load作业或外部表创建资源。                |
| CREATE EXTERNAL CATALOG   | 创建外部目录。                                    |
| PLUGIN                    | 安装或卸载插件。                                  |
| REPOSITORY                | 创建、删除或查看存储库。                          |
| BLACKLIST                 | 创建、删除或显示SQL黑名单。                        |
| FILE                      | 创建、删除或查看文件。                             |
| OPERATE                   | 管理副本、配置项、变量和事务。                    |
| CREATE GLOBAL FUNCTION    | 创建全局UDF。                                     |
| CREATE STORAGE VOLUME     | 为远程存储系统创建存储卷。                       |

### RESOURCE GROUP

| 权限    | 描述                                       |
| ------- | ------------------------------------------ |
| ALTER   | 为资源组添加或删除分类器。                 |
| DROP    | 删除资源组。                               |
| ALL     | 在资源组上具有上述所有权限。               |

### RESOURCE

| 权限    | 描述                    |
| ------- | ----------------------- |
| USAGE   | 使用资源。              |
| ALTER   | 更改资源。              |
| DROP    | 删除资源。              |
| ALL     | 在资源上具有上述所有权限。 |

### USER

| 权限        | 描述                         |
| ----------- | ---------------------------- |
| IMPERSONATE | 允许用户A以用户B的身份执行操作。 |

### GLOBAL FUNCTION (全局UDF)

| 权限    | 描述                            |
| ------- | ------------------------------ |
| USAGE   | 在查询中使用函数。                |
| DROP    | 删除函数。                      |
| ALL     | 在函数上具有上述所有权限。       |

### CATALOG

| 对象                             | 权限                                             | 描述                                     |
| -------------------------------- | ------------------------------------------------ | ---------------------------------------- |
| CATALOG(内部目录)                | USAGE                                            | 使用内部目录（default_catalog）。         |
| CATALOG(内部目录)                | CREATE DATABASE                                  | 在内部目录中创建数据库。                             |
| CATALOG(内部目录)                | ALL                                              | 在内部目录上具有上述所有权限。            |
| CATALOG(外部目录)                | USAGE                                            | 使用外部目录以查看其中的表。               |
| CATALOG(外部目录)                | DROP                                            | 删除外部目录。                                         |
| CATALOG(外部目录)                | ALL                                              | 在外部目录上具有上述所有权限。               |

> 注意：StarRocks内部目录不能被删除。

### DATABASE

| 权限                | 描述                                                     |
| -------------------- | -------------------------------------------------------- |
| ALTER                | 为数据库设置属性，重命名数据库，或为数据库设置配额。      |
| DROP                 | 删除数据库。                                              |
| CREATE TABLE         | 在数据库中创建表。                                       |
| CREATE VIEW          | 创建视图。                                                |
| CREATE FUNCTION      | 创建函数。                                                |
| CREATE MATERIALIZED VIEW | 创建物化视图。                                          |
| ALL                  | 在数据库上具有上述所有权限。                              |

### TABLE

| 权限    | 描述                                                     |
| ------- | --------------------------------------------------------- |
| ALTER   | 修改表或刷新外部表的元数据。                             |
| DROP    | 删除表。                                                  |
| SELECT  | 查询表中的数据。                                           |
| INSERT  | 向表中插入数据。                                           |
| UPDATE  | 更新表中的数据。                                           |
| EXPORT  | 从StarRocks表中导出数据。                                 |
| DELETE  | 根据指定的条件删除表中的数据，或删除表中的所有数据。     |
| ALL     | 在表上具有上述所有权限。                                  |

### VIEW

| 权限    | 描述                            |
| ------- | ------------------------------ |
| SELECT  | 查询视图中的数据。               |
| ALTER   | 修改视图的定义。                 |
| DROP    | 删除逻辑视图。                   |
| ALL     | 在视图上具有上述所有权限。      |

### MATERIALIZED VIEW

| 权限    | 描述                                      |
| ------- | ---------------------------------------- |
| SELECT  | 查询物化视图以加速查询。                 |
| ALTER   | 更改物化视图。                           |
| REFRESH | 刷新物化视图。                           |
| DROP    | 删除物化视图。                           |
| ALL     | 在物化视图上具有上述所有权限。          |

### FUNCTION（数据库级UDF）

| 权限    | 描述                           |
| ------- | ----------------------------- |
| USAGE   | 使用函数。                     |
| DROP    | 删除函数。                     |
| ALL     | 在函数上具有上述所有权限。         |

### STORAGE VOLUME

| 权限    | 描述                                      |
| ------- | ---------------------------------------- |
| ALTER   | 更改存储卷的凭据属性、注释或状态（已启用）。 |
| DROP    | 删除存储卷。                              |
| USAGE   | 描述存储卷并将存储卷设置为默认存储卷。     |
| ALL     | 在存储卷上具有上述所有权限。               |

## 升级说明

从v2.x升级到v3.0期间，由于引入新的权限系统，您的一些操作可能无法执行。以下表格描述了升级前后的变化情况。

| **操作**               | **涉及的命令**           | **升级前**                                                      | **升级后**                                                     |
| ----------------------- | ----------------------- | --------------------------------------------------------------- | --------------------------------------------------------------- |
| 更改表                   | ALTER TABLE，CANCEL ALTER TABLE | 具有表上的`LOAD_PRIV`权限或表所属数据库上的用户可以执行`ALTER TABLE`和`CANCEL ALTER TABLE`操作。 | 执行这两个操作需要具有表上的ALTER权限。                       |
| 刷新外部表               | REFRESH EXTERNAL TABLE | 具有外部表上的`LOAD_PRIV`权限的用户可以刷新外部表。              | 执行此操作需要具有外部表上的ALTER权限。                         |
| 备份和恢复               | BACKUP，RESTORE         | 具有数据库上的`LOAD_PRIV`权限的用户可以备份和恢复数据库或数据库中的任何表。 | 管理员必须在升级后再次向用户授予备份和恢复权限。                |
| 删除后恢复               | RECOVER                 | 具有数据库和表上的`ALTER_PRIV`、`CREATE_PRIV`和`DROP_PRIV`权限的用户可以恢复数据库和表。 | 恢复数据库需要具有default_catalog上的CREATE DATABASE权限。同时需要在数据库上具有CREATE TABLE权限和在表上具有DROP权限。 |
| 创建和更改用户           | CREATE USER，ALTER USER | 具有数据库上的`GRANT_PRIV`权限的用户可以创建和更改用户。           | 创建和更改用户需要`user_admin`角色。                             |
| 授予和撤销权限           | GRANT，REVOKE           | 具有对象上的`GRANT_PRIV`权限的用户可以向其他用户或角色授予权限。 | 升级后，您可以继续向其他用户或角色授予您已在该对象上具有的权限。<br />在新的权限系统中：<ul><li>您必须具有`user_admin`角色才能向其他用户或角色授予权限。</li><li>如果您的GRANT语句包括`WITH GRANT OPTION`，则可以将语句中涉及的权限授予其他用户或角色。 </li></ul>|

在v2.x中，StarRocks没有完全实现基于角色的访问控制（RBAC）。当您向用户分配角色时，StarRocks会直接向用户授予角色的所有权限，而不是角色本身。因此，用户实际上不拥有该角色。

```
In v3.0, StarRocks renovates its privilege system. After an upgrade to v3.0, your original roles are retained but there is still no ownership between users and roles. If you want to use the new RBAC system, perform the GRANT operation to assign roles and privileges.
```

```
在v3.0中，StarRocks更新了其权限系统。在升级到v3.0后，您的原始角色会被保留，但用户和角色之间仍然没有所有权。如果您想使用新的RBAC系统，请执行GRANT操作来分配角色和权限。
```
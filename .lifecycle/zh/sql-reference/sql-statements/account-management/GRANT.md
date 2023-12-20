---
displayed_sidebar: English
---

# 授权

从 '../../../assets/commonMarkdown/userPrivilegeCase.md' 导入 UserPrivilegeCase

## 描述

向用户或角色授予特定对象的一个或多个权限。

将角色赋予用户或其他角色。

有关可授予的权限的更多信息，请参见[权限项](../../../administration/privilege_item.md)。

执行 GRANT 操作后，可以运行 [SHOW GRANTS](./SHOW_GRANTS.md) 查看详细的权限信息，或运行 [REVOKE](REVOKE.md) 撤销权限或角色。

在执行 **GRANT** 操作之前，请确保相关用户或角色已被创建。更多信息，请参见 [CREATE USER](./CREATE_USER.md) 和 [CREATE ROLE](./CREATE_ROLE.md)。

> **注意**
> 只有拥有 `user_admin` 角色的用户才能向其他用户和角色授予任何权限。
其他用户只能使用 WITH GRANT OPTION 关键字向其他用户和角色授予权限。

## 语法

### 向角色或用户授予权限

#### 系统

```SQL
GRANT
    { CREATE RESOURCE GROUP | CREATE RESOURCE | CREATE EXTERNAL CATALOG | REPOSITORY | BLACKLIST | FILE | OPERATE | CREATE STORAGE VOLUME } 
    ON SYSTEM
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### 资源组

```SQL
GRANT
    { ALTER | DROP | ALL [PRIVILEGES] } 
    ON { RESOURCE GROUP <resource_group_name> [, <resource_group_name >,...] ｜ ALL RESOURCE GROUPS} 
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### 资源

```SQL
GRANT
    { USAGE | ALTER | DROP | ALL [PRIVILEGES] } 
    ON { RESOURCE <resource_name> [, < resource_name >,...] ｜ ALL RESOURCES} 
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### 全局 UDF

```SQL
GRANT
    { USAGE | DROP | ALL [PRIVILEGES]} 
    ON { GLOBAL FUNCTION <function_name>(input_data_type) [, < function_name >(input_data_type),...]    
       | ALL GLOBAL FUNCTIONS }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

示例：授予用户 kevin 全局函数 a(string) 的使用权限；

#### 内部目录

```SQL
GRANT
    { USAGE | CREATE DATABASE | ALL [PRIVILEGES]} 
    ON CATALOG default_catalog
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### 外部目录

```SQL
GRANT
   { USAGE | DROP | ALL [PRIVILEGES] } 
   ON { CATALOG <catalog_name> [, <catalog_name>,...] | ALL CATALOGS}
   TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### 数据库

```SQL
GRANT
    { ALTER | DROP | CREATE TABLE | CREATE VIEW | CREATE FUNCTION | CREATE MATERIALIZED VIEW | ALL [PRIVILEGES] } 
    ON { DATABASE <database_name> [, <database_name>,...] | ALL DATABASES }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* 在执行此命令之前，必须先运行 SET CATALOG。
* 对于外部目录中的数据库，你只能在 Hive（自 v3.1 起）和 Iceberg 数据库（自 v3.2 起）上授予 CREATE TABLE 权限。

#### 表

```SQL
GRANT
    { ALTER | DROP | SELECT | INSERT | EXPORT | UPDATE | DELETE | ALL [PRIVILEGES]} 
    ON { TABLE <table_name> [, < table_name >,...]
       | ALL TABLES} IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* 在执行此命令之前，必须先运行 SET CATALOG。
* 你也可以使用 <db_name>.<table_name> 来指代一个表。
* 你可以在内部和外部目录的所有表上授予 SELECT 权限，以读取这些表的数据。对于 Hive 和 Iceberg 目录中的表，你可以授予 INSERT 权限以将数据写入这些表（Iceberg 自 v3.1 起，Hive 自 v3.2 起支持）。

  ```SQL
  GRANT <priv> ON TABLE <db_name>.<table_name> TO {ROLE <role_name> | USER <user_name>}
  ```

#### 视图

```SQL
GRANT  
    { ALTER | DROP | SELECT | ALL [PRIVILEGES]} 
    ON { VIEW <view_name> [, < view_name >,...]
       ｜ ALL VIEWS} IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* 在执行此命令之前，必须先运行 SET CATALOG。
* 你还可以使用 <db_name>.<view_name> 来指代一个视图。
* 对于外部目录中的表，你只能在 Hive 视图上授予 SELECT 权限（自 v3.1 起）。

  ```SQL
  GRANT <priv> ON VIEW <db_name>.<view_name> TO {ROLE <role_name> | USER <user_name>}
  ```

#### 物化视图

```SQL
GRANT
    { SELECT | ALTER | REFRESH | DROP | ALL [PRIVILEGES]} 
    ON { MATERIALIZED VIEW <mv_name> [, < mv_name >,...]
       ｜ ALL MATERIALIZED VIEWS} IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* 在执行此命令之前，必须先运行 SET CATALOG。
* 你还可以使用 <db_name>.<mv_name> 来指代一个物化视图。

  ```SQL
  GRANT <priv> ON MATERIALIZED VIEW <db_name>.<mv_name> TO {ROLE <role_name> | USER <user_name>}
  ```

#### 函数

```SQL
GRANT
    { USAGE | DROP | ALL [PRIVILEGES]} 
    ON { FUNCTION <function_name>(input_data_type) [, < function_name >(input_data_type),...]
       ｜ ALL FUNCTIONS} IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* 在执行此命令之前，必须先运行 SET CATALOG。
* 你也可以使用 <db_name>.<function_name> 来指代一个函数。

  ```SQL
  GRANT <priv> ON FUNCTION <db_name>.<function_name> TO {ROLE <role_name> | USER <user_name>}
  ```

#### 用户

```SQL
GRANT IMPERSONATE
ON USER <user_identity>
TO USER <user_identity_1> [ WITH GRANT OPTION ]
```

#### 存储卷

```SQL
GRANT  
    { USAGE | ALTER | DROP | ALL [PRIVILEGES] } 
    ON { STORAGE VOLUME < name > [, < name >,...] ｜ ALL STORAGE VOLUMES} 
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

### 向角色或用户授予角色

```SQL
GRANT <role_name> [,<role_name>, ...] TO ROLE <role_name>
GRANT <role_name> [,<role_name>, ...] TO USER <user_identity>
```

## 示例

示例 1：授予用户 Jack 权限，以读取所有数据库中所有表的数据。

```SQL
GRANT SELECT ON *.* TO 'jack'@'%';
```

示例 2：授予角色 my_role 权限，以将数据加载进数据库 db1 的所有表中。

```SQL
GRANT INSERT ON db1.* TO ROLE 'my_role';
```

示例 3：授予用户 Jack 权限，在数据库 db1 的表 tbl1 上读取、更新和加载数据。

```SQL
GRANT SELECT,ALTER,INSERT ON db1.tbl1 TO 'jack'@'192.8.%';
```

示例 4：授予用户 Jack 权限，使用所有资源。

```SQL
GRANT USAGE ON RESOURCE * TO 'jack'@'%';
```

示例 5：授予用户 Jack 权限，使用资源 spark_resource。

```SQL
GRANT USAGE ON RESOURCE 'spark_resource' TO 'jack'@'%';
```

示例 6：向角色 my_role 授予使用资源 spark_resource 的权限。

```SQL
GRANT USAGE ON RESOURCE 'spark_resource' TO ROLE 'my_role';
```

示例 7：授予用户 Jack 权限，从表 sr_member 读取数据，并允许用户 Jack 将此权限授予其他用户或角色（通过指定 WITH GRANT OPTION）。

```SQL
GRANT SELECT ON TABLE sr_member TO USER jack@'172.10.1.10' WITH GRANT OPTION;
```

示例 8：将系统定义的角色 db_admin、user_admin 和 cluster_admin 授予用户 user_platform。

```SQL
GRANT db_admin, user_admin, cluster_admin TO USER user_platform;
```

示例 9：允许用户 Jack 以用户 Rose 的身份执行操作。

```SQL
GRANT IMPERSONATE ON 'rose'@'%' TO 'jack'@'%';
```

## 最佳实践

### 根据场景定制角色

<UserPrivilegeCase />


有关多服务访问控制的最佳实践，请参见[Multi-service access control](../../../administration/User_privilege.md#multi-service-access-control)。

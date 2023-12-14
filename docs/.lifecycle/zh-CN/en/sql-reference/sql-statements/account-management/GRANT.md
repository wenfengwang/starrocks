---
displayed_sidebar: "Chinese"
---

# GRANT

import UserPrivilegeCase from '../../../assets/commonMarkdown/userPrivilegeCase.md'

## 描述

向用户或角色授予特定对象的一个或多个权限。

向用户或其他角色授予角色。

有关可授予的权限的更多信息，请参见[权限项](../../../administration/privilege_item.md)。

执行GRANT操作后，您可以运行[SHOW GRANTS](./SHOW_GRANTS.md)查看详细的权限信息或运行[REVOKE](REVOKE.md)来撤销权限或角色。

在执行GRANT操作之前，请确保相关用户或角色已被创建。有关更多信息，请参见[CREATE USER](./CREATE_USER.md)和[CREATE ROLE](./CREATE_ROLE.md)。

> **注意**
>
> 只有具有`user_admin`角色的用户才能向其他用户和角色授予任何权限。
> 其他用户只能使用WITH GRANT OPTION关键字向其他用户和角色授予权限。

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

#### 全局UDF

```SQL
GRANT
    { USAGE | DROP | ALL [PRIVILEGES]} 
    ON { GLOBAL FUNCTION <function_name>(input_data_type) [, < function_name >(input_data_type),...]    
       | ALL GLOBAL FUNCTIONS }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

示例：`GRANT usage ON GLOBAL FUNCTION a(string) to kevin;`

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

* 在运行此命令之前，您必须首先运行SET CATALOG。
* 对于外部目录中的数据库，您只能在Hive（自v3.1起）和Iceberg数据库（自v3.2起）上授予CREATE TABLE权限。

#### 表

```SQL
GRANT
    { ALTER | DROP | SELECT | INSERT | EXPORT | UPDATE | DELETE | ALL [PRIVILEGES]} 
    ON { TABLE <table_name> [, < table_name >,...]
       | ALL TABLES} IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* 在运行此命令之前，您必须首先运行SET CATALOG。
* 您还可以使用`<db_name>.<table_name>`表示表。
* 您可以向所有内部和外部目录中的表授予SELECT权限以从这些表中读取数据。对于Hive和Iceberg目录中的表，您可以授予INSERT权限以将数据写入此类表（自v3.1支持Iceberg，自v3.2支持Hive）。

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

* 在运行此命令之前，您必须首先运行SET CATALOG。
* 您还可以使用`<db_name>.<view_name>`表示视图。
* 对于外部目录中的表，您只能授予Hive表视图的SELECT权限（自v3.1起）。

  ```SQL
  GRANT <priv> ON VIEW <db_name>.<view_name> TO {ROLE <role_name> | USER <user_name>}
  ```

#### 材料化视图

```SQL
GRANT
    { SELECT | ALTER | REFRESH | DROP | ALL [PRIVILEGES]} 
    ON { MATERIALIZED VIEW <mv_name> [, < mv_name >,...]
       ｜ ALL MATERIALIZED VIEWS} IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* 在运行此命令之前，您必须首先运行SET CATALOG。
* 您还可以使用`<db_name>.<mv_name>`表示材料化视图。

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

* 在运行此命令之前，您必须首先运行SET CATALOG。
* 您还可以使用`<db_name>.<function_name>`表示函数。

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

### 向用户或角色授予角色

```SQL
GRANT <role_name> [,<role_name>, ...] TO ROLE <role_name>
GRANT <role_name> [,<role_name>, ...] TO USER <user_identity>
```

## 示例

示例1：向用户`jack`授予从所有数据库中的所有表中读取数据的权限。

```SQL
GRANT SELECT ON *.* TO 'jack'@'%';
```

示例2：向角色`my_role`授予将数据加载到数据库`db1`的所有表中的权限。

```SQL
GRANT INSERT ON db1.* TO ROLE 'my_role';
```

示例3：向用户`jack`授予读取、更新和加载数据库`db1`的表`tbl1`中的数据的权限。

```SQL
GRANT SELECT,ALTER,INSERT ON db1.tbl1 TO 'jack'@'192.8.%';
```

示例4：向用户`jack`授予使用所有资源的权限。

```SQL
GRANT USAGE ON RESOURCE * TO 'jack'@'%';
```

示例5：向用户`jack`授予使用资源`spark_resource`的权限。

```SQL
GRANT USAGE ON RESOURCE 'spark_resource' TO 'jack'@'%';
```

示例6：向角色`my_role`授予使用资源`spark_resource`的权限。

```SQL
GRANT USAGE ON RESOURCE 'spark_resource' TO ROLE 'my_role';
```

```SQL
示例7：授予用户`jack`从表`sr_member`读取数据的权限，并允许用户`jack`通过指定WITH GRANT OPTION将此权限授予其他用户或角色。

```SQL
GRANT SELECT ON TABLE sr_member TO USER jack@'172.10.1.10' WITH GRANT OPTION;
```

示例8：向用户`user_platform`授予系统定义的角色`db_admin`、`user_admin`和`cluster_admin`。

```SQL
GRANT db_admin, user_admin, cluster_admin TO USER user_platform;
```

示例9：允许用户`jack`以用户`rose`的身份执行操作。

```SQL
GRANT IMPERSONATE ON 'rose'@'%' TO 'jack'@'%';
```

## 最佳实践

### 根据场景定制角色

<UserPrivilegeCase />

有关多服务访问控制的最佳实践，请参见[多服务访问控制](../../../administration/User_privilege.md#multi-service-access-control)。
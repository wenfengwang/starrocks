---
displayed_sidebar: English
---

# 撤销

## 描述

撤销用户或角色的特定权限或角色。有关 StarRocks 支持的权限，请参见 [StarRocks 支持的权限](../../../administration/privilege_item.md)。

> 注意：只有 `user_admin` 角色才能执行此操作。

## 语法

### 撤销权限

可以撤销的权限是特定于对象的。以下部分根据对象描述了语法。

#### 系统

```SQL
REVOKE
    { CREATE RESOURCE GROUP | CREATE RESOURCE | CREATE EXTERNAL CATALOG | REPOSITORY | BLACKLIST | FILE | OPERATE } 
    ON SYSTEM
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

#### 资源组

```SQL
REVOKE
    { ALTER | DROP | ALL [PRIVILEGES] } 
    ON { RESOURCE GROUP <resourcegroup_name> [, <resourcegroup_name>,...] ｜ ALL RESOURCE GROUPS} 
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

#### 资源

```SQL
REVOKE
    { USAGE | ALTER | DROP | ALL [PRIVILEGES] } 
    ON { RESOURCE <resource_name> [, <resource_name>,...] ｜ ALL RESOURCES} 
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

#### 用户

```SQL
REVOKE IMPERSONATE ON USER <user_identity> FROM USER <user_identity>;
```

#### 全局 UDF

```SQL
REVOKE
    { USAGE | DROP | ALL [PRIVILEGES]} 
    ON { GLOBAL FUNCTION <function_name> [, <function_name>,...]    
       | ALL GLOBAL FUNCTIONS }
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

#### 内部目录

```SQL
REVOKE 
    { USAGE | CREATE DATABASE | ALL [PRIVILEGES]} 
    ON CATALOG default_catalog
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

#### 外部目录

```SQL
REVOKE  
   { USAGE | DROP | ALL [PRIVILEGES] } 
   ON { CATALOG <catalog_name> [, <catalog_name>,...] | ALL CATALOGS}
   FROM { ROLE | USER} {<role_name>|<user_identity>}
```

#### 数据库

```SQL
REVOKE 
    { ALTER | DROP | CREATE TABLE | CREATE VIEW | CREATE FUNCTION | CREATE MATERIALIZED VIEW | ALL [PRIVILEGES] } 
    ON {{ DATABASE <database_name> [, <database_name>,...]} | ALL DATABASES }
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

* 在运行此命令之前，必须先运行 SET CATALOG。

#### 表

```SQL
REVOKE  
    { ALTER | DROP | SELECT | INSERT | EXPORT | UPDATE | DELETE | ALL [PRIVILEGES]} 
    ON { TABLE <table_name> [, < table_name >,...]
       | ALL TABLES} IN 
           { { DATABASE <database_name> [, <database_name>,...]} | ALL DATABASES }
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

* 在运行此命令之前，必须先运行 SET CATALOG。
* 还可以使用 db.tbl 来表示表。

  ```SQL
  REVOKE <priv> ON TABLE db.tbl FROM {ROLE <role_name> | USER <user_identity>}
  ```

#### 视图

```SQL
REVOKE  
    { ALTER | DROP | SELECT | ALL [PRIVILEGES]} 
    ON { VIEW <view_name> [, < view_name >,...]
       ｜ ALL VIEWS} IN 
           { { DATABASE <database_name> [, <database_name>,...]}  | ALL DATABASES }
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

* 在运行此命令之前，必须先运行 SET CATALOG。
* 您还可以使用 db.view 来表示视图。

  ```SQL
  REVOKE <priv> ON VIEW db.view FROM {ROLE <role_name> | USER <user_identity>}
  ```

#### 物化视图

```SQL
REVOKE
    { SELECT | ALTER | REFRESH | DROP | ALL [PRIVILEGES]} 
    ON { MATERIALIZED VIEW <mv_name> [, < mv_name >,...]
       ｜ ALL MATERIALIZED VIEWS} IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL [DATABASES] }
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

* 在运行此命令之前，必须先运行 SET CATALOG。
* 您还可以使用 db.mv 来表示 mv。

  ```SQL
  REVOKE <priv> ON MATERIALIZED VIEW db.mv FROM {ROLE <role_name> | USER <user_identity>}
  ```

#### 功能

```SQL
REVOKE
    { USAGE | DROP | ALL [PRIVILEGES]} 
    ON { FUNCTION <function_name> [, < function_name >,...]
       ｜ ALL FUNCTIONS} IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

* 在运行此命令之前，必须先运行 SET CATALOG。
* 还可以使用 db.function 来表示函数。

  ```SQL
  REVOKE <priv> ON FUNCTION db.function FROM {ROLE <role_name> | USER <user_identity>}
  ```

#### 存储卷

```SQL
REVOKE
    CREATE STORAGE VOLUME 
    ON SYSTEM
    FROM { ROLE | USER} {<role_name>|<user_identity>}

REVOKE
    { USAGE | ALTER | DROP | ALL [PRIVILEGES] } 
    ON { STORAGE VOLUME < name > [, < name >,...] ｜ ALL STORAGE VOLUME} 
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

### 撤销角色

```SQL
REVOKE <role_name> [,<role_name>, ...] FROM ROLE <role_name>
REVOKE <role_name> [,<role_name>, ...] FROM USER <user_identity>
```

## 参数

| **参数**      | **描述**                                 |
| ------------------ | ----------------------------------------------- |
| role_name          | 角色名称。                                  |
| user_identity      | 用户标识，例如，'jack'@'192.%'。 |
| resourcegroup_name | 资源组名称                         |
| resource_name      | 资源名称。                              |
| function_name      | 函数名称。                              |
| catalog_name       | 外部目录的名称。               |
| database_name      | 数据库名称。                              |
| table_name         | 表名。                                 |
| view_name          | 视图名称。                                  |
| mv_name            | 实例化视图的名称。              |

## 例子

### 撤销权限

从用户 `jack` 撤销对表 `sr_member` 的 SELECT 权限：

```SQL
REVOKE SELECT ON TABLE sr_member FROM USER 'jack'@'192.%'
```

从角色 `test_role` 撤销对资源 `spark_resource` 的 USAGE 权限：

```SQL
REVOKE USAGE ON RESOURCE 'spark_resource' FROM ROLE 'test_role';
```

### 撤销角色

从用户 `jack` 撤销角色 `example_role`：

```SQL
REVOKE example_role FROM 'jack'@'%';
```

从角色 `test_role` 撤销角色 `example_role`：

```SQL
REVOKE example_role FROM ROLE 'test_role';
```

## 引用

[授权](GRANT.md)

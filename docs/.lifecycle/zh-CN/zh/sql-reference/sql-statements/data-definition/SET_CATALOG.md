---
displayed_sidebar: "Chinese"
---

# 设定目录

## 功能

切换到指定的目录。此命令自3.0版本起支持。

> **注意**
>
> 如果是新部署的3.1集群，用户需要有目标目录的使用权限才能执行此操作。如果是从较低版本升级过来的集群，则无需重新授权。您可以使用[GRANT](../account-management/GRANT.md)命令进行授权操作。

## 语法

```SQL
SET CATALOG <catalog_name>
```

## 参数

`catalog_name`：当前会话中生效的目录，支持内部目录和外部目录。如果指定的目录不存在，将引发异常。

## 示例

通过以下命令，在当前会话中将生效的目录切换为Hive目录`hive_metastore`：

```SQL
SET CATALOG hive_metastore;
```

通过以下命令，在当前会话中将生效的目录切换为内部目录`default_catalog`：

```SQL
SET CATALOG default_catalog;
```
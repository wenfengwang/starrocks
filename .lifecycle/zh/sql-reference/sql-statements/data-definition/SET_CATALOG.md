---
displayed_sidebar: English
---

# 设置目录

在当前会话中切换到指定的目录。

该命令从 v3.0 版本开始支持。

> **注意**
> 对于新部署的 **StarRocks v3.1** 集群，如果您想运行 **SET CATALOG** 命令以切换到目标外部目录，您必须拥有该目录的 **USAGE** 权限。您可以使用 [**GRANT**](../account-management/GRANT.md) 命令授予所需的权限。对于从早期版本升级至 v3.1 的集群，您可以依据继承的权限运行 **SET CATALOG** 命令。

## 语法

```SQL
SET CATALOG <catalog_name>
```

## 参数

catalog_name：在当前会话中要使用的目录名称。您可以切换到一个内部或外部目录。如果您指定的目录不存在，系统将抛出一个异常。

## 示例

执行以下命令，在当前会话中切换到名为 hive_metastore 的 Hive 目录：

```SQL
SET CATALOG hive_metastore;
```

执行以下命令，在当前会话中切换到内部目录 default_catalog：

```SQL
SET CATALOG default_catalog;
```

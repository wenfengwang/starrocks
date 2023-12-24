---
displayed_sidebar: English
---

# 设置目录

在当前会话中切换到指定的目录。

此命令从v3.0版本开始支持。

> **注意**
>
> 对于新部署的StarRocks v3.1集群，如果要运行SET CATALOG切换到该目录，您必须具有目标外部目录的USAGE权限。您可以使用[GRANT](../account-management/GRANT.md)来授予所需的权限。对于从较早版本升级的v3.1集群，您可以使用继承的权限运行SET CATALOG。

## 语法

```SQL
SET CATALOG <catalog_name>
```

## 参数

`catalog_name`：要在当前会话中使用的目录的名称。您可以切换到内部或外部目录。如果您指定的目录不存在，将引发异常。

## 例子

运行以下命令，切换到当前会话中名为`hive_metastore`的Hive目录：

```SQL
SET CATALOG hive_metastore;
```

运行以下命令，切换到当前会话中的内部目录`default_catalog`：

```SQL
SET CATALOG default_catalog;
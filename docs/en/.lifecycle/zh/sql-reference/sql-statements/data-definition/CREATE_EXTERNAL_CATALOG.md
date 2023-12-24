---
displayed_sidebar: English
---

# 创建外部目录

## 描述

创建外部目录。您可以使用外部目录查询外部数据源中的数据，而无需将数据加载到 StarRocks 或创建外部表。目前，您可以创建以下类型的外部目录：

- [Hive目录](../../../data_source/catalog/hive_catalog.md)：用于从Apache Hive™查询数据。
- [Iceberg 目录](../../../data_source/catalog/iceberg_catalog.md)：用于查询 Apache Iceberg 中的数据。
- [Hudi目录](../../../data_source/catalog/hudi_catalog.md)：用于从 Apache Hudi 查询数据。
- [Delta Lake 目录](../../../data_source/catalog/deltalake_catalog.md)：用于从 Delta Lake 查询数据。
- [JDBC目录](../../../data_source/catalog/jdbc_catalog.md)：用于查询JDBC兼容数据源中的数据。

> **注意**
>
> - 在 v3.0 及更高版本中，此语句需要系统级 CREATE EXTERNAL CATALOG 权限。
> - 在创建外部目录之前，请配置您的 StarRocks 集群，以满足外部数据源的数据存储系统（如 Amazon S3）、元数据服务（如 Hive 元存储）和认证服务（如 Kerberos）的需求。有关详细信息，请参阅每个[外部目录主题](../../../data_source/catalog/catalog_overview.md)中的“开始之前”部分。

## 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

## 参数

| **参数** | **必填** | **描述**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| catalog_name  | 是          | 外部目录的名称。命名约定如下：<ul><li>名称可以包含字母、数字（0-9）、下划线（_）。必须以字母开头。</li><li>名称区分大小写，长度不能超过 1023 个字符。</li></ul> |
| comment       | 否           | 外部目录的说明。 |
| properties    | 是          | 外部目录的属性。根据外部目录的类型配置属性。更多信息，请参见[Hive目录](../../../data_source/catalog/hive_catalog.md)、[Iceberg目录](../../../data_source/catalog/iceberg_catalog.md)、[Hudi目录](../../../data_source/catalog/hudi_catalog.md)、[Delta Lake目录](../../../data_source/catalog/deltalake_catalog.md)和[JDBC目录](../../../data_source/catalog/jdbc_catalog.md)。 |

## 例子

示例 1：创建名为 `hive_metastore_catalog` 的 Hive 目录。对应的 Hive 群集使用 Hive 元存储作为其元数据服务。

```SQL
CREATE EXTERNAL CATALOG hive_metastore_catalog
PROPERTIES(
   "type"="hive", 
   "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

示例 2：创建名为 `hive_glue_catalog` 的 Hive 目录。相应的 Hive 集群使用 AWS Glue 作为其元数据服务。

```SQL
CREATE EXTERNAL CATALOG hive_glue_catalog
PROPERTIES(
    "type"="hive", 
    "hive.metastore.type"="glue",
    "aws.hive.metastore.glue.aws-access-key"="xxxxxx",
    "aws.hive.metastore.glue.aws-secret-key"="xxxxxxxxxxxx",
    "aws.hive.metastore.glue.endpoint"="https://glue.x-x-x.amazonaws.com"
);
```

示例 3：创建名为 `iceberg_metastore_catalog` 的 Iceberg 目录。对应的 Iceberg 集群使用 Hive 元存储作为其元数据服务。

```SQL
CREATE EXTERNAL CATALOG iceberg_metastore_catalog
PROPERTIES(
    "type"="iceberg",
    "iceberg.catalog.type"="hive",
    "iceberg.catalog.hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

示例 4：创建名为 `iceberg_glue_catalog` 的 Iceberg 目录。相应的 Iceberg 集群使用 AWS Glue 作为其元数据服务。

```SQL
CREATE EXTERNAL CATALOG iceberg_glue_catalog
PROPERTIES(
    "type"="iceberg", 
    "iceberg.catalog.type"="glue",
    "aws.hive.metastore.glue.aws-access-key"="xxxxx",
    "aws.hive.metastore.glue.aws-secret-key"="xxxxxxxxxxxx",
    "aws.hive.metastore.glue.endpoint"="https://glue.x-x-x.amazonaws.com"
);
```

示例 5：创建名为 `hudi_metastore_catalog` 的 Hudi 目录。对应的 Hudi 集群使用 Hive 元存储作为其元数据服务。

```SQL
CREATE EXTERNAL CATALOG hudi_metastore_catalog
PROPERTIES(
    "type"="hudi",
    "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

示例 6：创建名为 `hudi_glue_catalog` 的 Hudi 目录。相应的 Hudi 集群使用 AWS Glue 作为其元数据服务。

```SQL
CREATE EXTERNAL CATALOG hudi_glue_catalog
PROPERTIES(
    "type"="hudi", 
    "hive.metastore.type"="glue",
    "aws.hive.metastore.glue.aws-access-key"="xxxxxx",
    "aws.hive.metastore.glue.aws-secret-key"="xxxxxxxxxxxx",
    "aws.hive.metastore.glue.endpoint"="https://glue.x-x-x.amazonaws.com"
);
```

示例 7：创建名为 `delta_metastore_catalog` 的 Delta Lake 目录。相应的 Delta Lake 服务使用 Hive 元存储作为其元数据服务。

```SQL
CREATE EXTERNAL CATALOG delta_metastore_catalog
PROPERTIES(
    "type"="deltalake",
    "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

示例 8：创建名为 `delta_glue_catalog` 的 Delta Lake 目录。相应的 Delta Lake 服务使用 AWS Glue 作为其元数据服务。

```SQL
CREATE EXTERNAL CATALOG delta_glue_catalog
PROPERTIES(
    "type"="deltalake", 
    "hive.metastore.type"="glue",
    "aws.hive.metastore.glue.aws-access-key"="xxxxxx",
    "aws.hive.metastore.glue.aws-secret-key"="xxxxxxxxxxxx",
    "aws.hive.metastore.glue.endpoint"="https://glue.x-x-x.amazonaws.com"
);
```

## 引用

- 要查看 StarRocks 集群中的所有目录，请参见 [SHOW CATALOGS](../data-manipulation/SHOW_CATALOGS.md)。
- 要查看外部目录的创建语句，请参阅 [SHOW CREATE CATALOG。](../data-manipulation/SHOW_CREATE_CATALOG.md)
- 要从 StarRocks 集群中删除外部目录，请参见 [DROP CATALOG](../data-definition/DROP_CATALOG.md)。

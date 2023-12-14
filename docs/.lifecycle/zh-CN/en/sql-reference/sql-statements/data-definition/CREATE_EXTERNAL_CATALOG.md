---
displayed_sidebar: "Chinese"
---

# 创建外部目录

## 描述

创建一个外部目录。您可以使用外部目录来查询外部数据源中的数据，而无需将数据加载到 StarRocks 中或创建外部表。目前，您可以创建以下类型的外部目录：

- [Hive 目录](../../../data_source/catalog/hive_catalog.md)：用于从 Apache Hive™ 查询数据。
- [Iceberg 目录](../../../data_source/catalog/iceberg_catalog.md)：用于从 Apache Iceberg 查询数据。
- [Hudi 目录](../../../data_source/catalog/hudi_catalog.md)：用于从 Apache Hudi 查询数据。
- [Delta Lake 目录](../../../data_source/catalog/deltalake_catalog.md)：用于查询来自 Delta Lake 的数据。
- [JDBC 目录](../../../data_source/catalog/jdbc_catalog.md)：用于查询来自兼容 JDBC 的数据源的数据。

> **注意**
>
> - 从 v3.0 开始，此语句需要 SYSTEM 级别的 CREATE EXTERNAL CATALOG 权限。
> - 在创建外部目录之前，配置您的 StarRocks 集群，以满足外部数据源的数据存储系统（例如 Amazon S3）、元数据服务（例如 Hive 元数据存储）和认证服务（例如 Kerberos）的要求。有关更多信息，请参阅每个[外部目录主题](../../../data_source/catalog/catalog_overview.md)中的“开始之前”部分。

## 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

## 参数

| **参数**       | **必需**    | **描述**                                                     |
| ------------- | ------------ | ------------------------------------------------------------ |
| catalog_name  | 是           | 外部目录的名称。命名约定如下：<ul><li>名称可以包含字母、数字（0-9）和下划线（_）。它必须以字母开头。</li><li>名称区分大小写，长度不能超过 1023 个字符。</li></ul> |
| comment       | 否           | 外部目录的描述。 |
| PROPERTIES    | 是           | 外部目录的属性。根据外部目录的类型配置属性。有关更多信息，请参见[Hive 目录](../../../data_source/catalog/hive_catalog.md)、[Iceberg 目录](../../../data_source/catalog/iceberg_catalog.md)、[Hudi 目录](../../../data_source/catalog/hudi_catalog.md)、[Delta Lake 目录](../../../data_source/catalog/deltalake_catalog.md)和[JDBC 目录](../../../data_source/catalog/jdbc_catalog.md)。 |

## 示例

示例 1：创建一个名为 `hive_metastore_catalog` 的 Hive 目录。相应的 Hive 集群使用 Hive 元数据服务。

```SQL
CREATE EXTERNAL CATALOG hive_metastore_catalog
PROPERTIES(
   "type"="hive", 
   "hive.metastore.uris"="thrift://x.x.x.x:9083"
);
```

示例 2：创建一个名为 `hive_glue_catalog` 的 Hive 目录。相应的 Hive 集群使用 AWS Glue 作为其元数据服务。

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

示例 3：创建一个名为 `iceberg_metastore_catalog` 的 Iceberg 目录。相应的 Iceberg 集群使用 Hive 元数据服务。

```SQL
CREATE EXTERNAL CATALOG iceberg_metastore_catalog
PROPERTIES(
    "type"="iceberg",
    "iceberg.catalog.type"="hive",
    "iceberg.catalog.hive.metastore.uris"="thrift://x.x.x.x:9083"
);
```

示例 4：创建一个名为 `iceberg_glue_catalog` 的 Iceberg 目录。相应的 Iceberg 集群使用 AWS Glue 作为其元数据服务。

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

示例 5：创建一个名为 `hudi_metastore_catalog` 的 Hudi 目录。相应的 Hudi 集群使用 Hive 元数据服务。

```SQL
CREATE EXTERNAL CATALOG hudi_metastore_catalog
PROPERTIES(
    "type"="hudi",
    "hive.metastore.uris"="thrift://x.x.x.x:9083"
);
```

示例 6：创建一个名为 `hudi_glue_catalog` 的 Hudi 目录。相应的 Hudi 集群使用 AWS Glue 作为其元数据服务。

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

示例 7：创建一个名为 `delta_metastore_catalog` 的 Delta Lake 目录。相应的 Delta Lake 服务使用 Hive 元数据服务。

```SQL
CREATE EXTERNAL CATALOG delta_metastore_catalog
PROPERTIES(
    "type"="deltalake",
    "hive.metastore.uris"="thrift://x.x.x.x:9083"
);
```

示例 8：创建一个名为 `delta_glue_catalog` 的 Delta Lake 目录。相应的 Delta Lake 服务使用 AWS Glue 作为其元数据服务。

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

## 参考

- 要查看您的 StarRocks 集群中的所有目录，请参阅 [SHOW CATALOGS](../data-manipulation/SHOW_CATALOGS.md)。
- 要查看外部目录的创建语句，请参阅 [SHOW CREATE CATALOG](../data-manipulation/SHOW_CREATE_CATALOG.md)。
- 要从 StarRocks 集群中删除外部目录，请参阅 [DROP CATALOG](../data-definition/DROP_CATALOG.md)。
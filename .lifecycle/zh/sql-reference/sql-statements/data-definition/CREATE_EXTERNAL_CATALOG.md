---
displayed_sidebar: English
---

# 创建外部目录

## 描述

此操作用于创建一个外部目录。通过外部目录，您可以查询外部数据源中的数据，而无需将数据导入StarRocks或创建外部表。目前，您可以创建以下类型的外部目录：

- [Hive目录](../../../data_source/catalog/hive_catalog.md)：用来查询Apache Hive™中的数据。
- [Iceberg目录](../../../data_source/catalog/iceberg_catalog.md)：用来查询Apache Iceberg中的数据。
- [Hudi目录](../../../data_source/catalog/hudi_catalog.md)：用来查询Apache Hudi中的数据。
- [Delta Lake目录](../../../data_source/catalog/deltalake_catalog.md)：用来查询Delta Lake中的数据。
- [JDBC目录](../../../data_source/catalog/jdbc_catalog.md)：用来查询兼容JDBC的数据源中的数据。

> **注意**
- 在v3.0及以后的版本中，执行此语句需要SYSTEM级别的CREATE EXTERNAL CATALOG权限。
- 在创建外部目录前，配置您的StarRocks集群以满足数据存储系统（如Amazon S3）、元数据服务（如Hive Metastore）和认证服务（如Kerberos）的要求。有关更多信息，请参阅每个[外部目录主题](../../../data_source/catalog/catalog_overview.md)中的"开始前"部分。

## 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

## 参数

|参数|必填|说明|
|---|---|---|
|catalog_name|是|外部目录的名称。命名规则如下：名称可以包含字母、数字（0-9）和下划线（_）。它必须以字母开头。名称区分大小写，长度不能超过 1023 个字符。|
|注释|否|外部目录的描述。|
|属性|是|外部目录的属性。根据外部目录的类型配置属性。有关详细信息，请参阅 Hive 目录、Iceberg 目录、Hudi 目录、Delta Lake 目录和 JDBC 目录。|

## 示例

示例1：创建一个名为hive_metastore_catalog的Hive目录。对应的Hive集群使用Hive Metastore作为其元数据服务。

```SQL
CREATE EXTERNAL CATALOG hive_metastore_catalog
PROPERTIES(
   "type"="hive", 
   "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

示例2：创建一个名为hive_glue_catalog的Hive目录。对应的Hive集群使用AWS Glue作为其元数据服务。

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

示例3：创建一个名为iceberg_metastore_catalog的Iceberg目录。对应的Iceberg集群使用Hive Metastore作为其元数据服务。

```SQL
CREATE EXTERNAL CATALOG iceberg_metastore_catalog
PROPERTIES(
    "type"="iceberg",
    "iceberg.catalog.type"="hive",
    "iceberg.catalog.hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

示例4：创建一个名为iceberg_glue_catalog的Iceberg目录。对应的Iceberg集群使用AWS Glue作为其元数据服务。

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

示例5：创建一个名为hudi_metastore_catalog的Hudi目录。对应的Hudi集群使用Hive Metastore作为其元数据服务。

```SQL
CREATE EXTERNAL CATALOG hudi_metastore_catalog
PROPERTIES(
    "type"="hudi",
    "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

示例6：创建一个名为hudi_glue_catalog的Hudi目录。对应的Hudi集群使用AWS Glue作为其元数据服务。

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

示例7：创建一个名为delta_metastore_catalog的Delta Lake目录。对应的Delta Lake服务使用Hive Metastore作为其元数据服务。

```SQL
CREATE EXTERNAL CATALOG delta_metastore_catalog
PROPERTIES(
    "type"="deltalake",
    "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

示例8：创建一个名为delta_glue_catalog的Delta Lake目录。对应的Delta Lake服务使用AWS Glue作为其元数据服务。

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

## 参考资料

- 要查看StarRocks集群中所有目录，请参见[SHOW CATALOGS](../data-manipulation/SHOW_CATALOGS.md)。
- 要查看外部目录的创建语句，请参见[SHOW CREATE CATALOG](../data-manipulation/SHOW_CREATE_CATALOG.md)。
- 要从StarRocks集群中删除外部目录，请参见[DROP CATALOG](../data-definition/DROP_CATALOG.md)。

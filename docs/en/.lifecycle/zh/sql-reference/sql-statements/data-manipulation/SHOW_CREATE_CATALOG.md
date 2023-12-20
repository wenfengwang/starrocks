---
displayed_sidebar: English
---

# SHOW CREATE CATALOG

## 描述

查询外部目录的创建语句，例如[Hive Catalog](../../../data_source/catalog/hive_catalog.md)、[Iceberg Catalog](../../../data_source/catalog/iceberg_catalog.md)、[Hudi Catalog](../../../data_source/catalog/hudi_catalog.md)、[Delta Lake Catalog](../../../data_source/catalog/deltalake_catalog.md)和[JDBC Catalog](../../../data_source/catalog/jdbc_catalog.md)。请注意，返回结果中的认证相关信息将被匿名处理。

该命令从 v3.0 版本开始支持。

## 语法

```SQL
SHOW CREATE CATALOG <catalog_name>;
```

## 参数

|**参数**|**必填**|**说明**|
|---|---|---|
|catalog_name|是|您想要查看创建语句的目录名称。|

## 返回结果

```Plain
+------------+-----------------+
| Catalog    | Create Catalog  |
+------------+-----------------+
```

|**字段**|**描述**|
|---|---|
|Catalog|目录的名称。|
|Create Catalog|执行以创建目录的语句。|

## 示例

以下示例查询名为 `hive_catalog_hms` 的 Hive Catalog 的创建语句：

```SQL
SHOW CREATE CATALOG hive_catalog_hms;
```

返回结果如下：

```SQL
CREATE EXTERNAL CATALOG `hive_catalog_hms`
PROPERTIES ("aws.s3.access_key"  =  "AK******M4",
"hive.metastore.type"  =  "glue",
"aws.s3.secret_key"  =  "iV******iD",
"aws.glue.secret_key"  =  "iV******iD",
"aws.s3.use_instance_profile"  =  "false",
"aws.s3.region"  =  "us-west-1",
"aws.glue.region"  =  "us-west-1",
"type"  =  "hive",
"aws.glue.access_key"  =  "AK******M4"
)
```
---
displayed_sidebar: English
---

# 显示创建目录

## 描述

查询外部目录的创建语句，例如Hive、Iceberg、Hudi、Delta Lake或JDBC目录。详情请参见[Hive目录](../../../data_source/catalog/hive_catalog.md)、[Iceberg目录](../../../data_source/catalog/iceberg_catalog.md)、[Hudi目录](../../../data_source/catalog/hudi_catalog.md)、[Delta Lake目录](../../../data_source/catalog/deltalake_catalog.md)以及[JDBC目录](../../../data_source/catalog/jdbc_catalog.md)。请注意，返回结果中与认证相关的信息将进行匿名处理。

该命令从v3.0版本开始支持。

## 语法

```SQL
SHOW CREATE CATALOG <catalog_name>;
```

## 参数

|参数|必填|说明|
|---|---|---|
|catalog_name|Yes|您要查看其创建语句的目录的名称。|

## 返回结果

```Plain
+------------+-----------------+
| Catalog    | Create Catalog  |
+------------+-----------------+
```

|字段|描述|
|---|---|
|目录|目录的名称。|
|创建目录|为创建目录而执行的语句。|

## 示例

以下示例查询名为hive_catalog_hms的Hive目录的创建语句：

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

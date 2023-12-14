---
displayed_sidebar: "Chinese"
---

# 显示创建目录

## 功能

显示特定外部目录（如 Hive 目录、Iceberg 目录、Hudi 目录、Delta Lake 目录或 JDBC 目录）的创建语句。参见[Hive 目录](../../../data_source/catalog/hive_catalog.md)、[Iceberg 目录](../../../data_source/catalog/iceberg_catalog.md)、[Hudi 目录](../../../data_source/catalog/hudi_catalog.md)、[Delta Lake 目录](../../../data_source/catalog/deltalake_catalog.md)和[JDBC 目录](../../../data_source/catalog/jdbc_catalog.md)。其中认证相关的密钥信息会进行脱敏展示，无法查看。

该命令自 3.0 版本起支持。

## 语法

```SQL
SHOW CREATE CATALOG <catalog_name>;
```

## 参数说明

| **参数**       | **是否必选** | **说明**             |
| -------------- | ------------ | -------------------- |
| catalog_name   | 是           | 待查看的目录的名称。 |

## 返回结果说明

```Plain
+------------+-----------------+
| 目录       | 创建目录         |
+------------+-----------------+
```

| **字段**         | **说明**             |
| -------------- | -------------------- |
| 目录             | 目录的名称。       |
| 创建目录         | 目录的创建语句。   |

## 示例

以一个名为`hive_catalog_glue`的 Hive 目录为例，查询该目录的创建语句：

```SQL
SHOW CREATE CATALOG hive_catalog_glue;
```

返回如下信息：

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
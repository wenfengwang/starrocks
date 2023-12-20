---
displayed_sidebar: English
---

# 修改资源属性

## 描述

您可以使用 ALTER RESOURCE 语句来修改资源的属性。

## 语法

```SQL
ALTER RESOURCE 'resource_name' SET PROPERTIES ("key"="value", ...)
```

## 参数

- resource_name：需要修改的资源名称。

- PROPERTIES ("key"="value", ...)：资源的属性。您可以根据不同的资源类型来修改各种属性。目前，StarRocks 支持修改以下资源类型的 Hive metastore URI。
  - Apache Iceberg 资源支持修改以下属性：
    - `iceberg.catalog-impl`：[自定义目录](../../../data_source/External_table.md)的完整类名。
    - iceberg.catalog.hive.metastore.uris：Hive metastore 的 URI。
  - Apache Hive™ 资源和 Apache Hudi 资源支持修改 hive.metastore.uris，表示 Hive metastore 的 URI。

## 使用说明

在引用资源创建外部表之后，如果您修改了该资源的 Hive metastore URI，那么外部表将变得不可用。如果您仍然希望使用外部表来查询数据，请确保新的 metastore 包含有一个与原始 metastore 中的表名称和架构相同的表。

## 示例

修改 Hive 资源 hive0 的 Hive metastore URI。

```SQL
ALTER RESOURCE 'hive0' SET PROPERTIES ("hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083")
```

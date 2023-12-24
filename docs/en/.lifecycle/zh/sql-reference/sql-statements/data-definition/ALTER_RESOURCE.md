---
displayed_sidebar: English
---

# 修改资源

## 描述

您可以使用 ALTER RESOURCE 语句来修改资源的属性。

## 语法

```SQL
ALTER RESOURCE 'resource_name' SET PROPERTIES ("key"="value", ...)
```

## 参数

- `resource_name`：要修改的资源的名称。

- `PROPERTIES ("key"="value", ...)`：资源的属性。您可以根据资源类型修改不同的属性。目前，StarRocks 支持修改以下资源的 Hive 元存储的 URI。
  - Apache Iceberg 资源支持修改以下属性：
    - `iceberg.catalog-impl`：自定义目录的[完全限定类名](../../../data_source/External_table.md)。
    - `iceberg.catalog.hive.metastore.uris`：Hive 元存储的 URI。
  - Apache Hive 资源和 Apache Hudi 资源支持修改 `hive.metastore.uris`，表示 Hive™ 元存储的 URI。

## 使用说明

在引用资源创建外部表后，如果修改该资源的 Hive 元存储的 URI，则外部表将变为不可用。如果仍然希望使用外部表来查询数据，请确保新的元存储中包含一个与原始元存储中名称和架构相同的表。

## 例子

修改 Hive 资源 `hive0` 的 Hive 元存储的 URI。

```SQL
ALTER RESOURCE 'hive0' SET PROPERTIES ("hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083")
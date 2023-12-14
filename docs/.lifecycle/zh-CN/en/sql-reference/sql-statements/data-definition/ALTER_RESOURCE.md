---
displayed_sidebar: "Chinese"
---

# 修改资源

## 描述

您可以使用ALTER RESOURCE语句修改资源的属性。

## 语法

```SQL
ALTER RESOURCE 'resource_name' SET PROPERTIES ("key"="value", ...)
```

## 参数

- `resource_name`：要修改的资源的名称。

- `PROPERTIES ("key"="value", ...)`：资源的属性。您可以基于资源类型修改不同的属性。目前，StarRocks支持修改以下资源的Hive元数据存储的URI。
  - Apache Iceberg资源支持修改以下属性：
    - `iceberg.catalog-impl`：[自定义目录](../../../data_source/External_table.md)的完全限定类名。
    - `iceberg.catalog.hive.metastore.uris`：Hive元数据存储的URI。
  - Apache Hive™资源和Apache Hudi资源支持修改`hive.metastore.uris`，表示Hive元数据存储的URI。

## 用法注意事项

在引用资源创建外部表后，如果修改此资源的Hive元数据存储的URI，则外部表将不可用。如果仍要使用外部表来查询数据，请确保新的元数据存储包含一个与原元数据存储中名称和架构相同的表。

## 示例

修改Hive资源`hive0`的Hive元数据存储的URI。

```SQL
ALTER RESOURCE 'hive0' SET PROPERTIES ("hive.metastore.uris" = "thrift://10.10.44.91:9083")
```
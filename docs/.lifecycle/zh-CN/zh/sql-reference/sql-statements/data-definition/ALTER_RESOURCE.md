---
displayed_sidebar: "中文"
---

# 修改资源

## 功能

修改资源的属性。只有 StarRocks 2.3 及更高版本支持修改资源属性。

## 语法

```SQL
ALTER RESOURCE 'resource_name' SET PROPERTIES ("key"="value", ...)
```

## 参数说明

- `resource_name`：要修改的资源名称。
- `PROPERTIES ("key"="value", ...)`：资源的属性。不同类型的资源支持修改的属性不同，目前支持修改以下资源的 Hive metastore 地址。
  - Apache Iceberg 资源支持修改以下属性：
    - `iceberg.catalog-impl`：[自定义目录](../../../data_source/External_table.md#步骤一创建-iceberg-资源) 的完全限定类名。
    - `iceberg.catalog.hive.metastore.uris`：Hive metastore 的地址。
  - Apache Hive™ 和 Apache Hudi 资源支持修改 `hive.metastore.uris`，也就是 Hive metastore 的地址。

## 注意事项

当一个资源已经被用来创建了外部表，修改了该资源的 Hive metastore 地址将会导致该外部表不可用。如果仍然希望使用该外部表来查询数据，需要确保新的 Hive metastore 中存在与原来 Hive metastore 中名称和表结构相同的数据表。

## 示例

修改 Apache Hive™ 资源 `hive0` 的 Hive metastore 地址。

```SQL
ALTER RESOURCE 'hive0' SET PROPERTIES ("hive.metastore.uris" = "thrift://10.10.44.91:9083")
```
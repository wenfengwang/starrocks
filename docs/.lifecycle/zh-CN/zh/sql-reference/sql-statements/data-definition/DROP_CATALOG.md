```yaml
---
displayed_sidebar: "Chinese"
---

# 删除目录

## 功能

删除指定的外部目录。请注意，目前不支持删除内部目录。一个 StarRocks 集群中只有一个默认的内部目录，名为 `default_catalog`。

## 语法

```SQL
DROP CATALOG catalog_name
```

## 参数说明

`catalog_name`：外部目录的名称，这是一个必选参数。

## 示例

创建名为`hive1`的 Hive 目录。

```SQL
CREATE EXTERNAL CATALOG hive1
PROPERTIES(
  "type"="hive", 
  "hive.metastore.uris"="thrift://x.x.x.x:9083"
);
```

删除`hive1`。

```SQL
DROP CATALOG hive1;
```
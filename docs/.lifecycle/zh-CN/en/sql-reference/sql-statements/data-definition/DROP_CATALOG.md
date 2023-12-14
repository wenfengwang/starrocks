```yaml
---
displayed_sidebar: "English"
---

# 删除目录

## 描述

删除外部目录。内部目录不能被删除。StarRocks集群只有一个名为 `default_catalog` 的内部目录。

## 语法

```SQL
DROP CATALOG <catalog_name>
```

## 参数

`catalog_name`: 外部目录的名称。

## 例子

创建一个名为 `hive1` 的Hive目录。

```SQL
CREATE EXTERNAL CATALOG hive1
PROPERTIES(
  "type"="hive", 
  "hive.metastore.uris"="thrift://x.x.x.x:9083"
);
```

删除Hive目录。

```SQL
DROP CATALOG hive1;
```
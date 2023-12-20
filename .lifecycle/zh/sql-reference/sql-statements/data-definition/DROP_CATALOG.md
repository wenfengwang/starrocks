---
displayed_sidebar: English
---

# 删除目录

## 描述

删除一个外部目录。无法删除内部目录。StarRocks 集群只有一个名为 default_catalog 的内部目录。

## 语法

```SQL
DROP CATALOG <catalog_name>
```

## 参数

catalog_name：外部目录的名称。

## 示例

创建一个名为 hive1 的 Hive 目录。

```SQL
CREATE EXTERNAL CATALOG hive1
PROPERTIES(
  "type"="hive", 
  "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

删除该 Hive 目录。

```SQL
DROP CATALOG hive1;
```

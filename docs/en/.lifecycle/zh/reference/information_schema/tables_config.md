---
displayed_sidebar: English
---

# tables_config

`tables_config` 提供有关表配置的信息。

`tables_config` 中提供了以下字段：

|**字段**|**描述**|
|---|---|
|TABLE_SCHEMA|存储表的数据库名称。|
|TABLE_NAME|表名称。|
|TABLE_ENGINE|表的引擎类型。|
|TABLE_MODEL|表的数据模型。有效值：`DUP_KEYS`、`AGG_KEYS`、`UNQ_KEYS` 或 `PRI_KEYS`。|
|PRIMARY_KEY|主键表或唯一键表的主键。如果表不是主键表或唯一键表，返回空字符串。|
|PARTITION_KEY|表的分区键。|
|DISTRIBUTE_KEY|表的分桶键。|
|DISTRIBUTE_TYPE|表的数据分布方法。|
|DISTRUBTE_BUCKET|表中的桶数量。|
|SORT_KEY|表的排序键。|
|PROPERTIES|表的属性。|
|TABLE_ID|表的 ID。|
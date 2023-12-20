---
displayed_sidebar: English
---

# 表配置信息

tables_config 提供了有关表配置的详细信息。

在 tables_config 中，您可以找到以下字段：

|字段|描述|
|---|---|
|TABLE_SCHEMA|存储表的数据库的名称。|
|TABLE_NAME|表的名称。|
|TABLE_ENGINE|表的引擎类型。|
|TABLE_MODEL|表的数据模型。有效值：DUP_KEYS、AGG_KEYS、UNQ_KEYS 或 PRI_KEYS。|
|PRIMARY_KEY|主键表或唯一键表的主键。如果表不是主键表或唯一键表，则返回空字符串。|
|PARTITION_KEY|表的分区列。|
|DISTRIBUTE_KEY|对表的列进行分桶。|
|DISTRIBUTE_TYPE|表的数据分布方式。|
|DISTRUBTE_BUCKET|表中的桶数。|
|SORT_KEY|表的排序键。|
|属性|表的属性。|
|TABLE_ID|表的 ID。|

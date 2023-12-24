---
displayed_sidebar: English
---

# tables_config

`tables_config` 提供关于表格配置的信息。

`tables_config` 中提供了以下字段：

| **字段**        | **描述**                                              |
| ---------------- | ------------------------------------------------------------ |
| TABLE_SCHEMA     | 存储表格的数据库名称。                  |
| TABLE_NAME       | 表格的名称。                                           |
| TABLE_ENGINE     | 表格的引擎类型。                                    |
| TABLE_MODEL      | 表格的数据模型。有效值： `DUP_KEYS`、 `AGG_KEYS`、`UNQ_KEYS` 或 `PRI_KEYS`。 |
| PRIMARY_KEY      | 主键表或唯一键表的主键。如果该表不是主键表或唯一键表，则返回空字符串。 |
| PARTITION_KEY    | 表格的分区列。                       |
| DISTRIBUTE_KEY   | 表格的分桶列。                          |
| DISTRIBUTE_TYPE  | 表格的数据分布方法。                   |
| DISTRUBTE_BUCKET | 表格中的分桶数。                              |
| SORT_KEY         | 表格的排序键。                                      |
| PROPERTIES       | 表格的属性。                                     |
| TABLE_ID         | 表格的 ID。                                             |

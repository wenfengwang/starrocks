---
displayed_sidebar: "Chinese"
---

# 表格

`tables` 提供关于表格的信息。

`tables` 包含以下字段：

| **字段**        | **描述**                                                     |
| --------------- | ------------------------------------------------------------ |
| TABLE_CATALOG   | 表所属的目录名称。                                      |
| TABLE_SCHEMA    | 表所属的数据库名称。                                         |
| TABLE_NAME      | 表格名称。                                                       |
| TABLE_TYPE      | 表格类型。有效值：`基本表` 和 `视图`。                  |
| ENGINE          | 表格的引擎类型。有效值：`StarRocks`、`MySQL`、`MEMORY` 和空字符串。 |
| VERSION         | 该字段暂不可用。                                             |
| ROW_FORMAT      | 该字段暂不可用。                                             |
| TABLE_ROWS      | 表格的行数。                                                   |
| AVG_ROW_LENGTH  | 表格的平均行长度（大小），等于 `DATA_LENGTH`/`TABLE_ROWS`。单位：字节。 |
| DATA_LENGTH     | 数据长度（大小）。单位：字节。                              |
| MAX_DATA_LENGTH | 该字段暂不可用。                                             |
| INDEX_LENGTH    | 该字段暂不可用。                                             |
| DATA_FREE       | 该字段暂不可用。                                             |
| AUTO_INCREMENT  | 该字段暂不可用。                                             |
| CREATE_TIME     | 创建表格的时间。                                               |
| UPDATE_TIME     | 最后一次更新表格的时间。                                       |
| CHECK_TIME      | 最后一次对表格进行一致性检查的时间。                           |
| TABLE_COLLATION | 表格的默认 Collation。                                         |
| CHECKSUM        | 该字段暂不可用。                                             |
| CREATE_OPTIONS  | 该字段暂不可用。                                             |
| TABLE_COMMENT   | 表格的注释。                                               |
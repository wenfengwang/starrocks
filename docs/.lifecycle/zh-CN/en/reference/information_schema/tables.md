---
displayed_sidebar: "Chinese"
---

# 表

`tables`提供了有关表的信息。

`tables`中提供了以下字段：

| **字段**       | **描述**                                                      |
| --------------- | ------------------------------------------------------------   |
| TABLE_CATALOG   | 存储表的目录名称。                                             |
| TABLE_SCHEMA    | 存储表的数据库名称。                                           |
| TABLE_NAME      | 表格名称。                                                     |
| TABLE_TYPE      | 表格类型。有效值：`基本表`或`视图`。                           |
| ENGINE          | 表格的引擎类型。有效值：`StarRocks`，`MySQL`，`MEMORY`或空字符串。|
| VERSION         | 适用于StarRocks中不可用的功能。                                 |
| ROW_FORMAT      | 适用于StarRocks中不可用的功能。                                 |
| TABLE_ROWS      | 表格的行数。                                                   |
| AVG_ROW_LENGTH  | 表格的平均行长度（大小）。等同于`DATA_LENGTH`/`TABLE_ROWS`。单位：字节。|
| DATA_LENGTH     | 表格的数据长度（大小）。单位：字节。                            |
| MAX_DATA_LENGTH | 适用于StarRocks中不可用的功能。                                 |
| INDEX_LENGTH    | 适用于StarRocks中不可用的功能。                                 |
| DATA_FREE       | 适用于StarRocks中不可用的功能。                                 |
| AUTO_INCREMENT  | 适用于StarRocks中不可用的功能。                                 |
| CREATE_TIME     | 表格创建时间。                                                 |
| UPDATE_TIME     | 表格上次更新时间。                                             |
| CHECK_TIME      | 对表格执行一致性检查的最后时间。                               |
| TABLE_COLLATION | 表格的默认校对规则。                                           |
| CHECKSUM        | 适用于StarRocks中不可用的功能。                                 |
| CREATE_OPTIONS  | 适用于StarRocks中不可用的功能。                                 |
| TABLE_COMMENT   | 对表格的注释。                                                 |
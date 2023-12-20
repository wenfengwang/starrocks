---
displayed_sidebar: English
---

# 表格

`tables` 提供有关表的信息。

`tables` 中提供了以下字段：

|**字段**|**描述**|
|---|---|
|TABLE_CATALOG|存储表的目录名称。|
|TABLE_SCHEMA|存储表的数据库名称。|
|TABLE_NAME|表的名称。|
|TABLE_TYPE|表的类型。有效值：`BASE TABLE` 或 `VIEW`。|
|ENGINE|表的引擎类型。有效值：`StarRocks`、`MySQL`、`MEMORY` 或空字符串。|
|VERSION|适用于 StarRocks 中不可用的特性。|
|ROW_FORMAT|适用于 StarRocks 中不可用的特性。|
|TABLE_ROWS|表的行数。|
|AVG_ROW_LENGTH|表的平均行长度（大小）。相当于 `DATA_LENGTH`/`TABLE_ROWS`。单位：字节。|
|DATA_LENGTH|表的数据长度（大小）。单位：字节。|
|MAX_DATA_LENGTH|适用于 StarRocks 中不可用的特性。|
|INDEX_LENGTH|适用于 StarRocks 中不可用的特性。|
|DATA_FREE|适用于 StarRocks 中不可用的特性。|
|AUTO_INCREMENT|适用于 StarRocks 中不可用的特性。|
|CREATE_TIME|表创建的时间。|
|UPDATE_TIME|表最后更新的时间。|
|CHECK_TIME|表最后进行一致性检查的时间。|
|TABLE_COLLATION|表的默认字符集排序规则。|
|CHECKSUM|适用于 StarRocks 中不可用的特性。|
|CREATE_OPTIONS|适用于 StarRocks 中不可用的特性。|
|TABLE_COMMENT|表的注释。|
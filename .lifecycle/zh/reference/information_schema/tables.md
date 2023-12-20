---
displayed_sidebar: English
---

# 表格

Tables 提供关于表的信息。

表中提供以下字段：

|字段|描述|
|---|---|
|TABLE_CATALOG|存储表的目录的名称。|
|TABLE_SCHEMA|存储表的数据库的名称。|
|TABLE_NAME|表的名称。|
|TABLE_TYPE|表的类型。有效值：基表或视图。|
|ENGINE|表的发动机类型。有效值：StarRocks、“MySQL、MEMORY` 或空字符串。|
|版本|适用于 StarRocks 中不可用的功能。|
|ROW_FORMAT|适用于 StarRocks 中不可用的功能。|
|TABLE_ROWS|表的行数。|
|AVG_ROW_LENGTH|表的平均行长度（大小）。它相当于 DATA_LENGTH/TABLE_ROWS。单位：字节。|
|DATA_LENGTH|表的数据长度（大小）。单位：字节。|
|MAX_DATA_LENGTH|适用于 StarRocks 中不可用的功能。|
|INDEX_LENGTH|适用于 StarRocks 中不可用的功能。|
|DATA_FREE|适用于 StarRocks 中不可用的功能。|
|AUTO_INCRMENT|适用于 StarRocks 中不可用的功能。|
|CREATE_TIME|创建表的时间。|
|UPDATE_TIME|上次更新表的时间。|
|CHECK_TIME|上次对表执行一致性检查的时间。|
|TABLE_COLLATION|表的默认排序规则。|
|CHECKSUM|适用于 StarRocks 中不可用的功能。|
|CREATE_OPTIONS|适用于 StarRocks 中不可用的功能。|
|TABLE_COMMENT|对表进行评论。|

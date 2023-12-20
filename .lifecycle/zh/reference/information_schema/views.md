---
displayed_sidebar: English
---

# 视图

视图提供了关于所有用户定义的视图的信息。

以下是视图中提供的字段：

|字段|描述|
|---|---|
|TABLE_CATALOG|视图所属目录的名称。该值始终是默认值。|
|TABLE_SCHEMA|视图所属数据库的名称。|
|TABLE_NAME|视图的名称。|
|VIEW_DEFINITION|提供视图定义的 SELECT 语句。|
|CHECK_OPTION|CHECK_OPTION 属性的值。该值为 NONE、CASCADE 或 LOCAL 之一。|
|IS_UPDATABLE|视图是否可更新。如果 UPDATE 和 DELETE（以及类似的操作）对于视图来说是合法的，则该标志设置为 YES (true)。否则，该标志被设置为NO（假）。如果视图不可更新，则 UPDATE、DELETE 和 INSERT 等语句是非法的并且会被拒绝。|
|DEFINER|创建视图的用户的用户。|
|SECURITY_TYPE|视图 SQL SECURITY 特征。该值为 DEFINER 或 INVOKER 之一。|
|字符集客户端|
|COLLATION_CONNECTION|

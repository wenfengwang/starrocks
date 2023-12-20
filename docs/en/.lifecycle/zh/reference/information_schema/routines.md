---
displayed_sidebar: English
---

# routines

`routines` 包含所有存储程序（stored procedures）和存储函数（stored functions）。

`routine` 中提供以下字段：

|**字段**|**描述**|
|---|---|
|SPECIFIC_NAME|例程的具体名称。|
|ROUTINE_CATALOG|例程所属的目录名称。此值始终为 `def`。|
|ROUTINE_SCHEMA|例程所属数据库的名称。|
|ROUTINE_NAME|例程的名称。|
|ROUTINE_TYPE|`PROCEDURE` 表示存储过程，`FUNCTION` 表示存储函数。|
|DTD_IDENTIFIER|如果例程是存储函数，则为返回值数据类型。若为存储过程，则此值为空。|
|ROUTINE_BODY|例程定义所使用的语言。此值始终为 `SQL`。|
|ROUTINE_DEFINITION|例程执行的 SQL 语句文本。|
|EXTERNAL_NAME|此值始终为 `NULL`。|
|EXTERNAL_LANGUAGE|存储例程的外部语言。|
|PARAMETER_STYLE|此值始终为 `SQL`。|
|IS_DETERMINISTIC|`YES` 或 `NO`，取决于例程是否定义为 `DETERMINISTIC` 特性。|
|SQL_DATA_ACCESS|例程的数据访问特性。值为 `CONTAINS SQL`、`NO SQL`、`READS SQL DATA` 或 `MODIFIES SQL DATA` 中的一个。|
|SQL_PATH|此值始终为 `NULL`。|
|SECURITY_TYPE|例程的 `SQL SECURITY` 特性。值为 `DEFINER` 或 `INVOKER`。|
|CREATED|创建例程的日期和时间。为 `DATETIME` 类型的值。|
|LAST_ALTERED|最后修改例程的日期和时间。为 `DATETIME` 类型的值。如果例程自创建以来未有修改，则此值与 `CREATED` 值相同。|
|SQL_MODE|创建或修改例程时生效的 SQL 模式，以及例程执行时的 SQL 模式。|
|ROUTINE_COMMENT|例程的注释文本，若有。若无，则此值为空。|
|DEFINER|在 `DEFINER` 子句中指定的用户（通常是创建例程的用户）。|
|CHARACTER_SET_CLIENT|客户端字符集。|
|COLLATION_CONNECTION|连接校对。|
|DATABASE_COLLATION|与例程关联的数据库校对规则。|
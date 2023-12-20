---
displayed_sidebar: English
---

# 存储例程

存储例程包括所有的存储过程和存储函数。

在例程中提供了以下字段：

|字段|描述|
|---|---|
|SPECIFIC_NAME|例程的名称。|
|ROUTINE_CATALOG|例程所属目录的名称。该值始终是默认值。|
|ROUTINE_SCHEMA|例程所属的数据库的名称。|
|ROUTINE_NAME|例程的名称。|
|ROUTINE_TYPE|PROCEDURE 用于存储过程，FUNCTION 用于存储函数。|
|DTD_IDENTIFIER|如果例程是存储函数，则返回值数据类型。如果例程是存储过程，则该值为空。|
|ROUTINE_BODY|用于例程定义的语言。该值始终是 SQL。|
|ROUTINE_DEFINITION|例程执行的 SQL 语句的文本。|
|EXTERNAL_NAME|此值始终为 NULL。|
|EXTERNAL_LANGUAGE|存储例程的语言。|
|PARAMETER_STYLE|此值始终为 SQL。|
|IS_DETERMINISTIC|YES 或 NO，取决于例程是否使用 DETERMINISTIC 特征定义。|
|SQL_DATA_ACCESS|例程的数据访问特征。该值为 CONTAINS SQL、NO SQL、READS SQL DATA 或 MODIFIES SQL DATA 之一。|
|SQL_PATH|此值始终为 NULL。|
|SECURITY_TYPE|例程 SQL SECURITY 特征。该值为 DEFINER 或 INVOKER 之一。|
|CREATED|创建例程的日期和时间。这是一个 DATETIME 值。|
|LAST_ALTERED|上次修改例程的日期和时间。这是一个 DATETIME 值。如果例程自创建以来未曾修改过，则该值与 CREATED 值相同。|
|SQL_MODE|创建或更改例程时有效的 SQL 模式以及例程执行时的 SQL 模式。|
|ROUTINE_COMMENT|注释文本（如果例程有）。如果不是，则该值为空。|
|DEFINER|DEFINER 子句中指定的用户（通常是创建例程的用户）。|
|字符集客户端|
|COLLATION_CONNECTION|
|DATABASE_COLLATION|与例程关联的数据库的排序规则。|

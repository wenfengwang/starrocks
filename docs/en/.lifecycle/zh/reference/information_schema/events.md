---
displayed_sidebar: English
---

# 事件

`events` 提供有关事件管理器事件的信息。

`events` 中提供了以下字段：

|**字段**|**描述**|
|---|---|
|EVENT_CATALOG|事件所属目录的名称。该值始终是 `def`。|
|EVENT_SCHEMA|事件所属的数据库的名称。|
|EVENT_NAME|事件的名称。|
|DEFINER|`DEFINER` 子句中指定的用户（通常是创建事件的用户）。|
|TIME_ZONE|事件时区，它是用于安排事件的时区，并且在事件执行时在事件内有效。默认值为 `SYSTEM`。|
|EVENT_BODY|事件 `DO` 子句中的语句所使用的语言。该值始终为 `SQL`。|
|EVENT_DEFINITION|构成事件 `DO` 子句的 SQL 语句的文本；换句话说，该事件执行的语句。|
|EVENT_TYPE|事件重复类型，`ONE TIME`（一次性）或 `RECURRING`（重复）。|
|EXECUTE_AT|对于一次性事件，这是在用于创建事件的 `CREATE EVENT` 语句或修改事件的最后一个 `ALTER EVENT` 语句的 `AT` 子句中指定的 `DATETIME` 值。此列中显示的值反映了事件 `AT` 子句中包含的任何 `INTERVAL` 值的加法或减法。例如，如果使用 `ON SCHEDULE AT CURRENT_DATETIME + '1:6' DAY_HOUR` 创建事件，并且该事件创建于 2018-02-09 14:05:30，则此列中显示的值将为 `'2018-02-10 20:05:30'`。如果事件的计时由 `EVERY` 子句而不是 `AT` 子句确定（即，如果事件重复发生），则此列的值为 `NULL`。|
|INTERVAL_VALUE|对于重复事件，事件执行之间等待的时间间隔数。对于一次性事件，该值始终为 `NULL`。|
|INTERVAL_FIELD|用于重复事件在重复之前等待的时间间隔的时间单位。对于一次性事件，该值始终为 `NULL`。|
|SQL_MODE|创建或更改事件时有效的 SQL 模式，以及事件在该模式下执行。|
|STARTS|重复事件的开始日期和时间。这显示为 `DATETIME` 值，如果没有为事件定义开始日期和时间，则为 `NULL`。对于一次性事件，此列始终为 `NULL`。对于其定义包含 `STARTS` 子句的重复事件，此列包含相应的 `DATETIME` 值。与 `EXECUTE_AT` 列一样，该值解析使用的任何表达式。如果没有影响事件计时的 `STARTS` 子句，则此列为 `NULL`。|
|ENDS|对于其定义包含 `ENDS` 子句的重复事件，此列包含相应的 `DATETIME` 值。与 `EXECUTE_AT` 列一样，该值解析使用的任何表达式。如果没有影响事件计时的 `ENDS` 子句，则此列为 `NULL`。|
|STATUS|事件状态。`ENABLED`、`DISABLED` 或 `SLAVESIDE_DISABLED` 之一。`SLAVESIDE_DISABLED` 指示事件的创建发生在充当复制源的另一台 MySQL 服务器上，并复制到充当副本的当前 MySQL 服务器，但该事件当前并未在副本上执行。|
|ON_COMPLETION|有效值：`PRESERVE` 和 `NOT PRESERVE`。|
|CREATED|创建事件的日期和时间。这是一个 `DATETIME` 值。|
|LAST_ALTERED|上次修改事件的日期和时间。这是一个 `DATETIME` 值。如果事件自创建以来未曾修改过，则该值与 `CREATED` 值相同。|
|LAST_EXECUTED|上次执行事件的日期和时间。这是一个 `DATETIME` 值。如果事件从未执行过，则此列为 `NULL`。`LAST_EXECUTED` 指示事件开始的时间。因此，`ENDS` 列永远不会小于 `LAST_EXECUTED`。|
|EVENT_COMMENT|评论文本（如果事件有）。如果没有，则该值为空。|
|ORIGINATOR|创建事件的 MySQL 服务器的服务器 ID；用于复制。如果在复制源上执行，则 `ALTER EVENT` 可以将此值更新为发生该语句的服务器的服务器 ID。默认值为 0。|
|CHARACTER_SET_CLIENT|创建事件时 `character_set_client` 系统变量的会话值。|
|COLLATION_CONNECTION|创建事件时 `collation_connection` 系统变量的会话值。|
|DATABASE_COLLATION|与事件关联的数据库的排序规则。|
---
displayed_sidebar: English
---

# 事件

`events` 提供有关事件管理器事件的信息。

`events` 中提供了以下字段：

| **字段**            | **描述**                                              |
| -------------------- | ------------------------------------------------------------ |
| EVENT_CATALOG        | 事件所属的目录的名称。此值始终为 `def`。 |
| EVENT_SCHEMA         | 事件所属的数据库的名称。         |
| EVENT_NAME           | 事件的名称。                                       |
| DEFINER              | 在 `DEFINER` 子句中指定的用户（通常是创建事件的用户）。 |
| TIME_ZONE            | 事件的时区，即用于计划事件的时区，在事件执行时生效。默认值为 `SYSTEM`。 |
| EVENT_BODY           | 事件子句中语句使用的语言。`DO` 子句的值始终为 `SQL`。 |
| EVENT_DEFINITION     | 构成事件 `DO` 子句的 SQL 语句的文本；换句话说，就是此事件执行的语句。 |
| EVENT_TYPE           | 事件的重复类型，可以是 `ONE TIME`（一次性）或 `RECURRING`（重复）。 |
| EXECUTE_AT           | 对于一次性事件，这是在 `CREATE EVENT` 语句的 `AT` 子句中指定的 `DATETIME` 值，或者最后一个修改事件的 `ALTER EVENT` 语句中的 `AT` 子句的值。此列中显示的值反映了事件子句中包含的任何 `INTERVAL` 值的加法或减法。例如，如果使用 `ON SCHEDULE AT CURRENT_DATETIME + '1:6' DAY_HOUR` 创建事件，并且该事件是在 2018-02-09 14：05：30 创建的，则此列中显示的值将为 `'2018-02-10 20:05:30'`。如果事件的时间由 `EVERY` 子句而不是 `AT` 子句确定（即，如果事件重复发生），则此列的值为 `NULL`。 |
| INTERVAL_VALUE       | 对于重复事件，是指在事件执行之间等待的间隔数。对于一次性事件，该值始终为 `NULL`。 |
| INTERVAL_FIELD       | 用于重复事件在重复之前等待的时间间隔的时间单位。对于一次性事件，该值始终为 `NULL`。 |
| SQL_MODE             | 创建或修改事件时生效的 SQL 模式，以及事件执行时生效的 SQL 模式。 |
| STARTS               | 定期事件的开始日期和时间。如果未为事件定义开始日期和时间，则此列显示为 `NULL`。对于一次性事件，此列始终为 `NULL`。对于定义包含 `STARTS` 子句的定期事件，此列包含相应的 `DATETIME` 值。与 `EXECUTE_AT` 列一样，此值解析使用的任何表达式。如果没有 `STARTS` 子句影响事件时间，则此列为 `NULL`。 |
| ENDS                 | 对于定义包含 `ENDS` 子句的定期事件，此列包含相应的 `DATETIME` 值。与 `EXECUTE_AT` 列一样，此值解析使用的任何表达式。如果没有 `ENDS` 子句影响事件时间，则此列为 `NULL`。 |
| STATUS               | 事件的状态，可以是 `ENABLED`、`DISABLED` 或 `SLAVESIDE_DISABLED`。`SLAVESIDE_DISABLED` 表示事件的创建发生在另一个充当复制源的 MySQL 服务器上，并复制到充当副本的当前 MySQL 服务器，但该事件当前未在副本上执行。 |
| ON_COMPLETION        | 有效值：`PRESERVE` 和 `NOT PRESERVE`。                 |
| CREATED              | 创建事件的日期和时间。这是一个 `DATETIME` 值。 |
| LAST_ALTERED         | 上次修改事件的日期和时间。这是一个 `DATETIME` 值。如果事件自创建以来未被修改，则此值与 `CREATED` 值相同。 |
| LAST_EXECUTED        | 上次执行事件的日期和时间。这是一个 `DATETIME` 值。如果事件从未执行过，则此列为 `NULL`。`LAST_EXECUTED` 指示事件的开始时间。因此，`ENDS` 列永远不会小于 `LAST_EXECUTED`。 |
| EVENT_COMMENT        | 注释的文本（如果事件有注释）。如果没有注释，此值为空。 |
| ORIGINATOR           | 创建事件的 MySQL 服务器的服务器 ID；用于复制。如果在复制源上执行 `ALTER EVENT`，则此值可以更新为执行该语句的服务器的服务器 ID。默认值为 0。 |
| CHARACTER_SET_CLIENT | 创建事件时的 `character_set_client` 系统变量的会话值。 |
| COLLATION_CONNECTION | 创建事件时的 `collation_connection` 系统变量的会话值。 |
| DATABASE_COLLATION   | 与事件关联的数据库的排序规则。 |

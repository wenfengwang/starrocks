---
displayed_sidebar: "Chinese"
---

# 事件

`events` 提供有关事件管理器事件的信息。

`events` 提供以下字段：

| **字段**              | **描述**                                                      |
| -------------------- | ------------------------------------------------------------ |
| EVENT_CATALOG        | 事件所属目录的名称。该值始终为`def`。                         |
| EVENT_SCHEMA         | 事件所属数据库的名称。                                         |
| EVENT_NAME           | 事件的名称。                                                   |
| DEFINER              | 在`DEFINER`子句中指定的用户（通常是创建事件的用户）。          |
| TIME_ZONE            | 事件的时区，用于安排事件并在事件执行时生效的时区。默认值为`SYSTEM`。 |
| EVENT_BODY           | 事件“DO”子句中语句的语言。该值始终为`SQL`。                |
| EVENT_DEFINITION     | 组成事件“DO”子句的SQL语句的文本；换句话说，事件执行的语句。  |
| EVENT_TYPE           | 事件的重复类型，可为`ONE TIME`（一次性）或`RECURRING`（重复）。 |
| EXECUTE_AT           | 对于一次性事件，这是在`CREATE EVENT`语句的`AT`子句中指定的`DATETIME`值，或者修改事件的最后一个`ALTER EVENT`语句中指定的值。此列中显示的值反映了事件的`AT`子句中包含的任何`INTERVAL`值的增加或减少。例如，如果使用`ON SCHEDULE AT CURRENT_DATETIME + '1:6' DAY_HOUR`创建事件，并且事件是在2018-02-09 14:05:30创建的，则该列中显示的值将为`'2018-02-10 20:05:30'`。如果事件的时间由`EVERY`子句而不是`AT`子句确定（即如果事件是重复的），则此列的值为`NULL`。 |
| INTERVAL_VALUE       | 对于重复事件，事件执行之间等待的间隔数量。对于一次性事件，该值始终为`NULL`。 |
| INTERVAL_FIELD       | 重复事件等待重复之前使用的时间单位。对于一次性事件，该值始终为`NULL`。 |
| SQL_MODE             | 事件创建或修改时生效的SQL模式，以及事件执行时所采用的SQL模式。 |
| STARTS               | 重复事件的开始日期和时间。这显示为`DATETIME`值，如果未为事件定义开始日期和时间，则为空。对于一次性事件，此列始终为空。对于其定义包括`STARTS`子句的重复事件，此列包含相应的`DATETIME`值。与`EXECUTE_AT`列一样，此值解析了任何使用的表达式。如果没有影响事件时间的`STARTS`子句，则此列为空。 |
| ENDS                 | 对于其定义包括`ENDS`子句的重复事件，此列包含相应的`DATETIME`值。与`EXECUTE_AT`列一样，此值解析了任何使用的表达式。如果没有影响事件时间的`ENDS`子句，则此列为空。 |
| STATUS               | 事件状态。为`ENABLED`、`DISABLED`或`SLAVESIDE_DISABLED`中的一个。`SLAVESIDE_DISABLED`表示事件是在充当复制源的另一个MySQL服务器上创建并已复制到充当副本的当前MySQL服务器，但是事件目前未在副本上执行。 |
| ON_COMPLETION        | 有效值为`PRESERVE`和`NOT PRESERVE`。                          |
| CREATED              | 事件创建的日期和时间。这是一个`DATETIME`值。               |
| LAST_ALTERED         | 事件上次修改的日期和时间。这是一个`DATETIME`值。如果事件自创建以来没有被修改，此值与`CREATED`值相同。 |
| LAST_EXECUTED        | 事件上次执行的日期和时间。这是一个`DATETIME`值。如果事件从未执行过，则此列为`NULL`。`LAST_EXECUTED`表示事件启动的时间。因此，`ENDS`列永远不会早于`LAST_EXECUTED`。 |
| EVENT_COMMENT        | 事件的注释文本。如果没有，则此值为空。                    |
| ORIGINATOR           | 创建事件的MySQL服务器的服务器ID；用于复制。如果在复制源上执行，则此值可以由`ALTER EVENT`更新为该语句发生的服务器的服务器ID。默认值为0。 |
| CHARACTER_SET_CLIENT | 创建事件时的`character_set_client`系统变量的会话值。         |
| COLLATION_CONNECTION | 创建事件时的`collation_connection`系统变量的会话值。         |
| DATABASE_COLLATION   | 事件关联的数据库的排序规则。                                |
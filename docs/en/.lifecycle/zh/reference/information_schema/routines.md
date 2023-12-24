---
displayed_sidebar: English
---

# 例程

`routines` 包含所有存储例程（存储过程和存储函数）。

`routine` 中提供了以下字段：

| **字段**            | **描述**                                              |
| -------------------- | ------------------------------------------------------------ |
| SPECIFIC_NAME        | 例程的名称。                                     |
| ROUTINE_CATALOG      | 例程所属的目录名称。该值始终为 `def`。 |
| ROUTINE_SCHEMA       | 例程所属的数据库名称。       |
| ROUTINE_NAME         | 例程的名称。                                     |
| ROUTINE_TYPE         | 存储过程的情况下为 `PROCEDURE`，存储函数的情况下为 `FUNCTION`。 |
| DTD_IDENTIFIER       | 如果例程是存储函数，则为返回值的数据类型。如果例程是存储过程，则该值为空。 |
| ROUTINE_BODY         | 例程定义所使用的语言。该值始终为 `SQL`。 |
| ROUTINE_DEFINITION   | 例程执行的 SQL 语句文本。       |
| EXTERNAL_NAME        | 该值始终为 `NULL`。                                 |
| EXTERNAL_LANGUAGE    | 存储例程所使用的语言。                          |
| PARAMETER_STYLE      | 该值始终为 `SQL`。                                  |
| IS_DETERMINISTIC     | 根据例程是否使用 `DETERMINISTIC` 特性定义，为 `YES` 或 `NO`。 |
| SQL_DATA_ACCESS      | 例程的数据访问特性。该值为 `CONTAINS SQL`、 `NO SQL`、 `READS SQL DATA`或 `MODIFIES SQL DATA`之一。 |
| SQL_PATH             | 该值始终为 `NULL`。                                 |
| SECURITY_TYPE        | 例程的 `SQL SECURITY` 特性。该值为 `DEFINER` 或 `INVOKER`之一。 |
| CREATED              | 例程创建的日期和时间。这是一个 `DATETIME` 值。 |
| LAST_ALTERED         | 例程上次修改的日期和时间。这是一个 `DATETIME` 值。如果例程自创建以来未被修改，则该值与 `CREATED` 值相同。 |
| SQL_MODE             | 例程创建或修改时生效的 SQL 模式，以及例程执行时生效的 SQL 模式。 |
| ROUTINE_COMMENT      | 注释的文本（如果例程有注释）。如果没有，则该值为空。 |
| DEFINER              | 在 `DEFINER` 子句中指定的用户（通常是创建例程的用户）。 |
| CHARACTER_SET_CLIENT |                                                              |
| COLLATION_CONNECTION |                                                              |
| DATABASE_COLLATION   | 与例程关联的数据库的排序规则。 |
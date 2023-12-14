---
displayed_sidebar: "Chinese"
---

# 例行程序

`例行程序` 包含所有存储的例行程序（存储过程和存储函数）。

`例行程序` 提供以下字段：

| **字段**               | **描述**                                                      |
| ---------------------- | ------------------------------------------------------------ |
| SPECIFIC_NAME          | 例行程序的名称。                                              |
| ROUTINE_CATALOG        | 例行程序所属目录的名称。此值始终为 `def`。                  |
| ROUTINE_SCHEMA         | 例行程序所属数据库的名称。                                    |
| ROUTINE_NAME           | 例行程序的名称。                                              |
| ROUTINE_TYPE           | 存储过程为 `PROCEDURE`，存储函数为 `FUNCTION`。                |
| DTD_IDENTIFIER         | 如果例行程序是存储函数，则为返回值数据类型。如果例行程序是存储过程，则此值为空。 |
| ROUTINE_BODY           | 例行程序定义所使用的语言。此值始终为 `SQL`。                  |
| ROUTINE_DEFINITION     | 例行程序执行的 SQL 语句文本。                                 |
| EXTERNAL_NAME          | 此值始终为 `NULL`。                                           |
| EXTERNAL_LANGUAGE      | 存储例行程序的语言。                                          |
| PARAMETER_STYLE        | 此值始终为 `SQL`。                                            |
| IS_DETERMINISTIC       | 取决于例行程序是否使用了 `DETERMINISTIC` 特性，为 `YES` 或 `NO`。 |
| SQL_DATA_ACCESS        | 例行程序的数据访问特性。值为 `CONTAINS SQL`、`NO SQL`、`READS SQL DATA` 或 `MODIFIES SQL DATA` 中的一个。 |
| SQL_PATH               | 此值始终为 `NULL`。                                           |
| SECURITY_TYPE          | 例行程序的 `SQL SECURITY` 特性。值为 `DEFINER` 或 `INVOKER` 中的一个。 |
| CREATED                | 例行程序创建的日期和时间。这是一个 `DATETIME` 值。           |
| LAST_ALTERED           | 例行程序上次修改的日期和时间。这是一个 `DATETIME` 值。如果例行程序自创建以来未被修改，则该值与 `CREATED` 值相同。 |
| SQL_MODE               | 例行程序创建或修改时有效的 SQL 模式，以及例行程序执行时所在的模式。 |
| ROUTINE_COMMENT        | 例行程序的注释文本（如果存在）。如果不存在，则此值为空。     |
| DEFINER                | `DEFINER` 子句中指定的用户（通常为创建例行程序的用户）。     |
| CHARACTER_SET_CLIENT   |                                                              |
| COLLATION_CONNECTION   |                                                              |
| DATABASE_COLLATION     | 例行程序关联的数据库的排序规则。                               |
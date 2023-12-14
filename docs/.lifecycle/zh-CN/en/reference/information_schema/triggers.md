---
displayed_sidebar: "中文"
---

# 触发器

`triggers`提供有关触发器的信息。

在`triggers`中提供以下字段：

| **字段**                     | **描述**                                                     |
| -------------------------- | ------------------------------------------------------------ |
| TRIGGER_CATALOG            | 触发器所属目录的名称。此值始终为`def`。                      |
| TRIGGER_SCHEMA             | 触发器所属数据库的名称。                                       |
| TRIGGER_NAME               | 触发器的名称。                                                 |
| EVENT_MANIPULATION         | 触发器事件。这是触发器激活的相关表的操作类型。值为`INSERT`（已插入行），`DELETE`（已删除行）或`UPDATE`（已修改行）。 |
| EVENT_OBJECT_CATALOG       | 每个触发器都与一个表关联。该表所在的目录。                    |
| EVENT_OBJECT_SCHEMA        | 每个触发器都与一个表关联。该表所在的数据库。                   |
| EVENT_OBJECT_TABLE         | 触发器关联的表的名称。                                         |
| ACTION_ORDER               | 触发器动作在相同表的触发器列表中与相同`EVENT_MANIPULATION`和`ACTION_TIMING`值的顺序位置。 |
| ACTION_CONDITION           | 该值始终为`NULL`。                                             |
| ACTION_STATEMENT           | 触发器主体；即触发器激活时执行的语句。此文本使用UTF-8编码。    |
| ACTION_ORIENTATION         | 该值始终为`ROW`。                                             |
| ACTION_TIMING              | 触发器在触发事件之前或之后激活。该值为`BEFORE`或`AFTER`。       |
| ACTION_REFERENCE_OLD_TABLE | 该值始终为`NULL`。                                             |
| ACTION_REFERENCE_NEW_TABLE | 该值始终为`NULL`。                                             |
| ACTION_REFERENCE_OLD_ROW   | 旧列标识符。该值始终为`OLD`。                                  |
| ACTION_REFERENCE_NEW_ROW   | 新列标识符。该值始终为`NEW`。                                  |
| CREATED                    | 创建触发器的日期和时间。这是触发器的`DATETIME(2)`值（以秒的百分之一部分为小数） |
| SQL_MODE                   | 创建触发器时生效的SQL模式，以及触发器执行的模式。             |
| DEFINER                    | `DEFINER`子句中指定的用户（通常是创建触发器的用户）。          |
| CHARACTER_SET_CLIENT       |                                                              |
| COLLATION_CONNECTION       |                                                              |
| DATABASE_COLLATION         | 触发器关联的数据库的排序规则。                                 |
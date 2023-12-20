---
displayed_sidebar: English
---

# 触发器

`triggers` 提供有关触发器的信息。

`triggers` 中提供了以下字段：

|**字段**|**描述**|
|---|---|
|TRIGGER_CATALOG|触发器所属的目录名称。该值始终为 `def`。|
|TRIGGER_SCHEMA|触发器所属的数据库名称。|
|TRIGGER_NAME|触发器的名称。|
|EVENT_MANIPULATION|触发器事件。这是关联表上触发器激活的操作类型。值为 `INSERT`（插入了行）、`DELETE`（删除了行）或 `UPDATE`（修改了行）。|
|EVENT_OBJECT_CATALOG|每个触发器都与恰好一个表相关联。该表所在的目录。|
|EVENT_OBJECT_SCHEMA|每个触发器都与恰好一个表相关联。该表所在的数据库。|
|EVENT_OBJECT_TABLE|触发器关联的表的名称。|
|ACTION_ORDER|触发器动作在具有相同 `EVENT_MANIPULATION` 和 `ACTION_TIMING` 值的同一表上的触发器列表中的顺序位置。|
|ACTION_CONDITION|此值始终为 `NULL`。|
|ACTION_STATEMENT|触发器主体；即，触发器激活时执行的语句。此文本使用 UTF-8 编码。|
|ACTION_ORIENTATION|此值始终为 `ROW`。|
|ACTION_TIMING|触发器是在触发事件之前还是之后激活。值为 `BEFORE` 或 `AFTER`。|
|ACTION_REFERENCE_OLD_TABLE|此值始终为 `NULL`。|
|ACTION_REFERENCE_NEW_TABLE|此值始终为 `NULL`。|
|ACTION_REFERENCE_OLD_ROW|旧列标识符。该值始终为 `OLD`。|
|ACTION_REFERENCE_NEW_ROW|新列标识符。该值始终为 `NEW`。|
|CREATED|触发器创建的日期和时间。这是触发器的 `DATETIME(2)` 值（带有百分之一秒的小数部分）。|
|SQL_MODE|创建触发器时生效的 SQL 模式，触发器也在该模式下执行。|
|DEFINER|在 `DEFINER` 子句中命名的用户（通常是创建触发器的用户）。|
|CHARACTER_SET_CLIENT|客户端字符集。|
|COLLATION_CONNECTION|连接校对。|
|DATABASE_COLLATION|与触发器关联的数据库的校对规则。|
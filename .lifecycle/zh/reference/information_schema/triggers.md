---
displayed_sidebar: English
---

# 触发器

触发器提供有关触发器的信息。

以下是触发器中提供的字段：

|字段|描述|
|---|---|
|TRIGGER_CATALOG|触发器所属目录的名称。该值始终是默认值。|
|TRIGGER_SCHEMA|触发器所属数据库的名称。|
|TRIGGER_NAME|触发器的名称。|
|EVENT_MANIPULATION|触发事件。这是触发器激活的关联表上的操作类型。值为 INSERT（插入行）、DELETE（删除行）或 UPDATE（修改行）。|
|EVENT_OBJECT_CATALOG|每个触发器都与一个表相关联。此表所在的目录。|
|EVENT_OBJECT_SCHEMA|每个触发器都与一个表相关联。此表所在的数据库。|
|EVENT_OBJECT_TABLE|与触发器关联的表的名称。|
|ACTION_ORDER|触发器操作在具有相同 EVENT_MANIPULATION 和 ACTION_TIMING 值的同一表上的触发器列表中的顺序位置。|
|ACTION_CONDITION|此值始终为 NULL。|
|ACTION_STATMENT|触发器主体；即触发器激活时执行的语句。此文本使用 UTF-8 编码。|
|ACTION_ORIENTATION|此值始终为 ROW。|
|ACTION_TIMING|触发器是在触发事件之前还是之后激活。该值是之前或之后。|
|ACTION_REFERENCE_OLD_TABLE|此值始终为 NULL。|
|ACTION_REFERENCE_NEW_TABLE|此值始终为 NULL。|
|ACTION_REFERENCE_OLD_ROW|旧列标识符。该值始终为旧值。|
|ACTION_REFERENCE_NEW_ROW|新列标识符。价值始终是新的。|
|CREATED|创建触发器的日期和时间。这是触发器的 DATETIME(2) 值（带有百分之一秒的小数部分）。|
|SQL_MODE|创建触发器时有效的 SQL 模式，以及触发器在该模式下执行。|
|DEFINER|DEFINER 子句中指定的用户（通常是创建触发器的用户）。|
|字符集客户端|
|COLLATION_CONNECTION|
|DATABASE_COLLATION|与触发器关联的数据库的排序规则。|

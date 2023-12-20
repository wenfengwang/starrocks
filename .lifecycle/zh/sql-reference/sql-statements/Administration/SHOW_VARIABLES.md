---
displayed_sidebar: English
---

# 显示变量

## 描述

展示StarRocks的系统变量。关于系统变量的详细信息，请参阅[System Variables](../../../reference/System_variable.md)文档。

:::提示

此操作不需要特殊权限。

:::

## 语法

```SQL
SHOW [ GLOBAL | SESSION ] VARIABLES
    [ LIKE <pattern> | WHERE <expr> ]
```

## 参数

|参数|说明|
|---|---|
|修饰符:GLOBALSESSION|使用 GLOBAL 修饰符，该语句显示全局系统变量值。这些值用于初始化 StarRocks 新连接的相应会话变量。如果变量没有全局值，则不显示任何值。使用 SESSION 修饰符，该语句将显示对当前连接有效的系统变量值。如果变量没有会话值，则显示全局值。 LOCAL 是 SESSION 的同义词。如果没有修饰符，则默认为 SESSION。|
|pattern|用于通过变量名称与 LIKE 子句匹配变量的模式。您可以在此参数中使用 % 通配符。|
|expr|用于通过变量名variable_name或变量值value与WHERE子句来匹配变量的表达式。|

## 返回值

|返回|说明|
|---|---|
|Variable_name|变量的名称。|
|值|变量的值。|

## 示例

示例1：使用LIKE子句精确匹配变量名来显示一个变量。

```Plain
mysql> SHOW VARIABLES LIKE 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1 row in set (0.01 sec)
```

示例2：使用LIKE子句和通配符（%）模糊匹配变量名来显示变量。

```Plain
mysql> SHOW VARIABLES LIKE '%imeou%';
+------------------------------------+-------+
| Variable_name                      | Value |
+------------------------------------+-------+
| interactive_timeout                | 3600  |
| net_read_timeout                   | 60    |
| net_write_timeout                  | 60    |
| new_planner_optimize_timeout       | 3000  |
| query_delivery_timeout             | 300   |
| query_queue_pending_timeout_second | 300   |
| query_timeout                      | 300   |
| tx_visible_wait_timeout            | 10    |
| wait_timeout                       | 28800 |
+------------------------------------+-------+
9 rows in set (0.00 sec)
```

示例3：使用WHERE子句精确匹配变量名来显示一个变量。

```Plain
mysql> show variables where variable_name = 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1 row in set (0.17 sec)
```

示例4：使用WHERE子句精确匹配变量值来显示一个变量。

```Plain
mysql> show variables where value = '28800';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1 row in set (0.70 sec)
```

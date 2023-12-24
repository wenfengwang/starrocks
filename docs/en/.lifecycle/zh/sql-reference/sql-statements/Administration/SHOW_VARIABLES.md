---
displayed_sidebar: English
---

# 显示变量

## 描述

显示 StarRocks 的系统变量。有关系统变量的详细信息，请参阅 [系统变量](../../../reference/System_variable.md)。

:::提示

此操作无需特权。

:::

## 语法

```SQL
SHOW [ GLOBAL | SESSION ] VARIABLES
    [ LIKE <pattern> | WHERE <expr> ]
```

## 参数

| **参数**          | **描述**                                              |
| ---------------------- | ------------------------------------------------------------ |
| 修饰符：<ul><li>GLOBAL</li><li>SESSION</li></ul> | <ul><li>使用 `GLOBAL` 修饰符，该语句显示 StarRocks 的全局系统变量值。这些值用于初始化新连接到 StarRocks 的相应会话变量。如果变量没有全局值，则不显示任何值。</li><li>使用 `SESSION` 修饰符，该语句显示当前连接有效的系统变量值。如果变量没有会话值，则显示全局值。 `LOCAL` 是 `SESSION` 的同义词。</li><li>如果没有修饰符，则默认为 `SESSION`。</li></ul> |
| 模式                | 用于通过 LIKE 子句匹配变量名称的模式。您可以在此参数中使用 % 通配符。 |
| 表达式                   | 用于通过 WHERE 子句匹配变量名称 `variable_name` 或变量值 `value` 的表达式。|

## 返回值

| **返回**    | **描述**            |
| ------------- | -------------------------- |
| Variable_name | 变量的名称。  |
| Value         | 变量的值。 |

## 例子

示例 1：通过在 LIKE 子句中精确匹配变量名称来显示变量。

```Plain
mysql> SHOW VARIABLES LIKE 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1 行结果 (0.01 秒)
```

示例 2：通过在 LIKE 子句中使用通配符（%）来大致匹配变量名称来显示变量。

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
9 行结果 (0.00 秒)
```

示例 3：通过在 WHERE 子句中精确匹配变量名称来显示变量。

```Plain
mysql> show variables where variable_name = 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1 行结果 (0.17 秒)
```

示例 4：通过在 WHERE 子句中精确匹配变量值来显示变量。

```Plain
mysql> show variables where value = '28800';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1 行结果 (0.70 秒)
```

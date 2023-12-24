---
displayed_sidebar: English
---

# 设置

## 描述

设置 StarRocks 的指定系统变量或用户自定义变量。您可以使用 [SHOW VARIABLES](../Administration/SHOW_VARIABLES.md) 查看 StarRocks 的系统变量。有关系统变量的详细信息，请参阅[系统变量](../../../reference/System_variable.md)。有关用户自定义变量的详细信息，请参阅[用户自定义变量](../../../reference/user_defined_variables.md)。

:::提示

此操作不需要特殊权限。

:::

## 语法

```SQL
SET [ GLOBAL | SESSION ] <variable_name> = <value> [, <variable_name> = <value>] ...
```

## 参数

| **参数**          | **描述**                                              |
| ---------------------- | ------------------------------------------------------------ |
| 修饰符：<ul><li>GLOBAL</li><li>SESSION</li></ul> | <ul><li>使用 `GLOBAL` 修饰符，该语句在全局范围内设置变量。</li><li>使用 `SESSION` 修饰符，该语句在会话中设置变量。`LOCAL` 是 `SESSION` 的同义词。</li><li>如果没有修饰符，则默认为 `SESSION`。</li></ul>有关全局和会话变量的详细信息，请参阅[系统变量](../../../reference/System_variable.md)。<br/>**注意**<br/>只有具有 ADMIN 权限的用户才能在全局范围内设置变量。 |
| variable_name          | 变量的名称。                                    |
| value                  | 变量的值。                                   |

## 例子

示例 1：在会话中将 `time_zone` 设置为 `Asia/Shanghai`。

```Plain
mysql> SET time_zone = "Asia/Shanghai";
Query OK, 0 rows affected (0.00 sec)
```

示例 2：将 `exec_mem_limit` 在全局范围内设置为 `2147483648`。

```Plain
mysql> SET GLOBAL exec_mem_limit = 2147483648;
Query OK, 0 rows affected (0.00 sec)
```

示例 3：设置多个全局变量。每个变量都需要在前面加上 `GLOBAL` 关键字。

```Plain
mysql> SET 
       GLOBAL exec_mem_limit = 2147483648,
       GLOBAL time_zone = "Asia/Shanghai";
Query OK, 0 rows affected (0.00 sec)
```
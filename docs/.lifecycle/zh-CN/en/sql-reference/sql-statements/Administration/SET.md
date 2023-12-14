---
displayed_sidebar: "中文"
---

# SET

## 描述

设置StarRocks的指定系统变量或用户定义变量。您可以使用 [SHOW VARIABLES](../Administration/SHOW_VARIABLES.md) 查看StarRocks的系统变量。有关系统变量的详细信息，请参阅[系统变量](../../../reference/System_variable.md)。有关用户定义变量的详细信息，请参阅[用户定义变量](../../../reference/user_defined_variables.md)。

## 语法

```SQL
SET [ GLOBAL | SESSION ] <variable_name> = <value> [, <variable_name> = <value>] ...
```

## 参数

| **参数**                | **描述**                                                     |
| ----------------------- | ------------------------------------------------------------ |
| 修改器:<ul><li>GLOBAL</li><li>SESSION</li></ul> | <ul><li>带有 `GLOBAL` 修改器，语句会全局设置变量。</li><li>带有 `SESSION` 修改器，语句会在会话中设置变量。 `LOCAL` 是 `SESSION` 的同义词。</li><li>如果没有修改器，则默认为 `SESSION`。</li></ul>有关全局和会话变量的详细信息，请参阅[系统变量](../../../reference/System_variable.md)。<br/>**注意**<br/>只有具有 ADMIN 权限的用户才能全局设置变量。 |
| variable_name          | 变量名称。                                                   |
| value                  | 变量值。                                                     |

## 示例

示例 1: 在会话中将 `time_zone` 设置为 `Asia/Shanghai`。

```Plain
mysql> SET time_zone = "Asia/Shanghai";
查询 OK，受影响的行：0，用时 0.00 秒
```

示例 2: 全局将 `exec_mem_limit` 设置为 `2147483648`。

```Plain
mysql> SET GLOBAL exec_mem_limit = 2147483648;
查询 OK，受影响的行：0，用时 0.00 秒
```

示例 3: 设置多个全局变量。需要为每个变量前置 `GLOBAL` 关键字。

```Plain
mysql> SET 
       GLOBAL exec_mem_limit = 2147483648,
       GLOBAL time_zone = "Asia/Shanghai";
查询 OK，受影响的行：0，用时 0.00 秒
```
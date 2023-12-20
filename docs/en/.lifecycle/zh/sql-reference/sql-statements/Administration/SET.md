---
displayed_sidebar: English
---

# SET

## 描述

为 StarRocks 设置指定的系统变量或用户自定义变量。您可以使用 [SHOW VARIABLES](../Administration/SHOW_VARIABLES.md) 查看 StarRocks 的系统变量。有关系统变量的详细信息，请参见 [System Variables](../../../reference/System_variable.md)。有关用户自定义变量的详细信息，请参阅 [User-defined variables](../../../reference/user_defined_variables.md)。

:::tip

此操作不需要权限。

:::

## 语法

```SQL
SET [ GLOBAL | SESSION ] <variable_name> = <value> [, <variable_name> = <value>] ...
```

## 参数

|**参数**|**描述**|
|---|---|
|修饰符：<ul><li>GLOBAL</li><li>SESSION</li></ul>|<ul><li>使用 `GLOBAL` 修饰符，该语句将变量设置为全局。</li><li>使用 `SESSION` 修饰符，该语句将变量设置在会话中。`LOCAL` 是 `SESSION` 的同义词。</li><li>如果没有修饰符，缺省值为 `SESSION`。</li></ul>有关全局变量和会话变量的详细信息，请参见 [System Variables](../../../reference/System_variable.md)。<br/>**注意**<br/>只有具有 ADMIN 权限的用户才能设置全局变量。|
|variable_name|变量的名称。|
|value|变量的值。|

## 示例

示例 1：在会话中将 `time_zone` 设置为 `Asia/Shanghai`。

```Plain
mysql> SET time_zone = "Asia/Shanghai";
Query OK, 0 rows affected (0.00 sec)
```

示例 2：全局设置 `exec_mem_limit` 为 `2147483648`。

```Plain
mysql> SET GLOBAL exec_mem_limit = 2147483648;
Query OK, 0 rows affected (0.00 sec)
```

示例 3：设置多个全局变量。每个变量前都需要加上 `GLOBAL` 关键字。

```Plain
mysql> SET 
       GLOBAL exec_mem_limit = 2147483648,
       GLOBAL time_zone = "Asia/Shanghai";
Query OK, 0 rows affected (0.00 sec)
```
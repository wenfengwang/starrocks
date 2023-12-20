---
displayed_sidebar: English
---

# 设置

## 描述

为 StarRocks 设置指定的系统变量或用户自定义变量。您可以使用[SHOW VARIABLES](../Administration/SHOW_VARIABLES.md)命令查看 StarRocks 的系统变量。关于系统变量的详细信息，请参阅[System Variables](../../../reference/System_variable.md)文档。关于用户自定义变量的详细信息，请参阅[User-defined variables](../../../reference/user_defined_variables.md)文档。

:::提示

此操作不需要特殊权限。

:::

## 语法

```SQL
SET [ GLOBAL | SESSION ] <variable_name> = <value> [, <variable_name> = <value>] ...
```

## 参数

|参数|说明|
|---|---|
|修饰符:GLOBALSESSION|使用 GLOBAL 修饰符，该语句在全局范围内设置变量。使用 SESSION 修饰符，该语句在会话内设置变量。 LOCAL 是 SESSION 的同义词。如果没有修饰符，则默认为 SESSION。有关全局变量和会话变量的详细信息，请参阅系统变量。注意只有具有 ADMIN 权限的用户才能全局设置变量。|
|variable_name|变量的名称。|
|值|变量的值。|

## 示例

示例 1：在会话中将 time_zone 设置为 Asia/Shanghai。

```Plain
mysql> SET time_zone = "Asia/Shanghai";
Query OK, 0 rows affected (0.00 sec)
```

示例 2：全局设置 exec_mem_limit 为 2147483648。

```Plain
mysql> SET GLOBAL exec_mem_limit = 2147483648;
Query OK, 0 rows affected (0.00 sec)
```

示例 3：设置多个全局变量。对于每个变量都需要预先添加 GLOBAL 关键字。

```Plain
mysql> SET 
       GLOBAL exec_mem_limit = 2147483648,
       GLOBAL time_zone = "Asia/Shanghai";
Query OK, 0 rows affected (0.00 sec)
```

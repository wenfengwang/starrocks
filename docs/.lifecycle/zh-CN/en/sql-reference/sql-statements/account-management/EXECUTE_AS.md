---
displayed_sidebar: "Chinese"
---

# 以...方式执行

## 描述

在获得模拟用户权限后，您可以使用EXECUTE AS语句将当前会话的执行上下文切换到该用户。

该命令支持v2.4及以上版本。

## 语法

```SQL
EXECUTE AS user WITH NO REVERT
```

## 参数

`user`: 用户必须已经存在。

## 用法说明

- 当前登录用户（调用EXECUTE AS语句的用户）必须被授予模拟其他用户的权限。有关详细信息，请参阅[GRANT](../account-management/GRANT.md)。
- EXECUTE AS语句必须包含WITH NO REVERT子句，这意味着不能在当前会话结束之前将当前会话的执行上下文切换回原始登录用户。

## 示例

将当前会话的执行上下文切换到用户`test2`。

```SQL
EXECUTE AS test2 WITH NO REVERT;
```

切换成功后，您可以运行`select current_user()`命令以获取当前用户。

```SQL
select current_user();
+-----------------------------+
| CURRENT_USER()              |
+-----------------------------+
| 'default_cluster:test2'@'%' |
+-----------------------------+
```
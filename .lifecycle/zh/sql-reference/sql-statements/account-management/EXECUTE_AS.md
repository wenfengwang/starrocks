---
displayed_sidebar: English
---

# 执行 AS

## 描述

在获得冒充用户的权限后，您可以使用 EXECUTE AS 语句将当前会话的执行上下文切换到该用户。

该命令从 v2.4 版本开始支持。

## 语法

```SQL
EXECUTE AS user WITH NO REVERT
```

## 参数

user：用户必须是已经存在的。

## 使用须知

- 当前登录用户（调用 **EXECUTE AS** 语句的用户）必须被授权模拟另一个用户的权限。更多信息请参见 [GRANT](../account-management/GRANT.md) 命令。
- EXECUTE AS 语句必须包含 WITH NO REVERT 子句，这意味着在当前会话结束之前，不能将当前会话的执行上下文切换回原登录用户。

## 示例

将当前会话的执行上下文切换到用户 test2。

```SQL
EXECUTE AS test2 WITH NO REVERT;
```

切换成功后，您可以执行 select current_user() 命令来获取当前的用户。

```SQL
select current_user();
+-----------------------------+
| CURRENT_USER()              |
+-----------------------------+
| 'default_cluster:test2'@'%' |
+-----------------------------+
```

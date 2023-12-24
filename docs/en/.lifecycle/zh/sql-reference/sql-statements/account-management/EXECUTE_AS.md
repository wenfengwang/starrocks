---
displayed_sidebar: English
---

# 执行方式

## 描述

在获得模拟用户的权限之后，您可以使用 EXECUTE AS 语句将当前会话的执行上下文切换到该用户。

此命令从 v2.4 版本开始得到支持。

## 语法

```SQL
EXECUTE AS user WITH NO REVERT
```

## 参数

`user`：用户必须已经存在。

## 使用说明

- 调用 EXECUTE AS 语句的当前登录用户必须被授予模拟其他用户的权限。有关详细信息，请参阅 [GRANT](../account-management/GRANT.md)。
- EXECUTE AS 语句必须包含 WITH NO REVERT 子句，这意味着在当前会话结束之前，当前会话的执行上下文不能切换回原始登录用户。

## 例子

将当前会话的执行上下文切换到用户 `test2`。

```SQL
EXECUTE AS test2 WITH NO REVERT;
```

切换成功后，您可以运行 `select current_user()` 命令来获取当前用户。

```SQL
select current_user();
+-----------------------------+
| CURRENT_USER()              |
+-----------------------------+
| 'default_cluster:test2'@'%' |
+-----------------------------+
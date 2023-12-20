---
displayed_sidebar: English
---

# 当前角色

## 描述

查询当前用户已激活的角色。

## 语法

```Haskell
current_role();
current_role;
```

## 参数

无。

## 返回值

返回一个 VARCHAR 类型的值。

## 示例

```Plain
mysql> select current_role();
+----------------+
| current_role() |
+----------------+
| db_admin       |
+----------------+
```

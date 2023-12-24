---
displayed_sidebar: English
---

# current_role

## 描述

查询当前用户激活的角色。

## 语法

```Haskell
current_role();
current_role;
```

## 参数

无。

## 返回值

返回一个 VARCHAR 值。

## 例子

```Plain
mysql> select current_role();
+----------------+
| current_role() |
+----------------+
| db_admin       |
+----------------+
---
displayed_sidebar: "Japanese"
---

# current_role

## Description

現在のユーザーにアクティブになっているロールをクエリします。

## Syntax

```Haskell
current_role();
current_role;
```

## Parameters

なし。

## Return value

VARCHAR 値を返します。

## Examples

```Plain
mysql> select current_role();
+----------------+
| current_role() |
+----------------+
| db_admin       |
+----------------+
```
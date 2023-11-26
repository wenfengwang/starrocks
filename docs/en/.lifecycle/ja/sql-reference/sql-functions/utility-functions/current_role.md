---
displayed_sidebar: "Japanese"
---

# current_role

## 説明

現在のユーザーに対してアクティブ化されている役割をクエリします。

## 構文

```Haskell
current_role();
current_role;
```

## パラメーター

なし。

## 戻り値

VARCHAR 値を返します。

## 例

```Plain
mysql> select current_role();
+----------------+
| current_role() |
+----------------+
| db_admin       |
+----------------+
```

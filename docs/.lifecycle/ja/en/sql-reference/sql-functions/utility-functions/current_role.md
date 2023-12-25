---
displayed_sidebar: English
---

# current_role

## 説明

現在のユーザーにアクティブなロールを照会します。

## 構文

```Haskell
current_role();
current_role;
```

## パラメーター

なし。

## 戻り値

VARCHAR 型の値を返します。

## 例

```Plain
mysql> select current_role();
+----------------+
| current_role() |
+----------------+
| db_admin       |
+----------------+
```

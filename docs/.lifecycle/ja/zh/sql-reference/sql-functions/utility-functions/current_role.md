---
displayed_sidebar: Chinese
---

# current_role

## 機能

現在のユーザーがアクティブにしているロールを取得します。この関数はバージョン3.0からサポートされています。

## 文法

```Haskell
current_role();
current_role;
```

## パラメータ説明

なし

## 戻り値の説明

戻り値のデータ型は VARCHAR です。

## 例

```Plain
mysql> select current_role();
+----------------+
| current_role() |
+----------------+
| db_admin       |
+----------------+
```

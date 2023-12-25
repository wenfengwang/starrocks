---
displayed_sidebar: Chinese
---

# isnull

## 機能

入力値が `NULL` かどうかを判断します。`NULL` の場合は 1 を返し、`NULL` でない場合は 0 を返します。

## 文法

```Haskell
isnull(v)
```

## 引数説明

- `v`: 判断する値。すべてのデータ型をサポートしています。

## 戻り値の説明

`v` が `NULL` の場合は 1 を返し、`NULL` でない場合は 0 を返します。

## 例

```Plain Text
MYSQL > SELECT c1, isnull(c1) FROM t1;
+------+--------------+
| c1   | `c1` IS NULL |
+------+--------------+
| NULL |            1 |
|    1 |            0 |
+------+--------------+
```

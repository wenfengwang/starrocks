---
displayed_sidebar: "Japanese"
---

# ISNULL（イズヌル） 

## 説明

値が `NULL` であるかどうかを調べ、`NULL` であれば `1` を返し、`NULL` でなければ `0` を返します。

## 構文

```Haskell
ISNULL(v)
```

## パラメーター

- `v`: チェックする値。すべてのデータ型がサポートされています。

## 戻り値

`NULL` であれば `1` を返し、`NULL` でなければ `0` を返します。

## 例

```plain text
MYSQL > SELECT c1, isnull(c1) FROM t1;
+------+--------------+
| c1   | `c1` IS NULL |
+------+--------------+
| NULL |            1 |
|    1 |            0 |
+------+--------------+
```
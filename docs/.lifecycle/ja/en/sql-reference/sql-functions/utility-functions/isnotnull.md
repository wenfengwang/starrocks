---
displayed_sidebar: English
---

# isnotnull

## 説明

値が `NULL` でない場合は `1` を返し、`NULL` の場合は `0` を返します。

## 構文

```Haskell
ISNOTNULL(v)
```

## パラメーター

- `v`: チェックする値。すべてのデータ型がサポートされています。

## 戻り値

`NULL` でない場合は `1` を返し、`NULL` の場合は `0` を返します。

## 例

```plain text
MYSQL > SELECT c1, isnotnull(c1) FROM t1;
+------+--------------+
| c1   | isnotnull(c1) |
+------+--------------+
| NULL |            0 |
|    1 |            1 |
+------+--------------+
```

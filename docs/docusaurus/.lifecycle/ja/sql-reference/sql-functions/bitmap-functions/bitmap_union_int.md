---
displayed_sidebar: "Japanese"
---

# bitmap_union_int

## 説明

TINYINT、SMALLINT、およびINTの列の異なる値の数をカウントし、COUNT(DISTINCT expr)の合計を返します。

## 構文

```Haskell
BIGINT bitmap_union_int(expr)
```

### パラメーター

`expr`: 列の式。サポートされている列の型はTINYINT、SMALLINT、およびINTです。

## 戻り値

BIGINT型の値を返します。

## 例

```Plaintext
mysql> select bitmap_union_int(k1) from tbl1;
+------------------------+
| bitmap_union_int(`k1`) |
+------------------------+
|                      2 |
+------------------------+
```
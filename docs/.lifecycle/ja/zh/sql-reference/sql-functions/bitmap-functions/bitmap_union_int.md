---
displayed_sidebar: Chinese
---

# bitmap_union_int

## 機能

集約関数として、TINYINT、SMALLINT、およびINT型の列で異なる値の数を計算し、その結果は`COUNT(DISTINCT expr)`と同じです。

## 文法

```Haskell
BITMAP_UNION_INT(expr)
```

## パラメータ説明

- `expr`：式で、TINYINT、SMALLINT、およびINT型の列をサポートします。

## 戻り値の説明

戻り値のデータ型はBIGINTです。

## 例

```Plaintext
mysql> select bitmap_union_int(k1) from tbl1;
+------------------------+
| bitmap_union_int(`k1`) |
+------------------------+
|                      2 |
+------------------------+
```

---
displayed_sidebar: "Japanese"
---

# exp,dexp

## Description

`x`のべき乗のeの値を返します。この関数は自然対数関数と呼ばれます。

## Syntax

```SQL
EXP(x);
```

## Parameters

`x`: べき乗の数。DOUBLEがサポートされています。

## Return value

DOUBLEデータ型の値を返します。

## Examples

3.14のeのべき乗を返します。

```Plaintext
mysql> select exp(3.14);
+--------------------+
| exp(3.14)          |
+--------------------+
| 23.103866858722185 |
+--------------------+
1 row in set (0.01 sec)
```
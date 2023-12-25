---
displayed_sidebar: English
---

# map_apply

## 説明

元のMapのキーと値に[Lambda expression](../Lambda_expression.md)を適用し、新しいMapを生成します。この関数はv3.0からサポートされています。

## 構文

```Haskell
MAP map_apply(lambda_func, any_map)
```

## パラメーター

- `lambda_func`: Lambda式。

- `any_map`: Lambda式が適用されるマップ。

## 戻り値

マップ値を返します。結果マップのキーと値のデータ型は、Lambda式の結果によって決まります。

いずれかの入力パラメータがNULLの場合、NULLが返されます。

元のマップのキーまたは値がNULLの場合、NULLは通常の値として処理されます。

Lambda式には2つのパラメータが必要です。最初のパラメータはキーを表し、2番目のパラメータは値を表します。

## 例

次の例では[map_from_arrays()](map_from_arrays.md)を使用してマップ値`{1:"ab",3:"cd"}`を生成します。次に、Lambda式は各キーを1ずつインクリメントし、各値の長さを計算します。たとえば、"ab"の長さは2です。

```SQL
mysql> select map_apply((k,v)->(k+1,length(v)), col_map)
from (select map_from_arrays([1,3],["ab","cd"]) as col_map)A;
+--------------------------------------------------+
| map_apply((k, v) -> (k + 1, length(v)), col_map) |
+--------------------------------------------------+
| {2:2,4:2}                                        |
+--------------------------------------------------+
1 row in set (0.01 sec)

mysql> select map_apply((k,v)->(k+1,length(v)), col_map)
from (select map_from_arrays(null,null) as col_map)A;
+--------------------------------------------------+
| map_apply((k, v) -> (k + 1, length(v)), col_map) |
+--------------------------------------------------+
| NULL                                             |
+--------------------------------------------------+
1 row in set (0.02 sec)

mysql> select map_apply((k,v)->(k+1,length(v)), col_map)
from (select map_from_arrays([1,null],["ab","cd"]) as col_map)A;
+--------------------------------------------------+
| map_apply((k, v) -> (k + 1, length(v)), col_map) |
+--------------------------------------------------+
| NULL                                             |
+--------------------------------------------------+
```

---
displayed_sidebar: "Japanese"
---

# map_apply

## 説明

オリジナルのMapのキーと値に[ラムダ式](../Lambda_expression.md)を適用し、新しいMapを生成します。この関数はv3.0からサポートされています。

## 構文

```Haskell
MAP map_apply(lambda_func, any_map)
```

## パラメータ

- `lambda_func`: ラムダ式です。

- `any_map`: ラムダ式が適用されるマップです。

## 戻り値

マップの値を返します。結果のマップのキーと値のデータ型は、ラムダ式の結果によって決定されます。

入力パラメータのいずれかがNULLの場合、NULLが返されます。

元のマップのキーまたは値がNULLの場合、NULLは通常の値として処理されます。

ラムダ式は2つのパラメータを持つ必要があります。最初のパラメータはキーを表し、2番目のパラメータは値を表します。

## 例

次の例では、[map_from_arrays()](map_from_arrays.md)を使用して、`{1:"ab",3:"cd"}`というマップ値を生成します。次に、ラムダ式がそれぞれのキーを1ずつ増やし、それぞれの値の長さを計算します。例えば、"ab"の長さは2です。

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
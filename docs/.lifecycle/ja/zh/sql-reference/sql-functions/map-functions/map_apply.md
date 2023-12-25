---
displayed_sidebar: Chinese
---

# map_apply

## 機能

Map のすべての Key または Value に [Lambda](../Lambda_expression.md) 関数を適用した結果の Map を返します。

この関数はバージョン 3.0 からサポートされています。

## 文法

```Haskell
MAP map_apply(lambda_func, any_map)
```

## パラメータ説明

- `lambda_func`: Map に適用する Lambda 関数。

- `any_map`: Lambda 関数を適用する Map。

## 戻り値の説明

Lambda 関数の演算結果によって決まる Key と Value のデータ型を持つ Map を返します。

入力パラメータが NULL の場合、結果も NULL です。

Map の中に NULL の Key または Value がある場合、その NULL 値は正常に計算されて返されます。

## 例

以下の例では [map_from_arrays](map_from_arrays.md) を使用して Map `{1:"ab",3:"cd"}` を生成します。次に Lambda 関数を使用して Map の Key に 1 を加え、Value の文字列長を計算します。例えば "ab" の文字列長は 2 です。

```SQL
mysql> select map_apply((k,v)->(k+1,length(v)), col_map) from (select map_from_arrays([1,3],["ab","cd"]) as col_map)A;
+--------------------------------------------------+
| map_apply((k, v) -> (k + 1, length(v)), col_map) |
+--------------------------------------------------+
| {2:2,4:2}                                        |
+--------------------------------------------------+
1 行がセットされました (0.01 秒)

mysql> select map_apply((k,v)->(k+1,length(v)), col_map) from (select map_from_arrays(null,null) as col_map)A;
+--------------------------------------------------+
| map_apply((k, v) -> (k + 1, length(v)), col_map) |
+--------------------------------------------------+
| NULL                                             |
+--------------------------------------------------+
1 行がセットされました (0.02 秒)

mysql> select map_apply((k,v)->(k+1,length(v)), col_map) from (select map_from_arrays([1,null],["ab","cd"]) as col_map)A;
+--------------------------------------------------+
| map_apply((k, v) -> (k + 1, length(v)), col_map) |
+--------------------------------------------------+
|{2:2,null:2}                                      |
+--------------------------------------------------+
```

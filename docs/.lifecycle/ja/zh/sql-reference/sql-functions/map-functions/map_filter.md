---
displayed_sidebar: Chinese
---

# map_filter

## 機能

設定されたフィルター関数に基づいて、MAP内の一致するKey-valueペアを返します。このフィルター関数は、通常のBoolean配列または柔軟なLambda関数を使用することができます。Lambda関数の詳細については、[Lambda expression](../Lambda_expression.md)を参照してください。

この関数はバージョン3.1からサポートされています。

## 文法

```Haskell
MAP map_filter(any_map, array<boolean>)
MAP map_filter(lambda_func, any_map)
```

- `map_filter(any_map, array<boolean>)`

  `any_map`内で`array<boolean>`と一対一に対応する位置が`true`のkey-valueペアを返します。

- `map_filter(lambda_func, any_map)`

  `any_map`に`lambda_func`を適用した後、値が`true`のkey-valueペアを返します。

## パラメータ説明

- `any_map`: フィルター処理を行うMAP値。

- `array<boolean>`: `any_map`をフィルタリングするBoolean配列。

- `lambda_func`: `any_map`をフィルタリングするLambda関数。

## 戻り値の説明

入力された`any_map`と同じ型のmapを返します。

`any_map`がNULLの場合、結果もNULLです。`array<boolean>`がnullの場合、結果は空のMAPです。

Map内のあるKeyまたはValueがNULLの場合、そのNULL値は計算に正常に参加し、返されます。

Lambda関数には2つのパラメータが必要で、最初のパラメータはKeyを、2番目のパラメータはValueを表します。

## 例

### Lambda式を使用しない場合

以下の例では、[map_from_arrays](map_from_arrays.md)を使用して`{1:"ab",3:"cdd",2:null,null:"abc"}`のmap値を生成します。次に、`array<boolean>`をMap内の各key-valueペアに適用し、結果がtrueのkey-valueペアを返します。

```SQL
mysql> select map_filter(col_map, array<boolean>[0,0,0,1,1]) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+----------------------------------------------------+
| map_filter(col_map, ARRAY<BOOLEAN>[0, 0, 0, 1, 1]) |
+----------------------------------------------------+
| {null:"abc"}                                       |
+----------------------------------------------------+
1 row in set (0.02 sec)

mysql> select map_filter(null, array<boolean>[0,0,0,1,1]);
+-------------------------------------------------+
| map_filter(NULL, ARRAY<BOOLEAN>[0, 0, 0, 1, 1]) |
+-------------------------------------------------+
| NULL                                            |
+-------------------------------------------------+
1 row in set (0.02 sec)

mysql> select map_filter(col_map, null) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+---------------------------+
| map_filter(col_map, NULL) |
+---------------------------+
| {}                        |
+---------------------------+
1 row in set (0.01 sec)
```

### Lambda式を使用する場合

以下の例では、map_from_arrays()を使用して`{1:"ab",3:"cdd",2:null,null:"abc"}`のMap値を生成します。次に、Lambda関数をMap内の各key-valueペアに適用し、Valueがnullでないkey-valueペアを返します。

```SQL
mysql> select map_filter((k,v) -> v is not null,col_map) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+------------------------------------------------+
| map_filter((k,v) -> v is not null, col_map)    |
+------------------------------------------------+
| {1:"ab",3:"cdd",null:'abc'}                    |
+------------------------------------------------+
1 row in set (0.02 sec)
```

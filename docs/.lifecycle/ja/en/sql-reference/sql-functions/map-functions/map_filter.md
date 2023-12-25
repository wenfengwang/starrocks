---
displayed_sidebar: English
---

# map_filter

## 説明

マップ内のキーと値のペアに対してブール配列または[Lambda式](../Lambda_expression.md)を適用し、`true`と評価されるペアをフィルタリングして返します。

この関数はv3.1以降でサポートされています。

## 構文

```Haskell
MAP map_filter(any_map, array<boolean>)
MAP map_filter(lambda_func, any_map)
```

- `map_filter(any_map, array<boolean>)`

  `any_map`内のキーと値のペアを`array<boolean>`に対して一つずつ評価し、`true`と評価されるペアを返します。

- `map_filter(lambda_func, any_map)`

  `any_map`内のキーと値のペアに`lambda_func`を一つずつ適用し、結果が`true`であるペアを返します。

## パラメーター

- `any_map`: マップ値。

- `array<boolean>`: マップ値を評価するために使用されるブール配列。

- `lambda_func`: マップ値を評価するために使用されるLambda式。

## 戻り値

`any_map`と同じデータ型のマップを返します。

`any_map`がNULLの場合、NULLが返されます。`array<boolean>`がnullの場合、空のマップが返されます。

マップ値内のキーまたは値がNULLの場合、NULLは通常の値として処理されます。

Lambda式は2つのパラメータを持つ必要があります。最初のパラメータはキーを、2番目のパラメータは値を表します。

## 例

### `array<boolean>`を使用する

以下の例では、[map_from_arrays()](map_from_arrays.md)を使用してマップ値`{1:"ab",3:"cdd",2:null,null:"abc"}`を生成します。その後、`array<boolean>`に対して各キーと値のペアを評価し、`true`と評価されるペアを返します。

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

### Lambda式を使用する

以下の例では、map_from_arrays()を使用してマップ値`{1:"ab",3:"cdd",2:null,null:"abc"}`を生成します。その後、各キーと値のペアに対してLambda式を評価し、値がnullでないペアを返します。

```SQL

mysql> select map_filter((k,v) -> v is not null, col_map) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+------------------------------------------------+
| map_filter((k,v) -> v is not null, col_map)    |
+------------------------------------------------+
| {1:"ab",3:"cdd",null:"abc"}                   |
+------------------------------------------------+
1 row in set (0.02 sec)
```

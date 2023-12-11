---
displayed_sidebar: "Japanese"
---

# map_filter

## 説明

マップ内のキーと値のペアを、ブール配列または[ラムダ式](../Lambda_expression.md)を各々のキーと値のペアに適用してフィルタリングします。`true` と評価されるペアが返されます。

この関数はv3.1以降でサポートされています。

## 構文

```Haskell
MAP map_filter(any_map, array<boolean>)
MAP map_filter(lambda_func, any_map)
```

- `map_filter(any_map, array<boolean>)`

  `any_map`内のキーと値のペアを `array<boolean>` に対して一つずつ評価し、`true` と評価されるキーと値のペアを返します。

- `map_filter(lambda_func, any_map)`

  `any_map`内のキーと値のペアに対して `lambda_func` を適用し、結果が`true`となるキーと値のペアを返します。

## パラメータ

- `any_map`: マップの値。

- `array<boolean>`: マップの値を評価するために使用されるブール配列。

- `lambda_func`: マップの値を評価するために使用されるラムダ式。

## 戻り値

データ型が`any_map`と同じであるマップが返されます。

`any_map`がNULLの場合、NULLが返されます。`array<boolean>`がnullの場合、空のマップが返されます。

マップの値のキーまたは値がNULLの場合、NULLは通常の値として処理されます。

ラムダ式には2つのパラメータが必要です。最初のパラメータはキーを表し、2番目のパラメータは値を表します。

## 例

### `array<boolean>`の使用

次の例では、[map_from_arrays()](map_from_arrays.md)を使用して、マップの値 `{1:"ab",3:"cdd",2:null,null:"abc"}` を生成します。その後、それぞれのキーと値のペアを`array<boolean>`に対して評価し、結果が`true`となるペアが返されます。

```SQL
mysql> select map_filter(col_map, array<boolean>[0,0,0,1,1]) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+----------------------------------------------------+
| map_filter(col_map, ARRAY<BOOLEAN>[0, 0, 0, 1, 1]) |
+----------------------------------------------------+
| {null:"abc"}                                       |
+----------------------------------------------------+
1 行が返されました (0.02 秒)

mysql> select map_filter(null, array<boolean>[0,0,0,1,1]);
+-------------------------------------------------+
| map_filter(NULL, ARRAY<BOOLEAN>[0, 0, 0, 1, 1]) |
+-------------------------------------------------+
| NULL                                            |
+-------------------------------------------------+
1 行が返されました (0.02 秒)

mysql> select map_filter(col_map, null) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+---------------------------+
| map_filter(col_map, NULL) |
+---------------------------+
| {}                        |
+---------------------------+
1 行が返されました (0.01 秒)
```

### ラムダ式の使用

次の例では、map_from_array()を使用して、マップの値 `{1:"ab",3:"cdd",2:null,null:"abc"}` を生成します。その後、それぞれのキーと値のペアをラムダ式に対して評価し、値がNULLでないキーと値のペアが返されます。

```SQL
mysql> select map_filter((k,v) -> v is not null,col_map) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+------------------------------------------------+
| map_filter((k,v) -> v is not null, col_map)    |
+------------------------------------------------+
| {1:"ab",3:"cdd",null:'abc'}                        |
+------------------------------------------------+
1 行が返されました (0.02 秒)
```
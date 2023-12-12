---
displayed_sidebar: "Japanese"
---

# map_filter

## 説明

ブール配列または[ラムダ式](../Lambda_expression.md)を利用してマップ内のキーと値のペアをフィルタリングします。`true`と評価されるペアが返されます。

この機能はv3.1以降でサポートされています。

## 構文

```Haskell
MAP map_filter(any_map, array<boolean>)
MAP map_filter(lambda_func, any_map)
```

- `map_filter(any_map, array<boolean>)`

  `any_map`内のキーと値のペアを一つずつ`array<boolean>`と比較し、`true`と評価されたキーと値のペアを返します。

- `map_filter(lambda_func, any_map)`

  `lambda_func`を`any_map`内のキーと値のペアに一つずつ適用し、結果が`true`となるキーと値のペアを返します。

## パラメータ

- `any_map`: マップの値です。

- `array<boolean>`: マップの値を評価するために使用されるブール配列です。

- `lambda_func`: マップの値を評価するために使用されるラムダ式です。

## 戻り値

データ型が`any_map`と同じであるマップを返します。

`any_map`がNULLの場合、NULLが返されます。`array<boolean>`がnullの場合、空のマップが返されます。

マップの値のキーまたは値がNULLの場合、NULLは通常の値として処理されます。

ラムダ式は2つのパラメータを持つ必要があります。最初のパラメータはキーを表し、2番目のパラメータは値を表します。

## 例

### `array<boolean>`を使用

次の例では、[map_from_arrays()](map_from_arrays.md)を使用してマップ値`{1:"ab",3:"cdd",2:null,null:"abc"}`を生成します。それぞれのキーと値のペアを`array<boolean>`と比較し、結果が`true`となるペアが返されます。

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

### ラムダ式を使用

次の例では、map_from_arrays()を使用してマップ値`{1:"ab",3:"cdd",2:null,null:"abc"}`を生成します。それぞれのキーと値のペアをラムダ式と比較し、値がnullでないキーと値のペアが返されます。

```SQL
mysql> select map_filter((k,v) -> v is not null,col_map) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+------------------------------------------------+
| map_filter((k,v) -> v is not null, col_map)    |
+------------------------------------------------+
| {1:"ab",3:"cdd",null:'abc'}                        |
+------------------------------------------------+
1 行が返されました (0.02 秒)
```
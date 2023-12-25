---
displayed_sidebar: English
---

# transform_values

## 説明

マップ内の値を[ラムダ式](../Lambda_expression.md)を使用して変換し、マップ内の各エントリに対して新しい値を生成します。

この関数はv3.1以降でサポートされています。

## 構文

```Haskell
MAP transform_values(lambda_func, any_map)
```

`any_map` の後に `lambda_func` を配置することもできます。

```Haskell
MAP transform_values(any_map, lambda_func)
```

## パラメーター

- `any_map`: マップです。

- `lambda_func`: `any_map` に適用したいラムダ式です。

## 戻り値

ラムダ式の結果によって値のデータ型が決定され、キーのデータ型は `any_map` のキーと同じであるマップ値を返します。

いずれかの入力パラメータがNULLの場合、NULLが返されます。

元のマップのキーや値がNULLの場合、NULLは通常の値として処理されます。

ラムダ式には2つのパラメータが必要です。最初のパラメータはキーを表し、2番目のパラメータは値を表します。

## 例

以下の例では、[map_from_arrays](map_from_arrays.md)を使用してマップ値`{1:"ab",3:"cdd",2:null,null:"abc"}`を生成します。次に、ラムダ式がマップの各値に適用されます。最初の例では、各キーと値のペアの値を1に変更します。2番目の例では、各キーと値のペアの値をnullに変更します。

```SQL
mysql> select transform_values((k,v)->1, col_map) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+----------------------------------------+
| transform_values((k, v) -> 1, col_map) |
+----------------------------------------+
| {1:1,3:1,2:1,null:1}                   |
+----------------------------------------+
1 row in set (0.02 sec)

mysql> select transform_values((k,v)->null, col_map) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+--------------------------------------------+
| transform_values((k, v) -> NULL, col_map)  |
+--------------------------------------------+
| {1:null,3:null,2:null,null:null}           |
+--------------------------------------------+
1 row in set (0.01 sec)
```

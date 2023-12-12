---
displayed_sidebar: "Japanese"
---

# transform_values

## 説明

[ラムダ式](../Lambda_expression.md)を使用して、マップ内の値を変換し、マップ内の各エントリに新しい値を生成します。

この関数はv3.1以降でサポートされています。

## 構文

```Haskell
MAP transform_values(lambda_func, any_map)
```

`lambda_func`は`any_map`の後に配置することもできます：

```Haskell
MAP transform_values(any_map, lambda_func)
```

## パラメータ

- `any_map`: マップです。

- `lambda_func`: `any_map`に適用したいラムダ式です。

## 戻り値

値を変換すると、戻り値はデータ型の値がラムダ式の結果によって決定され、キーのデータ型は`any_map`のキーと同じです。

入力パラメータがNULLの場合、NULLが返されます。

元のマップ内のキーまたは値がNULLの場合、NULLは通常の値として処理されます。

ラムダ式には2つのパラメータが必要です。1つ目のパラメータはキーを、2つ目のパラメータは値を表します。

## 例

次の例では、[map_from_arrays](map_from_arrays.md)を使用して、マップ値`{1:"ab",3:"cdd",2:null,null:"abc"}`を生成します。その後、ラムダ式がマップの各値に適用されます。最初の例では、各キーと値の値が1に変更されます。2番目の例では、各キーと値の値がnullに変更されます。

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
| {1:null,3:null,2:null,null:null} |
+--------------------------------------------+
1 row in set (0.01 sec)
```
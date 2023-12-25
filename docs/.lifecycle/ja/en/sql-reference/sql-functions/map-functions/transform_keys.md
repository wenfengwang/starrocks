---
displayed_sidebar: English
---

# transform_keys

## 説明

マップ内のキーを[Lambda 式](../Lambda_expression.md)を使用して変換し、マップ内の各エントリに対して新しいキーを生成します。

この関数は v3.1 以降でサポートされています。

## 構文

```Haskell
MAP transform_keys(lambda_func, any_map)
```

`any_map` の後に `lambda_func` を配置することもできます。

```Haskell
MAP transform_keys(any_map, lambda_func)
```

## パラメーター

- `any_map`: マップです。

- `lambda_func`: `any_map` に適用したいLambda 式です。

## 戻り値

キーのデータ型がLambda 式の結果によって決定され、値のデータ型が `any_map` の値と同じであるマップ値を返します。

いずれかの入力パラメータがNULLの場合、NULLが返されます。

元のマップのキーまたは値がNULLの場合、NULLは通常の値として処理されます。

Lambda 式には2つのパラメータが必要です。最初のパラメータはキーを表し、2番目のパラメータは値を表します。

## 例

次の例では、[map_from_arrays](map_from_arrays.md)を使用してマップ値 `{1:"ab",3:"cdd",2:null,null:"abc"}` を生成します。その後、Lambda 式を各キーに適用し、キーを1増やします。

```SQL
mysql> select transform_keys((k,v)->(k+1), col_map) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+------------------------------------------+
| transform_keys((k, v) -> k + 1, col_map) |
+------------------------------------------+
| {2:"ab",4:"cdd",3:null,null:"abc"}       |
+------------------------------------------+
```

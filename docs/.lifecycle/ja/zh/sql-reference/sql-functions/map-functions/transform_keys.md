---
displayed_sidebar: Chinese
---

# transform_keys

## 機能

Map 中の key に対して Lambda 変換を行います。Lambda 関数についての詳細は、[Lambda expression](../Lambda_expression.md) を参照してください。

この関数はバージョン 3.1 からサポートされています。

## 文法

```Haskell
MAP transform_keys(lambda_func, any_map)
```

`lambda_func` を `any_map` の後に置くこともできます:

```Haskell
MAP transform_keys(any_map, lambda_func)
```

## パラメータ説明

- `any_map`：演算を行う Map 値。

- `lambda_func`：`any_map` のすべての key を変換する Lambda 関数。

## 戻り値の説明

map を返します。map の key の型は Lambda 関数の演算結果によって決まります；value の型は `any_map` の value の型と同じです。

入力パラメータが NULL の場合、結果も NULL です。

MAP の key または value が NULL の場合、その NULL 値は正常に処理されて返されます。

Lambda 関数には 2 つのパラメータが必要で、最初のパラメータが key、2 番目のパラメータが value を表します。

## 例

以下の例では、[map_from_arrays](map_from_arrays.md) を使用して map 値 `{1:"ab",3:"cdd",2:null,null:"abc"}` を生成し、その後で各 key に Lambda 関数を適用して key に 1 を加算します。

```SQL
mysql> select transform_keys((k,v)->(k+1), col_map) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+------------------------------------------+
| transform_keys((k, v) -> k + 1, col_map) |
+------------------------------------------+
| {2:"ab",4:"cdd",3:null,null:"abc"}       |
+------------------------------------------+
```

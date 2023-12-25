---
displayed_sidebar: Chinese
---

# transform_values

## 機能

Map の value に対して lambda 変換を行います。Lambda 関数の詳細については、[Lambda expression](../Lambda_expression.md) を参照してください。

この関数はバージョン 3.1 からサポートされています。

## 文法

```Haskell
map transform_values(lambda_func, any_map)
```

`lambda_func` を `any_map` の後に置くこともできます:

```Haskell
map transform_values(any_map, lambda_func)
```

## パラメータ説明

- `any_map`：計算対象の Map 値。

- `lambda_func`：`any_map` の全ての value を変換する Lambda 関数。

## 戻り値の説明

map を返します。map の value の型は Lambda 関数の計算結果によって決まります；key の型は `any_map` の key の型と同じです。

入力パラメータが NULL の場合、結果も NULL です。

MAP の中のある key または value が NULL の場合、その NULL 値は正常に処理されて返されます。

Lambda 関数には二つのパラメータが必要で、最初のパラメータが key、二番目のパラメータが value を表します。

## 例

以下の例では [map_from_arrays](map_from_arrays.md) を使用して map 値 `{1:"ab",3:"cdd",2:null,null:"abc"}` を生成し、その後で各 value に Lambda 関数を適用します。最初の例では各 value を 1 に変換し、二番目の例では各 value を null に変換します。

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
